---
keyword: [INSERT OVERWRITE, INSERT INTO]
---

# INSERT OVERWRITE and INSERT INTO

This topic describes how to use the INSERT OVERWRITE and INSERT INTO statements to update table data.

## INSERT statements

-   **Syntax**

    ```
    INSERT OVERWRITE|INTO TABLE table_name [PARTITION (partcol1=val1, partcol2=val2 ...)] [(col1,col2 ...)]
    select_statement
    FROM from_statement
    [ZORDER BY zcol1 [, zcol2 ...]] ;
    ```

-   **Description**

    When MaxCompute SQL processes data, the `INSERT OVERWRITE or INSERT INTO` statement is used to save the results to a destination table.

    -   INSERT INTO: inserts data into a table or partition. You cannot use `INSERT INTO` to insert data into a clustered table. If you want to insert a small amount of test data, you can use this statement with [VALUES](/intl.en-US/Development/SQL/Insert Operation/VALUES.md).
    -   INSERT OVERWRITE: clears the existing data in a table and inserts data into the table or its partition. If you use `INSERT OVERWRITE`, you cannot specify the columns into which data is inserted. If you want to specify the columns, use the `INSERT INTO` statement instead.

        **Note:**

        -   The syntax of `INSERT` statements in MaxCompute differs from that of `INSERT` statements in MySQL or Oracle. To execute `INSERT OVERWRITE or INSERT INTO` in MaxCompute, you must add keyword `TABLE` before `table_name` in the statement.
        -   If you execute the `INSERT OVERWRITE` statement on a partition several times, the size of the partition that you query by using `DESC` may vary. The reason is that the logic used to split a file changes after you use `SELECT` to extract the data from a partition and use `INSERT OVERWRITE` to insert the data into the same partition. The data length remains unchanged after the `INSERT OVERWRITE` statement is executed. Therefore, storage fees remain unchanged.
-   **Parameters**
    -   table\_name: the name of the destination table into which you want to insert data.
    -   PARTITION \(partcol1=val1, partcol2=val2 ...\): the name of the partition into which you want to insert data. The value must be a constant. It cannot be an expression, such as a function.
    -   \[\(col1,col2 ...\)\]: the name of the column into which you want to insert data. This parameter is unavailable for the `INSERT OVERWRITE` statement.
    -   select\_statement: the SELECT clause used to query the data that you want to insert from the source table.

        **Note:**

        -   The mappings between the source and destination tables are based on the column sequence in select\_statement. The mappings between column names of tables are not considered.
        -   If the destination table has static partitions and you want to insert data into a partition, its partition key columns cannot appear in select\_statement.
    -   from\_statement: the FROM clause used to indicate the data source. For example, the value can be a source table name.
    -   \[ZORDER BY zcol1 \[, zcol2 ...\]\]: If you write data to a table or partition, you can use this clause to place rows with similar data records in adjacent positions based on the columns specified in the select\_statement clause. This improves filtering performance for queries and reduces storage costs. The `ORDER BY x, y` clause sorts data records based on the ordering of x coming before y. The `ZORDER BY x, y` clause places rows that have similar x values in adjacent positions and rows that have similar y values in adjacent positions. If the filter condition of an SQL query statement includes sort columns, the ORDER BY clause filters and sorts data based on x while the ZORDER BY clause filters and sorts data based on x or on both x and y. This increases the column store ratio.

        **Note:**

        -   The ZORDER BY clause occupies a large number of resources to write data, which requires a longer time than data writes without ordering.
        -   If the destination table is a clustered table, ZORDER BY is not supported.

## Examples

Calculate the sales volumes of different regions listed in the `sale_detail` table. Then, insert the obtained data into the `sale_detail_insert` table.

