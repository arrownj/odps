---
keyword: [table schema, data storage unit, table]
---

# Table

Tables are the units that are used to store data in MaxCompute. Logically, a table is a two-dimensional structure that consists of rows and columns. Each row represents a record. Each column represents a field whose values are of the same data type. One record can contain one or more columns. The column names and data types constitute the schema of the table.

Tables are the input and output objects of all computing tasks in MaxCompute. You can create a table, delete a table, and import data to a table. For more information, see [Table operations](/intl.en-US/Development/Common commands/Table-level operations.md).

**Note:** The Data Map module of DataWorks allows you to create and organize MaxCompute tables, manage data lifecycles, modify table schemas, and manage permissions on tables, resources, or functions. For more information, see [Overview]().

MaxCompute V2.0 or later supports internal tables and foreign tables.

-   Data of internal tables is stored in MaxCompute. Columns in internal tables can be of any [data types](/intl.en-US/Development/Data types/Data type editions.md) that are supported by MaxCompute.
-   Data of foreign tables is not stored in MaxCompute. Instead, the data can be stored in [Object Storage Service \(OSS\)](https://www.alibabacloud.com/product/oss) or [Tablestore](https://www.alibabacloud.com/product/ots). MaxCompute records only metadata of foreign tables. You can use foreign tables of MaxCompute to process unstructured data that is stored in OSS or Tablestore, such as video, audio, genetic, meteorological, or geographic data.

**Note:**

Note the following points when you use DUAL tables:

-   Unlike databases such as Oracle, MaxCompute does not automatically create DUAL tables.
-   If you are used to using DUAL tables for testing, you can run the following commands to create a DUAL table.

    ```
    --- Enable MaxCompute V2.0 data types.
    set odps.sql.type.system.odps2=true;
    --- Create an empty table named DUAL with only one field.
    CREATE TABLE IF NOT EXISTS DUAL (DUMMY VARCHAR(1));
    ```

-   DUAL tables are used in the same way as DUAL tables of Oracle. For example, you can execute the `SELECT getdate() FROM DUAL;` statement.

