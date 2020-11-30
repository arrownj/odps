---
keyword: [sorting, group query, nested query]
---

# SELECT syntax

This topic describes the SELECT syntax in MaxCompute and the precautions for the use of SELECT statements to perform operations such as nested query, sorting, and group query.

## Syntax

```
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
FROM table_reference
[WHERE where_condition]
[GROUP BY col_list]
[ORDER BY order_condition]
[DISTRIBUTE BY distribute_condition [SORT BY sort_condition] ]
[LIMIT number]
```

## Limits

-   A SELECT statement can return a maximum of 10,000 rows of results. This limit does not apply to SELECT clauses. If SELECT is used as a clause, the clause returns all results in response to the query from the upper layer.
-   You are not allowed to perform full table scans on partitioned tables by using SELECT statements.

    If your project was created after 20:00 on January 10, 2018, you are not allowed to perform full table scans on partitioned tables in your project by using SQL statements. To query data in partitioned tables, you must specify the partitions that you want to scan. This reduces unnecessary I/O of SQL statements and saves computing resources. This also reduces your computing costs if you use the pay-as-you-go billing method.

    To perform a full table scan on a partitioned table, insert the `set odps.sql.allow.fullscan=true;` command before the SQL statement that is used for the full table scan. Then, commit and execute the inserted command with the SQL statement. Assume that `sale_detail` is a partitioned table. To perform a full table scan on this table, execute the following statements:

    ```
    set odps.sql.allow.fullscan=true;
    SELECT * FROM sale_detail;
    ```


## Column expression select\_expr

To use a SELECT statement to read data from a table, you can specify the columns that you want to read by using one of the following methods:

-   Specify the names of specific columns. For example, to read the `shop_name` column from the `sale_detail` table, execute the following statement:

    ```
    SELECT shop_name FROM sale_detail;
    ```

-   Use an asterisk \(`*`\) to represent all columns. For example, to read all columns from the `sale_detail` table, execute the following statement:

    ```
    SELECT * FROM sale_detail;
    ```

    You can use a `WHERE` clause to specify filter conditions.

    ```
    SELECT * FROM sale_detail WHERE shop_name LIKE 'hang%';
    ```

-   Use a regular expression. Examples:

    -   `SELECT `abc. *` FROM t;`: Select all columns whose names start with `abc` from the `t` table.
    -   `SELECT `(ds)? +. +` FROM t;`: Select all columns whose names are not `ds` from the `t` table.
    -   `SELECT `(ds|pt)? +. +` FROM t;`: Select all columns except the columns whose names are `ds` and `pt` from the `t` table.
    -   `SELECT `(d. *)? +. +` FROM t;`: Select all columns except the columns whose names start with `d` from the `t` table.
    **Note:**

    If the name of col2 is the prefix of the name of col1 and you want to exclude multiple columns, make sure that the name of col1 is written before that of col2. The longer one is placed in the front. For example, if you want to exclude partitions `ds` and `dshh` from your query of a table, execute `SELECT `(dshh|ds)? +. +` FROM tbl;` instead of `SELECT `(ds|dshh)? +. +` FROM tbl;`.

-   Use `DISTINCT` before the name of a column to eliminate duplicate values from the column and return distinct values. If you use `ALL` before the name of a column, all values, including the duplicate values, in the column are returned. If this option is not specified, `ALL` is used.

    Example:

    ```
    -- Query data from the region column in the sale_detail table and return distinct values.
    SELECT DISTINCT region FROM sale_detail;
    +------------+
    | region     |
    +------------+
    | shanghai   |
    +------------+
    -- If you specify multiple columns after the DISTINCT option in a SELECT statement, the DISTINCT option takes effect on all the specified columns instead of a single column.
    SELECT DISTINCT region, sale_date FROM sale_detail;
    +------------+------------+
    | region     | sale_date  |
    +------------+------------+
    | shanghai   | 20191110   |
    +------------+------------+
    ```


## table\_reference

`table_reference` specifies the table that you want to query. You can also use table\_reference to specify the condition of a nested subquery. The following statement is an example:

```
SELECT * FROM (SELECT region FROM sale_detail) t WHERE region = 'shanghai';
```

## WHERE

The following table describes the filter conditions that the `WHERE` clause supports.

|Filter condition|Description|
|:---------------|:----------|
|\>, <, =, \>=, <=, and <\>|Relational operators.|
|LIKE and RLIKE|The source and pattern parameters of LIKE and RLIKE must be of the STRING type.|
|IN and NOT IN|If a subquery is added after the condition IN or NOT IN, the values in only one column can be returned for the subquery and the number of return values cannot exceed 1,000.|
|BETWEEN…AND|The condition that specifies the query range.|

**Examples**

-   In the `WHERE` clause of a SELECT statement, you can specify the range of partitions that you want to scan in a table. The following statement is an example:

    ```
    SELECT sale_detail. * 
    FROM sale_detail
    WHERE sale_detail.sale_date >= '2008'
    AND sale_detail.sale_date <= '2014';
    ```

    **Note:** You can execute the `EXPLAIN SELECT` statement to check whether partition pruning takes effect. A common user-defined function \(UDF\) or the partition condition settings of `JOIN` may cause partition pruning to fail. For more information, see [Check whether partition pruning is effective](/intl.en-US/Best Practices/SQL/Check whether partition pruning is effective.md).