```
-- Create a partitioned table named sale_detail, add partitions, and insert data into the table. You do not need to define partition key columns as common columns in the statements that are used to create the source table.
CREATE TABLE IF NOT EXISTS sale_detail
(
shop_name     string,
customer_id   string,
total_price   double
)
PARTITIONED BY (sale_date STRING,region STRING);

-- Add a partition to the source table.
ALTER TABLE sale_detail ADD PARTITION (sale_date='2013', region='china');

-- Insert data into the source table. The INSERT INTO TABLE table_name statement can be abbreviated as INSERT INTO table_name. However, the INSERT OVERWRITE TABLE table_name has no abbreviation.
INSERT INTO sale_detail PARTITION (sale_date='2013', region='china') VALUES ('s1','c1',100.1),('s2','c2',100.2),('s3','c3',100.3);

-- Create a destination table named sale_detail_insert with the same schema as the source table.
CREATE TABLE sale_detail_insert LIKE sale_detail;

-- Add a partition to the destination table.
ALTER TABLE sale_detail_insert ADD PARTITION (sale_date='2013', region='china');

-- Extract data from the sale_detail table and insert the data into the sale_detail_insert table.
-- Fields in the destination table do not need to be declared or rearranged.
-- If the destination table contains static partitions, partition fields have been declared in PARTITION(). These fields are no longer included in select_statement. You only need to search for the fields based on the sequence of common columns in the destination table and sequentially map these fields to those in the destination table. If the destination table contains dynamic partitions, partition fields must be included in select_statement. For more information, see [Insert data in dynamic partition mode \(DYNAMIC PARTITION\)](/intl.en-US/Development/SQL/Insert Operation/Insert data in dynamic partition mode (DYNAMIC PARTITION).md).
INSERT OVERWRITE TABLE sale_detail_insert PARTITION (sale_date='2013', region='china')
  SELECT 
  shop_name, 
  customer_id,
  total_price 
  FROM sale_detail
  ZORDER BY customer_id, total_price;
```

Take note of the following points:

-   The mappings between the source and destination tables are based on the column sequence in `SELECT`, rather than the mappings between column names of tables. Example:

    ```
    INSERT OVERWRITE TABLE sale_detail_insert PARTITION (sale_date='2013', region='china')
        SELECT customer_id, shop_name, total_price FROM sale_detail;                      
    ```

    If you create the `sale_detail_insert` table, the columns `shop_name STRING, customer_id STRING, and total_price BIGINT` are listed in sequence. If you insert data from the `sale_detail` table to the `sale_detail_insert` table, the data is inserted into the `customer_id, shop_name, and total_price` columns in sequence. In this case, data in `sale_detail.customer_id` is inserted into `sale_detail_insert.shop_name`, and data in `sale_detail.shop_name` is inserted into `sale_detail_insert.customer_id`.

-   If you insert data into a partition, its partition key columns cannot be included in `SELECT`. If you execute the following statement, an error is returned because `sale_date` and `region` are partition key columns. These columns cannot be included in the INSERT statement that is used to update table data to a static partition.

    ```
    INSERT OVERWRITE TABLE sale_detail_insert PARTITION (sale_date='2013', region='china')
       SELECT shop_name, customer_id, total_price, sale_date, region FROM sale_detail;
    ```

-   The value of `PARTITION` must be a constant and cannot be represented by an expression. The following example shows an incorrect usage:

    ```
    INSERT OVERWRITE TABLE sale_detail_insert PARTITION (sale_date=datepart('2016-09-18 01:10:00', 'yyyy') , region='china')
       SELECT shop_name, customer_id, total_price FROM sale_detail;
    ```


## Precautions

If you want to update table data to a dynamic partition, take note of the following points:

-   If you execute the `INSERT INTO PARTITION` statement and the specified partition does not exist, the system automatically creates this partition.
-   If multiple `INSERT INTO PARTITION` statements are concurrently executed and the specified partitions do not exist, the system attempts to create these partitions but only one partition is created.
-   If you cannot control the concurrency of the `INSERT INTO PARTITION` statement, we recommend that you use `ALTER TABLE` to create partitions in advance. For more information, see [Partition and column operations](/intl.en-US/Development/SQL/DDL SQL/Partition and column operations.md).

For more information about dynamic partitions, see [Insert data in dynamic partition mode \(DYNAMIC PARTITION\)](/intl.en-US/Development/SQL/Insert Operation/Insert data in dynamic partition mode (DYNAMIC PARTITION).md).

