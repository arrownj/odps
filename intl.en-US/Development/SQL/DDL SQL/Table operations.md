---
keyword: [create a table, view table information, rename a table, delete a table, modify a table, hash clustering table, range clustering table]
---

# Table operations

This topic describes how to execute DDL statements to create, view, delete, rename, and modify tables on the MaxCompute client, odpscmd.

## Create a table

Syntax

```
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [DEFAULT value] [COMMENT col_comment], ...)]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[CLUSTERED BY | RANGE CLUSTERED BY (col_name [, col_name, ...]) [SORTED BY (col_name [ASC | DESC] [, col_name [ASC | DESC] ...])] INTO number_of_buckets BUCKETS] -- Used to set the shuffle and sort attributes when you create a clustering table.
[STORED BY StorageHandler] -- Used only for external tables.
[WITH SERDEPROPERTIES (Options)] -- Used only for external tables.
[LOCATION OSSLocation] -- Used only for external tables.
[LIFECYCLE days];

 CREATE TABLE [IF NOT EXISTS] table_name
 [AS select_statement | LIKE existing_table_name];
```

-   IF NOT EXISTS: If you do not specify `IF NOT EXISTS` and a table with an identical name already exists, an error is returned. If you specify this option, a success message is returned, regardless of whether a table with an identical name already exists. This is true even if the schema of the existing table is different from that of the table you want to create. If a table you want to create shares the same name with an existing table, the table is not created and the metadata of the existing table is not changed.
-   table\_name and col\_name:
    -   The table name and column name are not case-sensitive and cannot contain special characters. A table name or column name cannot exceed 128 bytes in length, and can contain only lowercase and uppercase letters, digits, and underscores \(\_\). We recommend that you start the table name or column name with a letter.
    -   A table can contain a maximum of 1,200 columns.
-   data\_type: the data type, such as BIGINT, DOUBLE, BOOLEAN, DATETIME, DECIMAL, and STRING. MaxCompute V2.0 supports more data types. For more information, see [Data types](/intl.en-US/Development/Data types/Data type editions.md).

    **Note:** To use new data types, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, and BINARY, in MaxCompute SQL, you must enable these data types.

    -   Session level: To enable new data types at the session level, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement and execute the statements together.
    -   Project level: To enable new data types at the project level, the project owner must run the following command:

        ```
        setproject odps.sql.type.system.odps2=true;
        ```

        For more information about `setproject`, see [Specify project properties](/intl.en-US/Development/Common commands/Project operations.md).

