---
keyword: unstructured OSS data
---

# Access OSS data by using the built-in extractor

This topic describes how to access Object Storage Service \(OSS\) data by using the built-in extractor of MaxCompute.

Before you begin, make sure that you are authorized to access unstructured OSS data. For more information, see [STS authorization for OSS](/intl.en-US/Development/External table/STS authorization.md).

After you create a foreign table in MaxCompute and associate the foreign table with OSS, you can query the foreign table to access OSS data. For more information, see [Create a foreign table](#section_b23_tfb_wdb) and [Query a foreign table](#section_ljx_ggb_wdb).

**Note:** You can create, search for, query, configure, process, and analyze foreign tables by using DataWorks, which is powered by MaxCompute. For more information, see [External table]().

## Overview

To use foreign tables to access external data sources, you can use the built-in extractor of MaxCompute. For example, you can use the built-in extractor to read data stored in the standard format in [OSS](https://www.alibabacloud.com/product/oss).

**Note:** The built-in extractor of MaxCompute cannot access DATETIME data contained in text files in OSS. To access DATETIME data, define a custom extractor. For more information, see [Use a custom extractor to access DATETIME data in OSS text files](https://yq.aliyun.com/articles/725544).

Assume that a CSV file is available in [OSS](https://www.alibabacloud.com/product/oss), the endpoint is `oss-cn-shanghai-internal.aliyuncs.com`, the bucket name is `oss-odps-test`, and the file is stored in /demo/vehicle.csv.

## Create a foreign table

Execute the following statement to create a foreign table:

```
CREATE EXTERNAL TABLE IF NOT EXISTS ambulance_data_csv_external
(
vehicleId INT,
recordId INT,
patientId INT,
calls INT,
locationLatitute DOUBLE,
locationLongtitue DOUBLE,
recordTime STRING,
direction STRING
)
STORED BY 'com.aliyun.odps.CsvStorageHandler' 
WITH SERDEPROPERTIES (
 'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
) 
LOCATION 'oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/';
```

-   STORED BY: specifies the name of the storage handler. `com.aliyun.odps.CsvStorageHandler` is a built-in storage handler used to process CSV files. It defines how to read and write CSV files. You can specify this parameter as required. The logic of reading and writing CSV files is implemented by the system.
-   WITH SERDEPROPERTIES: specifies properties of the foreign table.

    odps.properties.rolearn: specifies the Alibaba Cloud Resource Name \(ARN\) of AliyunODPSDefaultRole in Resource Access Management \(RAM\). If you use Security Token Service \(STS\) to authorize MaxCompute to access OSS, this parameter is required. For more information, see [STS authorization](https://www.alibabacloud.com/help/zh/doc-detail/72777.htm?spm=a2c63.l28256.b99.180.dd684e7aE0j0gX). You can obtain the parameter value from [role details](https://ram.console.aliyun.com/#/role/detailAliyunODPSDefaultRole/info) in the RAM console.

-   LOCATION: specifies the OSS directory to query. The system reads all files in the directory by default.
    -   We recommend that you use the internal same-region endpoint of OSS to avoid fees that are incurred by OSS data flows.
    -   We recommend that you store OSS data in the region where you activate MaxCompute. MaxCompute can only be deployed in specific regions, so cross-region data connectivity is not ensured.
    -   The OSS connection format is `oss://oss-cn-shanghai-internal.aliyuncs.com/Bucket name/Directory/`. Do not add a file name after the Directory parameter. The following examples are negative:

        ```
        http://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/  -- HTTP connections are not supported.
        https://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/ -- HTTPS connections are not supported.
        oss://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo -- The connection address is invalid.
        oss://oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/vehicle.csv  -- You do not need to specify the file name.
        ```

        **Note:** By creating a foreign table, you only set up a connection to the destination OSS directory. After you delete the foreign table, data stored in the OSS directory is retained.


You can execute the following statement to view the structure of the created foreign table:

```
DESC EXTENDED <table_name>;
```

In the output, you can view basic table information, which is similar to the information returned for a created internal table. In addition to basic table information, you can view the storage handler and the OSS directory to query.

## Query a foreign table

After a foreign table is created, you can use it the same way that you use a common table. Assume that /demo/vehicle.csv contains the following data:

```
1,1,51,1,46.81006,-92.08174,9/14/2014 0:00,S
1,2,13,1,46.81006,-92.08174,9/14/2014 0:00,NE
1,3,48,1,46.81006,-92.08174,9/14/2014 0:00,NE
1,4,30,1,46.81006,-92.08174,9/14/2014 0:00,W
1,5,47,1,46.81006,-92.08174,9/14/2014 0:00,S
1,6,9,1,46.81006,-92.08174,9/14/2014 0:00,S
1,7,53,1,46.81006,-92.08174,9/14/2014 0:00,N
1,8,63,1,46.81006,-92.08174,9/14/2014 0:00,SW
1,9,4,1,46.81006,-92.08174,9/14/2014 0:00,NE
1,10,31,1,46.81006,-92.08174,9/14/2014 0:00,N
```

Execute the following sample statement to query data:

```
SELECT recordId, patientId, direction FROM ambulance_data_csv_external WHRER patientId > 25;
```

**Note:**

-   A foreign table can only be managed by using the MaxCompute SQL engine, not MaxCompute MapReduce.
-   To obtain data by using HTTPS at the underlying layer, commit the `set odps.sql.unstructured.data.oss.use.https=true;` command with SQL statements for execution.

The preceding sample command submits a job, which calls the built-in CSV extractor to read and process data from OSS. The following result is returned:

```
+------------+------------+-----------+
| recordId   | patientId  | direction |
+------------+------------+-----------+
| 1          | 51         | S         |
| 3          | 48         | NE        |
| 4          | 30         | W         |
| 5          | 47         | S         |
| 7          | 53         | N         |
| 8          | 63         | SW        |
| 10         | 31         | N         |
+------------+------------+-----------+
```

## Use the MSCK REPAIR TABLE statement to add a partition to a foreign table

MaxCompute supports automatically adding partitions to foreign tables based on the OSS directory where the data is stored. Execute the following statement to add a partition to a foreign table:

```
MSCK [REPAIR] TABLE external_table_name [ADD PARTITIONS];
```

Syntax:

-   To import data to OSS, make sure that the OSS directory is in the following format: `oss://xxx/table-location/ptname1=ptvalue1/ptname2=ptvalue2/xxx`.
-   When you create a foreign table, you must specify the partition structure.
-   After you execute the `MSCK [REPAIR] TABLE external_table_name [ADD PARTITIONS];` statement, SQL automatically parses the OSS directory to recognize the partition, and adds the partition to the foreign table.

Example:

1.  Upload data to OSS. The following figure shows the OSS directory.

    ![OSS](../images/p84543.png)

2.  Execute the following statement to create a foreign table named orc\_pt\_v0, in which the structure of the pt partition is specified:

    ```
    CREATE EXTERNAL TABLE orc_pt_v0
    (
    name STRING
    )
    PARTITIONED BY (pt bigint)
    STORED BY 'com.aliyun.odps.CsvStorageHandler' 
    WITH SERDEPROPERTIES (
    'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
    ) 
    LOCATION 'oss://xxx/odps-ext-reg-perf/orc-pt-v0';
    ```

3.  Execute the following statement to add a partition to orc\_pt\_v0:

    ```
    MSCK REPAIR TABLE orc_pt_v0 ADD PARTITIONS;
    -- In this case, the MSCK statement is equivalent to the following three statements:
    ALTER TABLE orc_pt_v0 ADD PARTITION (pt=1);
    ALTER TABLE orc_pt_v0 ADD PARTITION (pt=10);
    ALTER TABLE orc_pt_v0 ADD PARTITION (pt=100);
    ```


## Read CSV or TSV files compressed in GZIP format

MaxCompute can read CSV and TSV files in GZIP format from OSS only by using the built-in extractor. The main difference between reading non-compressed and compressed files is the property specified by SERDEPROPERTIES.

**Note:** If the data file to be read is an archived object in OSS, log on to the OSS console to restore the object. For more information, see the [OSS documentation](/intl.en-US/Product Introduction/What is OSS?.md).

Use the following statement to create a foreign table:

```
CREATE EXTERNAL TABLE IF NOT EXISTS ambulance_data_csv_external
(
vehicleId BIGINT,
recordId BIGINT,
patientId BIGINT,
calls BIGINT,
locationLatitute DOUBLE,
locationLongtitue DOUBLE,
recordTime STRING,
direction STRING
)
STORED BY 'com.aliyun.odps.CsvStorageHandler'
WITH SERDEPROPERTIES (
 'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
 [,'odps.text.option.gzip.input.enabled'='true']
 [,'name3'='value3']
)
LOCATION 'oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/Demo/SampleData/CSV/AmbulanceData/';
```

The following table describes the properties supported by SERDEPROPERTIES.

|Property|Valid value|Default value|Description|
|:-------|:----------|:------------|:----------|
|odps.text.option.gzip.input.enabled|-   True
-   False

|False|Specifies whether to compress the file before data read.|
|odps.text.option.gzip.output.enabled|-   True
-   False

|False|Specifies whether to compress the file before data write.|
|odps.text.option.header.lines.count|Non-negative integers|0|Specifies the number of lines to skip in the file.|
|odps.text.option.null.indicator|Strings|""|Specifies the strings in the file to be parsed as NULL in SQL. For example, if you specify `\N` to represent NULL, `a,\N,b` is parsed as `a, NULL, b` in SQL.|
|odps.text.option.ignore.empty.lines|-   True
-   False

|True|Specifies whether to ignore blank lines.|
|odps.text.option.encoding|-   UTF-8
-   UTF-16
-   US-ASCII
-   GBK

|UTF-8|Specifies the encoding format of the file.|
|odps.text.option.delimiter|Single characters|,|Specifies the column delimiter of the file.|
|odps.text.option.use.quote|-   True
-   False

|False|Specifies whether to recognize column delimiters in the CSV file if it uses double quotation marks \(`"`\) as column delimiters. Fields in the CSV file that contain a carriage return/line feed \(CR/LF\) pair, double quotation mark \("\), or comma \(,\) for separating multiple values must be enclosed in double quotation marks. Note that if a field contains a double quotation mark \(`"`\), replace the double quotation mark \(\)" with two double quotation marks \(`"`\) for escaping.|

**Note:**

If you need to read data from and write data to a foreign table associated with compressed OSS data, set both odps.text.option.gzip.input.enabled and odps.text.option.gzip.output.enabled to True to create the foreign table.