-   UDFs support partition pruning. UDFs are executed as small jobs and then replaced with the execution results. You can enable UDF-based partition pruning by using one of the following methods:
    -   Add an annotation to the UDF class when you write a UDF.

        ```
        @com.aliyun.odps.udf.annotation.UdfProperty(isDeterministic=true)
        ```

        **Note:** The UDF annotation `com.aliyun.odps.udf.annotation.UdfProperty` is defined in the odps-sdk-udf.jar file. You must update the version of the referenced odps-sdk-udf to 0.30.X or later.

    -   Insert `set odps.sql.udf.ppr.deterministic = true;` before the SQL statements to execute. This way, all UDFs in the SQL statements are treated as `deterministic` UDFs. The preceding SET statement backfills partitions with execution results. A maximum of 1,000 partitions can be backfilled. If you add annotations to UDF classes, an error may be returned, which indicates that more than 1,000 backfilling results have been produced. To ignore this error, you can run the `set odps.sql.udf.ppr.to.subquery = false;` command. After you run this command, UDF-based partition pruning is no longer in effect.
-   The following example shows how to use the `BETWEEN…AND` condition to filter data:

    ```
    SELECT sale_detail. * 
    FROM sale_detail 
    WHERE sale_detail.sale_date BETWEEN '2008' AND '2014';
    ```


## GROUP BY

Generally, `GROUP BY` is used with [aggregate functions](/intl.en-US/Development/SQL/Builtin functions/Aggregate functions.md). If a `SELECT` statement contains aggregate functions, the following rules apply:

-   During the parsing of SQL statements, `GROUP BY` precedes `SELECT`. Therefore, the key of `GROUP BY` must be the name or expression of a column in the input table in the `SELECT` statement. You are not allowed to specify the alias of an output column in the `SELECT` statement as the key of GROUP BY.
-   The key of `GROUP BY` may be both the name or expression of a column in the input table and the alias of an output column in the `SELECT` statement. In this case, the name or expression of the column in the input table is used as the key of GROUP BY.
-   If `set hive.groupby.position.alias=true;` is inserted before a SELECT statement, the constants of the INTEGER type in `GROUP BY` are treated as column numbers in the SELECT statement.

    ```
    set hive.groupby.position.alias=true;-- Run this command with the subsequent SELECT statement.
    SELECT region, SUM(total_price) FROM sale_detail GROUP BY 1;-- 1 indicates region, which is the first column read by the SELECT statement. This statement groups the table data based on the values in the region column and returns the distinct region value and total sales of each group.
    ```


**Examples**

```
-- The statement uses the name of the region column in the input table as the key of GROUP BY and groups the table data based on the values in the region column.
SELECT region FROM sale_detail GROUP BY region;
-- The statement groups the table data based on the values in the region column and returns the total sales of each group.
SELECT SUM(total_price) FROM sale_detail GROUP BY region;
-- The statement groups the table data based on the values in the region column and returns the distinct region value and total sales of each group.
SELECT region, SUM(total_price) FROM sale_detail GROUP BY region;
-- An error is returned because the alias of the column in the SELECT statement is used as the key of the GROUP BY clause.
SELECT region AS r FROM sale_detail GROUP BY r;
-- A complete expression of the column is required.
SELECT 2 + total_price AS r FROM sale_detail GROUP BY 2 + total_price;
-- An error is returned because a column that does not use aggregate functions in the SELECT statement is not included in the GROUP BY clause.
SELECT region, total_price FROM sale_detail GROUP BY region;
-- The statement can be executed because all the columns that do not use aggregate functions exist in the GROUP BY clause.
SELECT region, total_price FROM sale_detail GROUP BY region, total_price;     
```

## ORDER BY\|DISTRIBUTE BY\|SORT BY

-   `ORDER BY`

    ORDER BY is used to sort all data records based on the specified columns.

    -   To sort data records in descending order, use the `DESC` keyword. By default, data records are sorted in ascending order.
    -   When you use `ORDER BY` to sort data records, NULL is considered as the smallest value. This is also the case in MySQL, but not in Oracle.
    -   `ORDER BY` must be followed by the alias of a specific column in the `SELECT` statement. If a column is included in the `SELECT` statement but you do not specify an alias for the column, the name of the column is treated as its alias.
    -   If `set hive.orderby.position.alias=true;` is inserted before a SELECT statement, the constants of the INTEGER type in ORDER BY are treated as column numbers in the SELECT statement.

        ```
        -- Set the flag.
        set hive.orderby.position.alias=true;
        -- Create the src table.
        CREAT table src(key BIGINT,value BIGINT);
        -- Query the src table and sort the first 100 records that are returned in ascending order by value.
        SELECT * FROM src ORDER BY 2 LIMIT 100;
        The preceding statement is equivalent to the following statement:
        SELECT * FROM src ORDER BY value LIMIT 100;
        ```

    -   You can use OFFSET with the ORDER BY LIMIT clause to skip a specific number of rows.

        ```
        -- Sort the rows of the src table in ascending order by key, and return the eleventh to thirtieth rows. OFFSET 10 indicates that the first 10 rows are skipped, and LIMIT 20 indicates that a maximum of 20 rows can be returned.
        SELECT * FROM src ORDER BY key LIMIT 20 OFFSET 10;
        ```

    **Examples**

    ```
    -- Query the sale_detail table and sort the first 100 records that are returned in ascending order by region.
    SELECT * FROM sale_detail ORDER BY region LIMIT 100;
    -- An error is returned because ORDER BY is not used with LIMIT.
    SELECT * FROM sale_detail ORDER BY region;
    -- ORDER BY is followed by a column alias.
    SELECT region AS r FROM sale_detail ORDER BY region LIMIT 100;
    SELECT region AS r FROM sale_detail ORDER BY r LIMIT 100;
    ```

