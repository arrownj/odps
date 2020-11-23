---
keyword: [add partitions, delete partitions, add columns, modify the name of a column, modify the comment of a column]
---

# Partition and column operations

This topic describes the DDL statements that are used for operations on partitions or columns in a MaxCompute table. You can execute these statements based on your business needs.

The following table lists the DDL statements that are used for operations on partitions or columns in a MaxCompute table.

|Operation|Description|
|---------|-----------|
|[Add partitions](#section_dfj_q1b_wdb)|Adds partitions to an existing partitioned table. This operation can add only partition column values instead of partition column names.|
|[Delete partitions](#section_ahp_kbb_wdb)|Deletes partitions from an existing partitioned table.|
|[Add columns or comments](#section_k3l_4bb_wdb)|Adds columns or comments to an existing non-partitioned table or partitioned table.|
|[Modify the name of a column](#section_k3x_qbb_wdb)|Modifies the name of a column in an existing non-partitioned table or partitioned table.|
|[Modify the comment of a column](#section_ugq_pcb_wdb)|Modifies the comment of a column in an existing non-partitioned table or partitioned table.|
|[Modify the name and comment of a column at the same time](#section_vcd_scb_wdb)|Modifies the name and comment of a column in an existing non-partitioned table or partitioned table at the same time.|
|[Change LastDataModifiedTime of a table or partition](#section_mnj_5cb_wdb)|Changes LastDataModifiedTime of a non-partitioned table or a partition in a partitioned table.|
|[Change the partition column value](#section_lrr_ycb_wdb)|Changes partition column values of a partitioned table.|
|[Merge partitions](#section_ke3_j8x_fh2)|Merges multiple partitions of a partitioned table into one partition. This operation deletes the dimension information about the merged partitions and transfers data to the specified partition.|

## Add partitions

Adds partitions to an existing partitioned table. This operation can add only partition column values instead of partition column names.

-   Syntax

    ```
    -- Add one partition at a time.
    ALTER TABLE table_name ADD [IF NOT EXISTS] PARTITION partition_spec;
    -- Add multiple partitions at a time.
    ALTER TABLE table_name ADD [IF NOT EXISTS] PARTITION partition_spec [PARTITION partition_spec PARTITION partition_spec...];
    -- Format of partition_spec.
    partition_spec: (partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...)
    ```

-   Parameters
    -   table\_name: the name of the destination table to which you want to add a partition.
    -   IF NOT EXISTS: If IF NOT EXISTS is not specified and a partition with an identical name already exists, an error is returned.
    -   partition\_spec: the name of the partition you want to add. Partition names are not case-sensitive. partition\_col indicates the partition column name, and partition\_col\_value indicates the partition column value.
-   Limits
    -   A MaxCompute table can contain a maximum of 60,000 partitions.
    -   To add partition column values to a table that has multi-level partitions, you must specify all the partitions.
-   Examples

    ```
    -- Add a partition to the sale_detail table. The partition stores the sales records of Hangzhou in December 2013.
    ALTER TABLE sale_detail ADD IF NOT EXISTS PARTITION (sale_date='201312', region='hangzhou');
    -- Add a partition to the sale_detail table. The partition stores the sales records of Shanghai in December 2013.
    ALTER TABLE sale_detail ADD IF NOT EXISTS PARTITION (sale_date='201312', region='shanghai');
    -- Add two partitions to the sale_detail table. The partitions store the sales records of Hangzhou and Shanghai in December 2013.
    ALTER TABLE sale_detail ADD IF NOT EXISTS PARTITION (sale_date='201312', region='hangzhou') PARTITION (sale_date='201312', region='shanghai');
    -- Add a partition to the sale_detail table and specify only the sale_date column for this partition. The addition fails and an error is returned.
    ALTER TABLE sale_detail ADD IF NOT EXISTS PARTITION (sale_date='20111011');
    -- Add a partition to the sale_detail table and specify only the region column for this partition. The addition fails and an error is returned.
    ALTER TABLE sale_detail ADD IF NOT EXISTS PARTITION (region='shanghai');
    ```


## Delete partitions

Deletes partitions from an existing partitioned table.

MaxCompute allows you to delete partitions that meet a specified rule condition. If you want to delete one or more partitions that meet a rule condition at a time, you can use an expression to specify the condition, use the condition to match partitions, and delete the partitions at a time.

-   Syntax
    -   The condition is not specified.

        ```
        -- Delete one partition at a time.
        ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec;
        -- Delete multiple partitions at a time.
        ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec,PARTITION partition_spec[,PARTITION partition_spec....];
        -- Format of partition_spec.
        partition_spec:(partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...)
        ```

    -   The condition is specified.

        ```
        ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_filtercondition;
        -- The format of PARTITION partition_filtercondition.
        PARTITION partition_filtercondition
            : PARTITION (partition_col relational_operators partition_col_value)
            | PARTITION (scalar(partition_col) relational_operators partition_col_value)
            | PARTITION (partition_filtercondition1 AND|OR partition_filtercondition2)
            | PARTITION (NOT partition_filtercondition)
            | PARTITION (partition_filtercondition1)[,PARTITION (partition_filtercondition2), ...]
        ```

-   Parameters
    -   table\_name: the name of the partitioned table from which you want to delete a partition.
    -   IF EXISTS: If IF EXISTS is not specified and the partition does not exist, an error is returned.
    -   partition\_spec: the name of the partition you want to delete. Partition names are not case-sensitive. partition\_col indicates the partition column name, and partition\_col\_value indicates the partition column value.
    -   PARTITION partition\_filtercondition: the partition you want to delete. It is not case-sensitive.
        -   partition\_col: the partition column name.
        -   relational\_operators: the relational operator. For more information, see [Operators](/intl.en-US/Development/SQL/Operators.md).
        -   partition\_col\_value: the partition column value, which is a comparison value or regular expression. The data type of the partition column value must be the same as that of the partition column name.
        -   scalar\(\): the scalar function. The scalar function generates a scalar based on the input value, processes the value of partition\_col, and uses the relational\_operators to compare the processed value with partition\_col\_value.
        -   The condition that is used to delete partitions supports the following logical operators: NOT, AND, and OR. You can use PARTITION \(NOT partition\_filtercondition\) to obtain the complementary set of the conditions. You can use PARTITION \(partition\_filtercondition1 AND\|OR partition\_filtercondition2\) to obtain the condition that is used to match the partitions you want to delete.
        -   Multiple PARTITION partition\_filtercondition clauses are supported. If these clauses are separated by commas \(,\), OR is used for each clause to obtain the condition that is used to match the partitions you want to delete.
-   Limits
    -   Each PARTITION partition\_filtercondition clause can be used for only one partition column.
    -   If you use an expression to specify PARTITION partition\_filtercondition, only the built-in scalar function can be used for the expression.
-   Examples
    -   The condition is not specified.

        ```
        -- Delete a partition from the sale_detail table. The partition stores the sales record of Hangzhou in December 2013.
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date='201312',region='hangzhou'); 
        -- Delete two partitions from the sale_detail table. The partitions store the sales records of Hangzhou and Shanghai in December 2013.
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date='201312',region='hangzhou'),PARTITION(sale_date='201312',region='shanghai');
        ```

    -   The condition is specified.

        ```
        -- Create a partitioned table named sale_detail.
        CREATE TABLE IF NOT EXISTS sale_detail(
        shop_name     STRING,
        customer_id   STRING,
        total_price   DOUBLE)
        PARTITIONED BY (sale_date STRING);
        -- Add partitions to the table.
        ALTER TABLE sale_detail ADD IF NOT EXISTS
        PARTITION (sale_date= '201910')
        PARTITION (sale_date= '201911')
        PARTITION (sale_date= '201912')
        PARTITION (sale_date= '202001')
        PARTITION (sale_date= '202002')
        PARTITION (sale_date= '202003')
        PARTITION (sale_date= '202004')
        PARTITION (sale_date= '202005')
        PARTITION (sale_date= '202006')
        PARTITION (sale_date= '202007');
        -- Delete partitions from the table.
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date < '201911');
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date >= '202007');
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date LIKE '20191%');
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date IN ('202002','202004','202006'));
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date BETWEEN '202001' AND '202007');
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(substr(sale_date, 1, 4) = '2020');
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date < '201912' OR sale_date >= '202006');
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date > '201912' AND sale_date <= '202004');
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(NOT sale_date > '202004');
        -- Delete partitions by using a condition. The condition is specified by expressions that have the OR relationship.
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date < '201911'), PARTITION(sale_date >= '202007');
        -- Add partitions in other formats.
        ALTER TABLE sale_detail ADD IF NOT EXISTS
        PARTITION (sale_date= '2019-10-05');
        PARTITION (sale_date= '2019-10-06')
        PARTITION (sale_date= '2019-10-07');
        -- Delete partitions in batches and use regular expressions to match the partitions you want to delete.
        ALTER TABLE sale_detail DROP IF EXISTS PARTITION(sale_date RLIKE '2019-\\d+-\\d+');
        -- Create a table named region_sale_detail. The table has multi-level partitions.
        CREATE TABLE IF NOT EXISTS region_sale_detail(
        shop_name     STRING,
        customer_id   STRING,
        total_price   DOUBLE)
        PARTITIONED BY (sale_date STRING , region STRING );
        -- Add partitions.
        ALTER TABLE region_sale_detail ADD IF NOT EXISTS
        PARTITION (sale_date= '201910',region = 'shanghai')
        PARTITION (sale_date= '201911',region = 'shanghai')
        PARTITION (sale_date= '201912',region = 'shanghai')
        PARTITION (sale_date= '202001',region = 'shanghai')
        PARTITION (sale_date= '202002',region = 'shanghai')
        PARTITION (sale_date= '201910',region = 'beijing')
        PARTITION (sale_date= '201911',region = 'beijing')
        PARTITION (sale_date= '201912',region = 'beijing')
        PARTITION (sale_date= '202001',region = 'beijing')
        PARTITION (sale_date= '202002',region = 'beijing');
        -- Delete the partitions that meet the sale_date < '201911' and region = 'beijing' conditions from the table.
        ALTER TABLE region_sale_detail DROP IF EXISTS PARTITION(sale_date < '201911'),PARTITION(region = 'beijing');
        ```

        The following error is returned: `FAILED: ODPS-0130071:[1,82] Semantic analysis exception - invalid column reference region, partition expression must have one and only one column reference`

        ```
        -- PARTITION partition_filtercondition can be specified for only one partition column. The following error is returned:
        ALTER TABLE region_sale_detail DROP IF EXISTS PARTITION(sale_date < '201911' AND region = 'beijing');
        ```


## Add columns or comments

Adds columns or comments to an existing non-partitioned table or partitioned table.

-   Syntax
    -   Add columns to a table

        ```
        ALTER TABLE table_name ADD COLUMNS (col_name1 type1,col_name2 type2...) ;
        ```

    -   Add columns and their comments

        ```
        ALTER TABLE table_name ADD COLUMNS (col_name1 type1 comment 'XXX',col_name2 type2 comment 'XXX');
        ```

-   Parameters
    -   table\_name: the name of the table to which you want to add a column. You cannot specify the order of a new column in the table. By default, the new column is added as the last column.
    -   col\_name: the name of the new column.
    -   type: the data type of the new column.
    -   comment: the comment of the new column.
-   Examples

    ```
    -- Add two columns to the sale_detail table.
    ALTER TABLE sale_detail ADD COLUMNS (customer_name STRING, education BIGINT);
    -- Add two columns and their comments to the sale_detail table.
    ALTER TABLE sale_detail ADD COLUMNS (customer_name STRING comment 'Customer', education BIGINT comment 'Education');
    ```


## Modify the name of a column

Modifies the name of a column in an existing non-partitioned table or partitioned table.

-   Syntax

    ```
    ALTER TABLE table_name CHANGE COLUMN old_col_name RENAME TO new_col_name;
    ```

-   Parameters
    -   table\_name: the name of the table for which you want to modify the name of a column.
    -   old\_col\_name: the name of the column you want to modify. The column specified by `old_col_name` must already exist in the table.
    -   new\_col\_name: the new name of the column. The table does not contain a column named `new_col_name`.
-   Example

    ```
    -- Modify the name of a column in the sale_detail table.
    ALTER TABLE sale_detail CHANGE COLUMN customer_name RENAME TO customer;
    ```


## Modify the comment of a column

Modifies the comment of a column in an existing non-partitioned table or partitioned table.

-   Syntax

    ```
    ALTER TABLE table_name CHANGE COLUMN col_name COMMENT 'comment_string';
    ```

-   Parameters
    -   table\_name: the name of the table for which you want to modify the comment of a column.
    -   col\_name: the name of the column for which you want to modify the comment. The column specified by `col_name` must already exist in the table.
    -   comment\_string: the new comment. The maximum size of a comment is 1,024 bytes.
-   Example

    ```
    -- Modify the comment of a column in the sale_detail table.
    ALTER TABLE sale_detail CHANGE COLUMN customer COMMENT 'customer';
    ```


## Modify the name and comment of a column at the same time

Modifies the name and comment of a column in an existing non-partitioned table or partitioned table at the same time.

-   Syntax

    ```
    ALTER TABLE table_name CHANGE COLUMN old_col_name new_col_name column_type COMMENT 'column_comment';
    ```

-   Parameters
    -   table\_name: the name of the table for which you want to modify the name and comment of a column.
    -   old\_col\_name: the name of the column you want to modify. The column specified by `old_col_name` must already exist in the table.
    -   new\_col\_name: the new column name. The table does not contain a column named `new_col_name`.
    -   column\_type: the data type of the column, which cannot be modified.
    -   column\_comment: the new column comment. The maximum size of a comment is 1,024 bytes.
-   Example

    ```
    -- Modify the name and comment of a column in the sale_detail table.
    ALTER TABLE sale_detail CHANGE COLUMN customer customer_name string COMMENT 'Customer';
    ```


## Change LastDataModifiedTime of a table or partition

MaxCompute SQL allows you to execute the `TOUCH` statement to change `LastDataModifiedTime` of a non-partitioned table or a partition in a partitioned table. This operation changes `LastDataModifiedTime` to the current time. In this case, MaxCompute considers that the table or partition data has changed, and the new lifecycle of the table or partition starts from the time specified by LastDataModifiedTime.

-   Syntax

    ```
    -- Change LastDataModifiedTime of a non-partitioned table.
    ALTER TABLE table_name TOUCH;
    -- Change LastDataModifiedTime of a partition in a partitioned table.
    ALTER TABLE table_name TOUCH PARTITION(partition_col='partition_col_value', ...) ;
    ```

-   Parameters
    -   table\_name: the name of the table whose LastDataModifiedTime you want to change. If the table does not exist, an error is returned.
    -   partition\_col='partition\_col\_value': the column name and value of the partition whose LastDataModifiedTime you want to change. If the specified partition column name and its column value do not exist, an error is returned.
-   Examples

    ```
    -- Change LastDataModifiedTime of a non-partitioned table.
    ALTER TABLE result_table TOUCH;
    -- Change LastDataModifiedTime of a partition in a partitioned table.
    ALTER TABLE sale_detail TOUCH PARTITION(sale_date='201312');
    ```


## Change the partition column value

MaxCompute SQL allows you to execute the `RENAME` statement to change the partition column values of a partitioned table.

-   Syntax

    ```
    ALTER TABLE table_name PARTITION (partition_col1 = 'partition_col_value1', partition_col2 = 'partiton_col_value2', ...) 
    RENAME TO PARTITION (partition_col1 = 'partition_col_newvalue1', partition_col2 = 'partiton_col_newvalue2', ...) ;
    ```

-   Parameters
    -   table\_name: the name of the table whose partition column value you want to change. If the table does not exist, an error is returned.
    -   partition\_col = partition\_col\_value: the partition column name and partition column value you want to change.
    -   partition\_col = partition\_col\_newvalue: the partition column name and new partition column value.
-   Limits
    -   This operation can only change partition column values instead of partition column names.
    -   To change one or more partition column values in a table that has multi-level partitions, you must specify the column values of the partitions at each level.
-   Example

    ```
    -- Change the partition column value of the sale_detail table.
    ALTER TABLE sale_detail PARTITION (sale_date = '201312', region = 'hangzhou') RENAME TO PARTITION (sale_date = '201310', region = 'beijing');
    ```


## Merge partitions

MaxCompute SQL allows you to execute the `MERGE PARTITION` statement to merge multiple partitions of a table into one partition. This operation deletes the dimension information about the merged partitions and transfers data to the specified partition.

**Note:**

-   This operation cannot be executed for external tables and table shards. After partitions of a clustered table are merged, the clustered attributes are no longer included in the partition files.
-   You can merge a maximum of 4,000 partitions at a time.

-   Syntax

    ```
    ALTER TABLE <table_name> MERGE [IF EXISTS] PARTITION(<predicate>) [, PARTITION(<predicate2>) ...] OVERWRITE PARTITION(<fullPartitionSpec>) [PURGE];
    ```

-   Parameters
    -   table\_name: the name of the partitioned table whose partitions you want to merge.
    -   IF EXISTS: If IF EXISTS is not specified and the partition does not exist, an error is returned. If no partitions meet merge conditions after `IF EXISTS` is specified, a new partition is not generated. If the source data is modified by concurrently executing the `INSERT`, `RENAME`, and `DROP` statements, an error is returned even if `IF EXISTS` is specified.
    -   predicate: the predicate conditions that the partitions you want to merge must meet.
    -   fullPartitionSpec: the information about the destination partition that you want to merge partitions into.
-   Example

    ```
    -- Partitions and data of a partitioned table:
    odps@ project_name>SHOW PARTITIONS intpstringstringstring;
    
    ds=20181101/hh=00/mm=00
    ds=20181101/hh=00/mm=10
    ds=20181101/hh=10/mm=00
    ds=20181101/hh=10/mm=10
    
    OK
    odps@ project_name>DESC intpstringstringstring;
    +------------+------------+------------+------------+
    | value      | ds         | hh         | mm         |
    +------------+------------+------------+------------+
    | 1          | 20181101   | 00         | 00         |
    | 1          | 20181101   | 00         | 10         |
    | 1          | 20181101   | 10         | 00         |
    | 1          | 20181101   | 10         | 10         |
    +------------+------------+------------+------------+
    
    -- Merge all partitions that meet the hh='00' condition into the ds='20181101', hh='00', mm='00' partition.
    odps@ project_name>ALTER TABLE intpstringstringstring MERGE PARTITION(hh='00') OVERWRITE PARTITION(ds='20181101', hh='00', mm='00');
    
    ID = 20190404025755844g80qwa7a
    OK
    
    -- View the merged partitions.
    odps@ project_name>SHOW PARTITIONS intpstringstringstring;
    
    ds=20181101/hh=00/mm=00
    ds=20181101/hh=10/mm=00
    ds=20181101/hh=10/mm=10
    
    OK
    
    Data in two partitions that meet the hh='00' condition is merged into the ds='20181101', hh='00', mm='00' partition.
    odps@ project_name>DESC intpstringstringstring;
    +------------+------------+------------+------------+
    | value      | ds         | hh         | mm         |
    +------------+------------+------------+------------+
    | 1          | 20181101   | 00         | 00         |
    | 1          | 20181101   | 00         | 00         |
    | 1          | 20181101   | 10         | 00         |
    | 1          | 20181101   | 10         | 10         |
    +------------+------------+------------+------------+
                            
    ```

    When you execute `MERGE PARTITION` to merge partitions, you can specify multiple predicate conditions. For example, you can execute the following statement to merge all the remaining partitions to the ds='20181101', hh='00', mm='00' partition:

    ```
    odps@ project_name>ALTER TABLE intpstringstringstring MERGE IF EXISTS PARTITION(ds='20181101', hh='00', mm='00'), PARTITION(ds='20181101', hh='10', mm='00'),  PARTITION(ds='20181101', hh='10', mm='10') OVERWRITE PARTITION(ds='20181101', hh='00', mm='00') PURGE;
    
    ID = 20190404034632854g431sqzt2
    OK
    
    odps@ project_name>SHOW PARTITIONS intpstringstringstring;
    
    ds=20181101/hh=00/mm=00
    
    OK
    ```


