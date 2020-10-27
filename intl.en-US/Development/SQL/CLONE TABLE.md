---
keyword: [clone table, CLONE TABLE]
---

# CLONE TABLE

This topic describes how to use the CLONE TABLE statement. The CLONE TABLE statement is used to clone data from one table to another. This statement improves efficiency of data migration.

## Limits

-   The schema of the destination table must be compatible with that of the source table.
-   The CLONE TABLE statement can be executed for partitioned tables and non-partitioned tables. This statement cannot be executed for hash clustering tables.
-   If a destination table is created before the CLONE TABLE statement is executed, data in a maximum of 10,000 partitions can be cloned at a time.
-   If a destination table is not created before the CLONE TABLE statement is executed, the number of partitions from which you can clone data at a time is not limited. This ensures atomicity.
-   You can execute the CLONE TABLE statement for up to seven times in the same non-partitioned table or in the same partition of a partitioned table.
-   You cannot execute the CLONE TABLE statement for projects across regions.

## Syntax

```
CLONE TABLE <[src_project_name.]src_table_name> [PARTITION(spec), ...]
 TO <[dest_project_name.]desc_table_name> [IF EXISTS (OVERWRITE | IGNORE)] ;
```

## Description

The CLONE TABLE statement is used to clone data from the src\_table\_name table to the desc\_table\_name table.

**Note:** After you clone data to a destination table, we recommend that you verify accuracy of the data. For example, you can execute the `SELECT COUNT` command to view the number of rows in the destination table or execute the `DESC` command to view the table size.

## Parameters

-   src\_table\_name: the name of the source table.
-   src\_project\_name: the name of the project to which the source table belongs. If this parameter is not specified, the name of the current project is used by default.
-   desc\_table\_name: the name of the destination table.
    -   If a destination table is not created, the table is created by using the CREATE TABLE LIKE statement when you execute the CLONE TABLE statement.
    -   If a destination table is created and IF EXISTS OVERWRITE is specified, data in the partitions of the destination table is overwritten when you execute the CLONE TABLE statement.
    -   If a destination table is created and IF EXISTS IGNORE is specified, existing partitions in the table are skipped and data in these partitions is not overwritten when you execute the CLONE TABLE statement.
-   dest\_project\_name: the name of the project to which the destination table belongs. If this parameter is not specified, the name of the current project is used by default.

## Examples

Assume that partitioned table srcpart\_copy and non-partitioned table src\_copy are source tables. The two tables have the following metadata:

```
odps@ multi>READ srcpart_copy;
+------------+------------+------------+------------+
| key        | value      | ds         | hr         |
+------------+------------+------------+------------+
| 1          | ok49       | 2008-04-09 | 11         |
| 1          | ok48       | 2008-04-08 | 12         |
+------------+------------+------------+------------+
odps@ multi>READ src_copy;
+------------+------------+
| key        | value      |
+------------+------------+
| 1          | ok         |
+------------+------------+
```

-   Clone all data from non-partitioned table src\_copy to destination table src\_clone.

    ```
    odps@ multi>CLONE TABLE src_copy TO src_clone;
    ID = 2019102303024544g2540cdv2
    OK
    //Query data in destination table src_clone after the data clone and verify accuracy of the data.
    odps@ multi>SELECT * FROM src_clone;
    ```

-   Clone data from a specified partition of partitioned table srcpart\_copy to destination table srcpart\_clone.

    ```
    odps@ multi>CLONE TABLE srcpart_copy PARTITION(ds="2008-04-09", hr='11') TO srcpart_clone IF EXISTS OVERWRITE;
    ID = 20191023030534986g4540cdv2
    OK
    //Query data in destination table srcpart_clone after the data clone and verify accuracy of the data.
    odps@ multi>SELECT * FROM srcpart_clone;
    ```

-   Clone all data from partitioned table srcpart\_copy to destination table srcpart\_clone and skip data in existing partitions of the destination table.

    ```
    odps@ multi>CLONE TABLE srcpart_copy TO srcpart_clone IF EXISTS IGNORE;
    ID = 20191023030619196g5540cdv2
    OK
    //Query data in destination table srcpart_clone after the data clone and verify accuracy of the data.
    odps@ multi>SELECT * FROM srcpart_clone;
    ```

-   Clone all data from partitioned table srcpart\_copy to destination table srcpart\_clone2.

    ```
    odps@ multi>CLONE TABLE srcpart_copy TO srcpart_clone2;
    ID = 20191023030825186g6540cdv2
    OK
    //Query data in destination table srcpart_clone2 after the data clone and verify accuracy of the data.
    odps@ multi>CLONE TABLE srcpart_clone2;
    ```