-   `DISTRIBUTE BY`

    DISTRIBUTE BY is used to shard data based on the hash values of specified columns. You must specify the alias of an output column of the `SELECT` statement as the key of DISTRIBUTE BY.

    **Examples**

    ```
    -- The statement queries values in the region column of the sale_detail table and shards data based on the hash values of the region column.
    SELECT region FROM sale_detail DISTRIBUTE BY region;
    -- The statement can be properly executed because the column name is an alias.
    SELECT region AS r FROM sale_detail DISTRIBUTE BY region;
    The preceding statement is equivalent to the following statement:
    SELECT region AS r FROM sale_detail DISTRIBUTE BY r;
    ```

-   `SORT BY`

    SORT BY is used to sort specific data records. You can add ASC or DESC to sort specific data records in ascending or descending order. If SORT BY is not followed by a keyword, data records are sorted in ascending order.

    -   If `SORT BY` is preceded by `DISTRIBUTE BY`, `SORT BY` sorts the results of `DISTRIBUTE BY`. `DISTRIBUTE BY` determines how the map output values are distributed to reducers. If you want to prevent the overlap of the reducer output values or you want to process the same group of data together, you can use `DISTRIBUTE BY` to ensure that the same group of data is distributed to the same reducer. Then, use `SORT BY` to sort the group of data. The following code shows an example:

        ```
        -- The statement queries values in the region column of the sale_detail table, shards data based on the hash values of the region column, and then sorts the sharded data.
        SELECT region FROM sale_detail DISTRIBUTE BY region SORT BY region;
        ```

    -   If the `SORT BY` statement is not preceded by `DISTRIBUTE BY`, `SORT BY` sorts the data in each reducer. This process partially sorts data records. This ensures that the output values of each reducer are sorted in order and increases the storage compression ratio. If data is filtered during data reading, this method reduces the amount of data read from disks and improves subsequent global sorting efficiency.
    **Examples**

    ```
    SELECT region FROM sale_detail SORT BY region;
    ```


**Note:**

-   The value of ORDER BY, DISTRIBUTE BY, or SORT BY must be the alias of an output column in the `SELECT` statement. The column alias can be Chinese.
-   During the parsing of MaxCompute SQL statements, the ORDER BY, DISTRIBUTE BY, or SORT BY statement is executed after the `SELECT` statement. Therefore, the value of ORDER BY, DISTRIBUTE BY, or SORT BY must be the alias of an output column in the `SELECT` statement.``
-   `ORDER BY` cannot be used with `DISTRIBUTE BY` or `SORT BY`. Similarly, `GROUP BY` cannot be used with `DISTRIBUTE BY` or `SORT BY`.

## LIMIT NUMBER

The `number` in the `LIMIT NUMBER` clause is a constant that limits the number of output rows.

**Limits on the number of rows displayed on the screen**

If you use a `SELECT` statement without `LIMIT` or the number in the `LIMIT` clause exceeds the maximum number of rows displayed on the screen, you can only view the rows within the upper limit on the screen.

The upper limit on the number of rows displayed on the screen varies based on projects. You can use one of the following methods to determine the upper limit:

-   If project data protection is disabled, modify the odpscmd config.ini file.

    Set `use_instance_tunnel=true` in the odpscmd config.ini file. If the `instance_tunnel_max_record` parameter is not configured, the number of rows displayed on the screen is not limited. Otherwise, the number of rows displayed is limited by the `instance_tunnel_max_record` parameter. The maximum value of the `instance_tunnel_max_record` parameter is 10000. For more information about Instance Tunnel, see [Tunnel command usage](/intl.en-US/Development/Data upload and download/Run Tunnel commands to upload and download data/Tunnel command usage.md).

-   If project data protection is enabled, the number of rows displayed on the screen is limited by `READ_TABLE_MAX_ROW`. The maximum value of this parameter is 10000.

**Note:** You can run the `show SecurityConfiguration;` command to view the configuration of `ProjectProtection`. If `ProjectProtection` is set to true, you can determine whether to disable project data protection as required. You can run the `set ProjectProtection=false;` command to disable this feature. `ProjectProtection` is disabled by default. For more information about project data protection, see [Project data protection](/intl.en-US/Management/Configure security features/Project data protection.md).

