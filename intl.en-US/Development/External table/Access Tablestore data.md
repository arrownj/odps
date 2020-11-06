# Access Tablestore data

This topic describes how to import data from Tablestore to MaxCompute. This allows you to create seamless connections between multiple data sources.

Tablestore is a NoSQL database service that is built on the Apsara distributed operating system. It allows you to store and access large amounts of structured data in real time. For more information, see [Tablestore documentation](https://www.alibabacloud.com/help/doc-detail/27280.html).

You can create, search for, configure, and process external tables in the DataWorks console. You can also query and analyze data in external tables. For more information, see [External table]().

You must ensure the connectivity between MaxCompute and Tablestore. If you use Alibaba Cloud MaxCompute to access a Tablestore instance, we recommend that you use the internal endpoint of the Tablestore instance. The endpoint ends with ots-internal.aliyuncs.com. Example: tablestore://odps-ots-dev.cn-shanghai.ots-internal.aliyuncs.com.

Both Tablestore and MaxCompute have their own data type systems. The following table lists the mapping between data types supported by Tablestore and MaxCompute.

|MaxCompute|Tablestore|
|:---------|:---------|
|STRING|STRING|
|BIGINT|INTEGER|
|DOUBLE|DOUBLE|
|BOOLEAN|BOOLEAN|
|BINARY|BINARY|

## STS authorization

A secure authorization channel is required for MaxCompute to access Tablestore data. MaxCompute uses Alibaba Cloud Resource Access Management \(RAM\) and Security Token Service \(STS\) to ensure the security of data access.

You can use one of the following methods to grant permissions:

-   If MaxCompute and Tablestore are under the same Alibaba Cloud account, log on to the Alibaba Cloud Management Console and [click here to perform one-click authorization](https://ram.console.aliyun.com/?spm=5176.100239.blogcont281191.24.uJg9dR#/role/authorize?request=%7B%22Requests%22:%20%7B%22request1%22:%20%7B%22RoleName%22:%20%22AliyunODPSDefaultRole%22,%20%22TemplateId%22:%20%22DefaultRole%22%7D%7D,%20%22ReturnUrl%22:%20%22https:%2F%2Fram.console.aliyun.com%2F%22,%20%22Service%22:%20%22ODPS%22%7D).
-   Customize authorization.
    1.  Authorize access from MaxCompute to Tablestore in the RAM console.

        Log on to the [RAM console](https://account.alibabacloud.com/login/login.html) and create a role such as AliyunODPSDefaultRole or AliyunODPSRoleForOtherUser. If MaxCompute and Tablestore are not under the same account, log on to the RAM console by using the Tablestore account.

    2.  Modify policy settings.

        ```
        -- If MaxCompute and Tablestore are under the same account, use the following policy configuration:
        {
        "Statement": [
        {
         "Action": "sts:AssumeRole",
         "Effect": "Allow",
         "Principal": {
           "Service": [
             "odps.aliyuncs.com"
           ]
         }
        }
        ],
        "Version": "1"
        }
        -- If MaxCompute and Tablestore are not under the same account, use the following policy configuration:
        {
        "Statement": [
        {
         "Action": "sts:AssumeRole",
         "Effect": "Allow",
         "Principal": {
           "Service": [
             "ID of the Alibaba Cloud account of MaxCompute@odps.aliyuncs.com"
           ]
         }
        }
        ],
        "Version": "1"
        }
        ```

        **Note:** You can click your profile picture in the upper-right corner of the page that appears to view the ID.

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5988564061/p49673.jpg)

    3.  Edit the AliyunODPSRolePolicy authorization policy for the role.

        ```
        {
        "Version": "1",
        "Statement": [
        {
         "Action": [
           "ots:ListTable",
           "ots:DescribeTable",
           "ots:GetRow",
           "ots:PutRow",
           "ots:UpdateRow",
           "ots:DeleteRow",
           "ots:GetRange",
           "ots:BatchGetRow",
           "ots:BatchWriteRow",
           "ots:ComputeSplitPointsBySize"
         ],
         "Resource": "*",
         "Effect": "Allow"
        }
        ]
        }
        -- You can also customize other permissions.
        ```

    4.  Grant the AliyunODPSRolePolicy permission to the role.

## Create an external table

In MaxCompute, you can use external tables to import Tablestore data to the meta system of MaxCompute for processing. This section describes the related concepts and how to connect MaxCompute to Tablestore.

Execute the following statement to create an external table:

```
DROP TABLE IF EXISTS ots_table_external;
CREATE EXTERNAL TABLE IF NOT EXISTS ots_table_external
(
odps_orderkey bigint,
odps_orderdate string,
odps_custkey bigint,
odps_orderstatus string,
odps_totalprice double
)
STORED BY 'com.aliyun.odps.TableStoreStorageHandler' -- (1)
WITH SERDEPROPERTIES ( -- (2)
'tablestore.columns.mapping'=':o_orderkey,:o_orderdate,o_custkey, o_orderstatus,o_totalprice', -- 1
'tablestore.table.name'='ots_tpch_orders' -- 2
'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole' --3
)
LOCATION 'tablestore://odps-ots-dev.cn-shanghai.ots-internal.aliyuncs.com'; -- (3)
```

Description:

-   `com.aliyun.odps.TableStoreStorageHandler` is a built-in MaxCompute storage handler that is used to process Tablestore data. It defines the interaction between MaxCompute and Tablestore. The related logic is implemented by MaxCompute.
-   `SERDEPROPERITES` provides parameter options. If you use TableStoreStorageHandler, you must specify tablestore.columns.mapping, tablestore.table.name, and odps.properties.rolearn.
    1.  tablestore.columns.mapping: the columns of the Tablestore table that MaxCompute needs to access. The columns include primary key columns and attribute columns.
        -   The column whose name starts with a colon \(:\) is a primary key column, such as `:o_orderkey` and `:o_orderdate`. Other columns are attribute columns.
        -   A Tablestore table supports a maximum of four primary key columns. The data types of these columns must be STRING, INTEGER, or BINARY. The first primary key column is the partition key.
        -   If you specify column mappings, you must specify all primary key columns of the Tablestore table. You only need to specify the attribute columns that MaxCompute needs to access instead of specifying all attribute columns. The specified attribute columns must be in the Tablestore table. Otherwise, errors are returned when you query the new external table.
    2.  tablestore.table.name: the name of the Tablestore table that MaxCompute needs to access. If the table name does not exist in Tablestore, an error is returned. MaxCompute does not create a Tablestore table.
    3.  odps.properties.rolearn: the ARN of AliyunODPSDefaultRole in RAM. You can obtain the ID on the **RAM Roles** page of the RAM console.
-   LOCATION specifies the name and endpoint of the Tablestore instance. You must complete RAM or STS authorization to ensure secure access to Tablestore data.

You can execute the following statement to view the structure of the new external table:

```
desc extended <table_name>;
```

In the execution result, Extended Info includes external table information such as the storage handler and location in addition to basic information.

## Query an external table

After you create an external table, Tablestore data is imported to MaxCompute. Then, you can use MaxCompute SQL syntax to access Tablestore data. Example:

```
SELECT odps_orderkey, odps_orderdate, SUM(odps_totalprice) AS sum_total
FROM ots_table_external
WHERE odps_orderkey > 5000 AND odps_orderkey < 7000 AND odps_orderdate >= '1996-05-03' AND odps_orderdate < '1997-05-01'
GROUP BY odps_orderkey, odps_orderdate
HAVING sum_total> 400000.0;
```

If you access Tablestore data by using MaxCompute SQL statements, all operations, such as the selection of column names, are completed in MaxCompute. In the preceding example, the column names are odps\_orderkey and odps\_totalprice rather than the name of the primary key column \(o\_orderkey\) or attribute column \(o\_totalprice\) in the original Tablestore table. Mapping is defined in the DDL statement that is used to create the external table. You can also retain the names of the primary key columns and attribute columns in the original Tablestore table as needed.

If you want to compute one piece of data multiple times, you can import the data from Tablestore to an internal table of MaxCompute. This way, you do not need to read the data from Tablestore every time.

```
CREATE TABLE internal_orders AS
SELECT odps_orderkey, odps_orderdate, odps_custkey, odps_totalprice
FROM ots_table_external
WHERE odps_orderkey > 5000 ;
```

internal\_orders is a MaxCompute table that supports all the features of a MaxCompute internal table. The internal\_orders table uses the efficiently compressed column store and includes complete internal macro data and statistical information. Tables are stored in MaxCompute, which delivers faster data access than Tablestore. This method is suitable for data that needs to be computed multiple times.

## Export data from MaxCompute to Tablestore

**Note:** MaxCompute does not create external tables in Tablestore. Before you export data to Tablestore table, make sure that the table exists. Otherwise, an error is reported.

An external table named ots\_table\_external is created to allow MaxCompute to access the ots\_tpch\_orders table in Tablestore. The data is also stored in the internal\_orders table of MaxCompute. If you want to process data in the internal\_orders table and export the processed data to Tablestore, execute the `INSERT OVERWRITE TABLE` statement. Example:

```
INSERT OVERWRITE TABLE ots_table_external
SELECT odps_orderkey, odps_orderdate, odps_custkey, CONCAT(odps_custkey, 'SHIPPED'), CEIL(odps_totalprice)
FROM internal_orders;
```

**Note:** If the data in a MaxCompute table is sorted based on primary keys, data is written to a single partition of the Tablestore table. In this case, you cannot fully utilize distributed write operations. We recommend that you use `DISTRIBUTE BY rand()` to first scatter the data. Example:

```
INSERT OVERWRITE TABLE ots_table_external
SELECT odps_orderkey, odps_orderdate, odps_custkey, CONCAT(odps_custkey, 'SHIPPED'), CEIL(odps_totalprice)
FROM (SELECT * FROM internal_orders DISTRIBUTE BY rand()) t;
```

Tablestore is a NoSQL data storage service that stores data in the key-value pair format. Data outputs from MaxCompute affect only the rows that include the primary keys of the Tablestore table. In this example, only the rows that include odps\_orderkey and odps\_orderdate are affected. Only the attribute columns that are specified when you create the ots\_table\_external table are updated. The columns that are not included in the external table are not modified.

**Note:**

-   If the size of data that you write from MaxCompute to Tablestore is greater than 4 MB at a time, you must remove the excess data and then write data to Tablestore again. In this case, an error may occur.

    ```
    ODPS-0010000:System internal error - Output to TableStore failed with exception:
    TableStore BatchWrite request id XXXXX failed with error code OTSParameterInvalid and message:The total data size of BatchWriteRow request exceeds the limit
    ```

-   It is considered a single operation to write multiple data entries at a time or by row. For more information, see [BatchWriteRow](https://www.alibabacloud.com/help/doc-detail/27311.html). If you want to write large amounts of data at a time, you can write the data by row.
-   If you write multiple data entries at a time, make sure that you do not write duplicate rows. If duplicate rows exist, the following error may occur:

    ```
    ErrorCode: OTSParameterInvalid, ErrorMessage: The input parameter is invalid 
    ```

    For more information, see [What do I do if OTSParameterInvalid is reported when I use BatchWriteRow to submit 100 data entries at a time?](https://www.alibabacloud.com/help/doc-detail/38586.html)