-   DEFAULT value: the default value of a column. If the value of the column is not specified in the INSERT statement, the default value is used for the column.
-   COMMENT table\_comment: the comment of the table. The comment must be a valid string that does not exceed 1,024 bytes in length.
-   PARTITIONED BY \(col\_name data\_type \[COMMENT col\_comment\], …\): the name and data type of the column in the table. The following data types are supported: TINYINT, SMALLINT, INT, BIGINT, VARCHAR, and STRING.

    The column value cannot exceed 128 bytes in length, and can contain letters, digits, and some supported special characters. The column value must start with a letter. It cannot contain double-byte characters. The following special characters are supported: spaces, colons \(:\), underscores \(\_\), dollar signs \($\), number signs \(\#\), periods \(.\), exclamation points \(!\), and at signs \(@\). Other characters, such as the tab \(\\t\), new line \(\\n\), and forward slash \(/\), are considered undefined characters. If you use column names to partition a table, a full table scan is not required when you add partitions, update partition data, or read partition data. This improves efficiency in processing tables.

    **Note:** A maximum of 60,000 partitions for six or lower levels are allowed in a table.

-   CLUSTERED BY \| RANGE CLUSTERED BY \(col\_name \[, col\_name, …\]\) \[SORTED BY \(col\_name \[ASC \| DESC\] \[, col\_name \[ASC \| DESC\] …\]\)\] INTO number\_of\_buckets BUCKETS: used to set the shuffle and sort attributes when you create a clustering table.

    Clustering tables are classified into hash clustering tables and range clustering tables.

    -   Hash clustering table
        -   CLUSTERED BY: the hash key. MaxCompute performs a hash operation on the specified columns and distributes data to each bucket based on the hash values. To prevent data skew and hot spots and to achieve high concurrence efficiency, we recommend that you specify a column that has large value ranges and a small number of duplicate key-value pairs in `CLUSTERED BY`. To optimize the `JOIN` operation, we recommend that you select commonly used join or aggregation keys. The join and aggregation keys are similar to the primary keys in conventional databases.
        -   SORTEDBY: the sequence of fields in a bucket. To achieve better performance, we recommend that you set SORTED BY and CLUSTERED BY to the same value. If you specify the SORTED BY clause, MaxCompute automatically generates indexes and uses these indexes to speed up your queries.
        -   INTO number\_of\_buckets BUCKETS: the number of hash buckets. This parameter is required and varies based on the data volume. By default, MaxCompute supports a maximum of 1,111 reducers. Therefore, MaxCompute supports a maximum of 1,111 hash buckets. You can run the `set odps.sql.reducer.instances=xxx;` command to increase the maximum number of hash buckets. However, the maximum number of hash buckets cannot exceed 4,000. Otherwise, performance may be affected.

            To maintain optimal performance, we recommend that you pay attention to the following rules when you specify the number of hash buckets:

            -   Keep the size of each bucket around 500 MB. For example, if you want to add 1,000 buckets to a partition whose size is 500 GB, the size of each bucket is 500 MB on average. If a table contains large amounts of data, you can increase the size of each bucket from 500 MB to a size in the range of 2 GB to 3 GB. You can also run the `set odps.sql.reducer.instances=xxx;` command to set the maximum number of hash buckets to a value greater than 1111.
            -   To optimize the performance of JOIN queries, we recommend that you do not set the shuffle and sort attributes for clustering tables that you created. The two tables must have a multiple relationship in terms of the number of hash buckets. For example, one table has 256 hash buckets and the other table has 512 hash buckets. We recommend that you set the number of hash buckets to 2n, such as 512, 1024, 2048, and 4096. This way, MaxCompute can automatically split and merge hash buckets. To improve execution efficiency, you can also skip the step to set the shuffle and sort attributes for the clustering tables.
    -   Range clustering table
        -   RANGE CLUSTERED BY: the range clustering table. MaxCompute performs the bucket operation on the specified columns and distributes data to each bucket based on the bucket numbers.
        -   SORTED BY: the sequence of fields in a bucket. You can use this parameter in the same way as you use it for a hash clustering table.
        -   INTO number\_of\_buckets BUCKETS: the number of hash buckets. Compared with hash clustering tables, range clustering tables have no limits on the number of buckets when the data is evenly distributed. If you do not specify the number of buckets in a range clustering table, MaxCompute automatically determines the optimal number based on the data volume.
        -   If join and aggregate operations are performed on range clustering tables and the join key or group key is the range clustering key or the prefix of the range clustering key, you can control flags to disable shuffling. This improves execution efficiency. To control shuffling, you can set `set odps.optimizer.enable.range.partial.repartitioning` to true or false. By default, this parameter is set to false, which indicates that shuffling is disabled.

            **Note:**

            -   Clustering tables help optimize the following aspects:
                -   Bucket pruning
                -   Aggregation
                -   Storage
            -   Limits on clustering tables
                -   The `INSERT INTO` statement is not supported. You can execute only the `INSERT OVERWRITE` statement to add data.
                -   The data that is imported by using Tunnel commands is not arranged in order. Therefore, you cannot import data into a range clustering table by using Tunnel commands.
-   Clauses related to external tables:
    -   STORED BY StorageHandler: the storage handler that is specified based on the format of data in the external table.
    -   WITH SERDEPROPERTIES \(Options\): the parameters for authorization, compression, and character parsing in the external table.
    -   LOCATION OSSLocation: the Object Storage Service \(OSS\) directory where the external table is saved. For more information, see [Access OSS data by using the built-in extractor](/intl.en-US/Development/External table/Access OSS data by using the built-in extractor.md) and [Process OSS data stored in open source formats](/intl.en-US/Development/External table/Process OSS data stored in open source formats.md).
-   LIFECYCLE: the lifecycle of the external table in days. If you execute the `CREATE TABLE LIKE` statement to create a table based on the definition of another table, the lifecycle settings of the source table are not replicated.
-   CREATE TABLE \[IF NOT EXISTS\] table\_name \[AS select\_statement \| LIKE existing\_table\_name\]:
    -   If you execute the `CREATE TABLE…AS select_statement…` statement to create a table with data replicated from another table, the replicated information does not include partition attributes. The reason is that partition columns in the source table are considered common columns in the newly created table.
    -   If you execute the CREATE TABLE…LIKE existing\_table\_name statement to create a table, the destination table may have the same schema as the source table. However, the destination table does not replicate data and lifecycle settings from the source table.

**Examples for creating a partitioned table**

The following example shows how to create a partitioned table named `sale_detail` to store sales records. The table has two partition columns: `sale_date` and `region`. Syntax:

```
CREATE TABLE IF NOT EXISTS sale_detail
(
shop_name     STRING,
customer_id   STRING,
total_price   DOUBLE
)
PARTITIONED BY (sale_date STRING,region STRING);
-- Create the sale_detail partitioned table.
```

**Example of `CREATE TABLE...AS...`**

You can run `CREATE TABLE...AS select_statement...` to create a table with data replicated from another table. Example:

```
CREATE TABLE sale_detail_ctas1 AS
SELECT * FROM sale_detail;
```

If the `sale_detail` table contains data, all data in the `sale_detail` table is replicated to the `sale_detail_ctas1` table after you execute the preceding statement.

**Note:** The `sale_detail` table is a partitioned table, whereas the sale\_detail\_ctas1 table created by using `CREATE TABLE...AS select_statement...` is a non-partitioned table that has five columns. This is because the replicated information does not include partition attributes and partition columns in the sale\_detail table are considered common columns in the `sale_detail_ctas1` table.

If the SELECT clause in the `CREATE TABLE...AS select_statement...` statement uses constants as column values, we recommend that you specify column names. Example:

```
CREATE TABLE sale_detail_ctas2
AS
SELECT shop_name, customer_id, total_price, '2013' AS sale_date, 'China' AS region
FROM sale_detail;
```

Example without column aliases listed:

```
CREATE TABLE sale_detail_ctas3
AS
SELECT shop_name, customer_id, total_price, '2013', 'China'
FROM sale_detail;
```

In this example, the names of the fourth and fifth columns in the `sale_detail_ctas3` table are similar to `_c4` and `_c5`.

**Example of `CREATE TABLE...LIKE...`**

If you want the source and destination tables to have the same schema, you can execute `CREATE TABLE...LIKE...`. Example:

```
CREATE TABLE sale_detail_like LIKE sale_detail;
```

In this case, the schema of the `sale_detail_like` table is the same as that of the `sale_detail` table. Except for the lifecycle settings, the two tables have the same column names, column comments, and table comments. However, data in the `sale_detail` table is not replicated to the `sale_detail_like` table.

**Example for creating a clustering table**

Examples for creating hash clustering tables:

```
CREATE TABLE T1 (a STRING, b STRING, c BIGINT) CLUSTERED BY (c) SORTED by (c) INTO 1024 BUCKETS; -- Create a hash clustering non-partitioned table.
CREATE TABLE T1 (a STRING, b STRING, c BIGINT) PARTITIONED BY (dt STRING) CLUSTERED BY (c) SORTED by (c) INTO 1024 BUCKETS; -- Create a hash clustering partitioned table.
```

Examples for creating range clustering tables:

```
CREATE TABLE T2 (a STRING, b STRING, c BIGINT) RANGE CLUSTERED BY (c) SORTED by (c) INTO 1024 BUCKETS; -- Create a range clustering non-partitioned table.
CREATE TABLE T2 (a STRING, b STRING, c BIGINT) PARTITIONED BY (dt STRING) CLUSTERED BY (c) SORTED by (c) ; -- Create a range clustering partitioned table for which the number of buckets is not specified.
```

## View table information

Syntax:

```
DESC <table_name>;
DESC EXTENDED <table_name>; -- View information about an external table.
```

Examples

-   To view the information about the `sale_detail` table, run the following command:

    ```
    DESC sale_detail;
    ```

    The following result is returned:

    ```
    odps@ $odps_project>DESC sale_detail;
    +--------------------------------------------------------------------+
    | Owner: ALIYUN$maoXXX@alibaba-inc.com | Project: $odps_project      |
    | TableComment:                                                      |
    +--------------------------------------------------------------------+
    | CreateTime:               2017-06-28 15:05:17                      |
    | LastDDLTime:              2017-06-28 15:05:17                      |
    | LastModifiedTime:         2017-06-28 15:05:17                      |
    +--------------------------------------------------------------------+
    | InternalTable: YES      | Size: 0                                  |
    +--------------------------------------------------------------------+
    | Native Columns:                                                    |
    +--------------------------------------------------------------------+
    | Field           | Type       | Label | Comment                     |
    +--------------------------------------------------------------------+
    | shop_name       | string     |       |                             |
    | customer_id     | string     |       |                             |
    | total_price     | double     |       |                             |
    +--------------------------------------------------------------------+
    | Partition Columns:                                                 |    
    +--------------------------------------------------------------------+
    | sale_date       | string     |                                     |
    | region          | string     |                                     |
    +--------------------------------------------------------------------+
    OK
    ```

-   To view the information about the `sale_detail_ctas2` table, run the following command:

    ```
    DESC sale_detail_ctas2;
    ```

    The following result is returned:

    ```
    odps@ $odps_project>desc sale_detail_ctas2;
    +--------------------------------------------------------------------+
    | Owner: ALIYUN$xxxxx@alibaba-inc.com | Project: $odps_project       |
    | TableComment:                                                      |
    +--------------------------------------------------------------------+
    | CreateTime:               2017-06-28 15:42:17                      |
    | LastDDLTime:              2017-06-28 15:42:17                      |
    | LastModifiedTime:         2017-06-28 15:42:17                      |
    +--------------------------------------------------------------------+
    | InternalTable: YES      | Size: 0                                  |
    +--------------------------------------------------------------------+
    | Native Columns:                                                    |
    +--------------------------------------------------------------------+
    | Field           | Type       | Label | Comment                     |
    +--------------------------------------------------------------------+
    | shop_name       | string     |       |                             |
    | customer_id     | string     |       |                             |
    | total_price     | double     |       |                             |
    | sale_date       | string     |       |                             |
    | region          | string     |       |                             |
    +--------------------------------------------------------------------+
    OK
    ```

-   To view the information about the `sale_detail_like` table, run the following command:

    ```
    DESC sale_detail_like;
    ```

    The following result is returned:

    ```
    odps@ $odps_project>DESC sale_detail_like;
    +--------------------------------------------------------------------+
    | Owner: ALIYUN$xxxxx@alibaba-inc.com | Project: $odps_project       |
    | TableComment:                                                      |
    +--------------------------------------------------------------------+
    | CreateTime:               2017-06-28 15:42:17                      |
    | LastDDLTime:              2017-06-28 15:42:17                      |
    | LastModifiedTime:         2017-06-28 15:42:17                      |
    +--------------------------------------------------------------------+
    | InternalTable: YES      | Size: 0                                  |
    +--------------------------------------------------------------------+
    | Native Columns:                                                    |
    +--------------------------------------------------------------------+
    | Field           | Type       | Label | Comment                     |
    +--------------------------------------------------------------------+
    | shop_name       | string     |       |                             |
    | customer_id     | string     |       |                             |
    | total_price     | double     |       |                             |
    +--------------------------------------------------------------------+
    | Partition Columns:                                                 |
    +--------------------------------------------------------------------+
    | sale_date       | string     |                                     |
    | region          | string     |                                     |
    +--------------------------------------------------------------------+
    OK
    ```


Except for the lifecycle settings, the attributes, such as field types and partition types, of the `sale_detail_like` table are the same as the attributes of the `sale_detail` table. For more information, see [Table-level operations](/intl.en-US/Development/Common commands/Table-level operations.md).

**Note:** The data size in the output of the `DESC table_name` command includes the data size of the recycle bin. Before you clear data from the recycle bin, you can execute `PURGE TABLE table_name` and `DESC table_name` in sequence to view the size of data except the data in the recycle bin. You can also execute `SHOW recyclebin` to view the details about data in the recycle bin for the current project.

If you view the information about the `sale_detail_ctas1` table, you can find that the `sale_date` and `region` columns are considered common columns and are not partition columns.

The types of data that the `DESC` statement can return increase with the data types supported by MaxCompute. For more information, see [Date types](/intl.en-US/Development/Data types/Data type editions.md). To use new data types in MaxCompute, you can run the `DESC` command without additional operations. If you use other SQL statements, you must enable the new data types in advance.

**Note:** If the output of SQL statements is based on the input of the `DESC table_name` command, we recommend that you update settings to parse new data types in MaxCompute in a timely manner.

Example:

```
set odps.sql.type.system.odps2=true;
CREATE TABLE test_newtype (
    c1 TINYINT
    ,c2 SMALLINT
    ,c3 INT
    ,c4 BIGINT
    ,c5 FLOAT
    ,c6 DOUBLE
    ,c7 DECIMAL
    ,c8 BINARY
    ,c9 TIMESTAMP
    ,c10 ARRAY<MAP<BIGINT,BIGINT>>
    ,c11 MAP<STRING,ARRAY<BIGINT>>
    ,c12 STRUCT<s1:STRING,s2:BIGINT>
    ,c13 VARCHAR(20))
LIFECYCLE 1
;
```

The following result is returned after you run the `DESC test_newtype;` command:

```
| Native Columns:                                                                    |
+------------------------------------------------------------------------------------+
| Field           | Type       | Label | Comment                                     |
+------------------------------------------------------------------------------------+
| c1              | tinyint    |       |                                             |
| c2              | smallint   |       |                                             |
| c3              | int        |       |                                             |
| c4              | bigint     |       |                                             |
| c5              | float      |       |                                             |
| c6              | double     |       |                                             |
| c7              | decimal    |       |                                             |
| c8              | binary     |       |                                             |
| c9              | timestamp  |       |                                             |
| c10             | array<map<bigint,bigint>> |       |                              |
| c11             | map<string,array<bigint>> |       |                              |
| c12             | struct<s1:string,s2:bigint> |       |                            |
| c13             | varchar(20) |       |                                            |
+------------------------------------------------------------------------------------+
```

You can run the `DESC EXTENDED table_name;` command to view the clustering attributes of the clustering table. For example, if you run the DESC extended t1; command, the following result is returned. The clustering attributes are displayed in Extended Info of the result.

```
odps@ $odps_project>DESC extended t1;
+------------------------------------------------------------------------------------+
| Owner: ALIYUN$xxxxxxx@aliyun.com | Project: xxxxx                                  |
| TableComment:                                                                      |
+------------------------------------------------------------------------------------+
| CreateTime: 2017-12-25 11:18:26                                                    |
| LastDDLTime: 2017-12-25 11:18:26                                                   |
| LastModifiedTime: 2017-12-25 11:18:26                                              |
| Lifecycle: 2                                                                       |
+------------------------------------------------------------------------------------+
| InternalTable: YES | Size: 0                                                       |
+------------------------------------------------------------------------------------+
| Native Columns:                                                                    |
+------------------------------------------------------------------------------------+
| Field | Type   | Label | Comment                                                   |
+------------------------------------------------------------------------------------+
| a     | string |       |                                                           |
| b     | string |       |                                                           |
| c     | bigint |       |                                                           |
+------------------------------------------------------------------------------------+
| Partition Columns:                                                                 |
+------------------------------------------------------------------------------------+
| dt    | string |                                                                   |
+------------------------------------------------------------------------------------+
| Extended Info:                                                                     |
+------------------------------------------------------------------------------------+
| TableID: 91a3395d3ef64b4d9ee1d2852755                                              |
| IsArchived: false                                                                  |
| PhysicalSize: 0                                                                    |
| FileNum: 0                                                                         |
| ClusterType: hash                                                                  |
| BucketNum: 1024                                                                    |
| ClusterColumns: [c]                                                                |
| SortColumns: [c ASC]                                                               |
+------------------------------------------------------------------------------------+
```

You can also use the following command to view the attributes of a partition in a clustering partitioned table.

```
DESC EXTENDED table_name PARTITION(pt_spec);
```

To view the details of a table, you can execute the `SELECT` statement. For more information, see [SELECT syntax](/intl.en-US/Development/SQL/Select Operation/SELECT syntax.md).

## View the CREATE TABLE statement

Syntax

```
SHOW CREATE TABLE <table_name>;
```

**Note:** You can use this statement to generate SQL DDL statements that are used to create tables. These DDL statements can be used to rebuild the table schema.

## Delete a table

Syntax

```
DROP TABLE [IF EXISTS] table_name;
```

**Note:**

-   If you do not specify `IF EXISTS` and the table that you want to delete does not exist, an error is returned. If you specify IF EXISTS, a success message is returned regardless of whether the table exists.
-   If you delete an external table, data in OSS is not deleted.

Examples

```
CREATE TABLE sale_detail_drop LIKE sale_detail;
    DROP TABLE sale_detail_drop;
    -- If the table exists, a success message is returned. If the table does not exist, an error is returned.
    DROP TABLE IF EXISTS sale_detail_drop2;
    -- A success message is returned regardless of whether the sale_detail_drop2 table exists.
```

## Rename a table

Syntax

```
ALTER TABLE table_name RENAME TO new_table_name;
```

**Note:**

-   The `RENAME` statement changes only the name of a table. Data in the table remains unchanged.
-   If an existing table has the name specified by `new_table_name`, an error is returned.
-   If the table specified by `table_name` does not exist, an error is returned.

Example

```
CREATE TABLE sale_detail_rename1 LIKE sale_detail;
ALTER TABLE sale_detail_rename1 RENAME TO sale_detail_rename2;
```

## Change the owner of a table

MaxCompute SQL allows you to execute the `CHANGEOWNER` statement to change the owner of a table. Syntax:

```
ALTER TABLE table_name CHANGEOWNER TO 'ALIYUN$xxx@aliyun.com';
```

## Modify the comment of a table

Syntax:

```
ALTER TABLE table_name SET COMMENT 'tbl comment';
```

**Note:**

-   The table specified by `table_name` must exist.
-   The maximum length of a comment is 1,024 bytes.

Example

```
ALTER TABLE sale_detail SET COMMENT 'new coments for table sale_detail';
```

You can execute the `DESC` statement to view the new `comment` after the change.

## Change LastDataModifiedTime of a table

MaxCompute SQL allows you to execute the `TOUCH` statement to change the value of `LastDataModifiedTime`. You can change the value of `LastDataModifiedTime` to the current time.

Syntax

```
ALTER TABLE table_name TOUCH;
```

**Note:**

-   If the table specified by `table_name` does not exist, an error is returned.
-   After you execute this statement to change the value of `LastDataModifiedTime`, MaxCompute determines that the table data has changed, and restarts the lifecycle of the table from the time specified by LastDataModifiedTime.

## Modify clustering attributes of a table

MaxCompute allows you to add or remove clustering attributes of a partitioned table by using the `ALTER TABLE` statement.

Syntax of the statement that is used to add hash clustering attributes of a table:

```
ALTER TABLE table_name     
[CLUSTERED BY (col_name [, col_name, ...]) [SORTED BY (col_name [ASC | DESC] [, col_name [ASC | DESC] ...])] INTO number_of_buckets BUCKETS]
```

Syntax of the statement that is used to remove hash clustering attributes of a table:

```
ALTER TABLE table_name NOT CLUSTERED;
```

If you add range clustering attributes of a table, the number of buckets is optional and can be omitted. In this case, MaxCompute automatically determines the optimal number based on the data volume. Example:

```
ALTER TABLE table_name 
    [RANGE CLUSTERED BY (col_name [, col_name, ...]) [SORTED BY (col_name [ASC | DESC] [, col_name [ASC | DESC] ...])] INTO number_of_buckets BUCKETS]
```

Syntax of the statement that is used to remove range clustering attributes of a table or partition:

```
ALTER TABLE table_name NOT CLUSTERED;
ALTER TABLE table_name partition_spec NOT CLUSTERED;
```

**Note:**

-   The `ALTER TABLE` statement can modify only the clustering attributes of a partitioned table. The clustering attributes of a non-partitioned table cannot be modified after these attributes are added. The `ALTER TABLE` statement is applicable for the tables that already exist. After new clustering attributes are added, new partitions are stored based on the added clustering attributes.
-   `ALTER TABLE` affects only the partitions created for the partitioned tables, including the partitions generated by using `INSERTOVERWRITE`. The new partition is stored based on the new clustering attributes, and the clustering attributes and storage of existing partitions remain unchanged. After you set the clustering attributes for a table, you can disable the clustering attributes and add the clustering attributes for the table again. Then, you can set clustering columns, order of columns, and number of buckets that are different from those configured before.
-   The `ALTER TABLE` statement takes effect only on the new partitions of a table. Therefore, you cannot specify a partition in this statement.

## Clear data from a non-partitioned table

You can use the TRUNCATE TABLE statement to clear data from a specified non-partitioned table. This statement cannot be executed for a partitioned table. You can execute the `ALTER TABLE table_name DROP PARTITION` statement to clear data from partitions in a partitioned table.

Syntax:

```
TRUNCATE TABLE table_name;
```

