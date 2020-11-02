---
keyword: OSS非结构化数据
---

# 内置Extractor访问OSS

本文为您介绍如何使用内置Extractor在MaxCompute上访问OSS的数据。

在访问OSS结构化数据前，您需要完成授权，详情请参见[OSS的STS模式授予权限](/intl.zh-CN/开发/外部表/STS模式授权.md)。

创建外部表后，即可通过查询外部表直接访问OSS上存储的数据。详情请参见[创建外部表](#section_b23_tfb_wdb)和[查询外部表](#section_ljx_ggb_wdb)。

**说明：** 您也可以通过DataWorks配合MaxCompute对外部表进行可视化创建、搜索、查询、配置、加工和分析，详情请参见[外部表]()。

## 功能介绍

通过外部表访问外部数据源时，您可以使用MaxCompute内置的Extractor，读取按照约定格式存储的[OSS](https://www.alibabacloud.com/product/oss)数据。

**说明：** 内置Extractor不支持访问OSS文本文件的DATETIME类型数据。如果您有此需求请自定义Extractor，详情请参见[MaxCompute自定义Exractor访问OSS文本文件DATETIME类型数据](https://yq.aliyun.com/articles/725544)。

假设有一份CSV数据存在[OSS](https://www.alibabacloud.com/product/oss)上，Endpoint为`oss-cn-shanghai-internal.aliyuncs.com`，Bucket为`oss-odps-test`，数据文件的存放路径为/demo/vehicle.csv。

## 创建外部表

创建外部表命令示例如下。

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

-   STORED BY：指定内置的StorageHandler。`com.aliyun.odps.CsvStorageHandler`是内置的处理CSV格式文件的StorageHandler，定义了如何读或写CSV文件。您只需要指定该参数，相关逻辑已经由系统实现。
-   WITH SERDEPROPERTIES：指定外部表相关参数。

    odps.properties.rolearn：指定RAM中AliyunODPSDefaultRole的ARN信息。当关联OSS权限使用STS模式授权时，您需要指定此参数，详情请参见[STS模式授权](https://www.alibabacloud.com/help/zh/doc-detail/72777.htm?spm=a2c63.l28256.b99.180.dd684e7aE0j0gX)。您可以通过RAM控制台中的[角色详情](https://ram.console.aliyun.com/#/role/detailAliyunODPSDefaultRole/info)获取。

-   LOCATION：指定需要读取数据的OSS目录，系统会默认读取该目录下所有的文件。
    -   建议您使用OSS提供的内网域名，否则将产生OSS流量费用。
    -   建议您存放数据的OSS区域对应您开通MaxCompute服务的区域。由于MaxCompute只在部分区域部署，跨区域的数据连通性可能存在问题。
    -   OSS的目录格式为`oss://oss-cn-shanghai-internal.aliyuncs.com/Bucket名称/目录名称/`。目录后不加文件名称，如下为错误用法。

        ```
        http://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/  -- 不支持HTTP连接。
        https://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/ -- 不支持HTTPS连接。
        oss://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo -- 连接地址错误。
        oss://oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/vehicle.csv  -- 不需要指定文件名。
        ```

        **说明：** 外部表只是在系统中记录了与OSS目录的关联。当删除外部表时，不会删除对应的LOCATION数据。


您可以执行如下命令，查看创建好的外部表结构信息。

```
DESC EXTENDED <table_name>;
```

返回结果包含和内部表一样的基础信息、外部表的StorageHandler和Location等信息。

## 查询外部表

外部表创建成功后，您可以像使用普通表一样使用外部表。假设/demo/vehicle.csv数据如下。

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

执行如下命令。

```
SELECT recordId, patientId, direction FROM ambulance_data_csv_external WHRER patientId > 25;
```

**说明：**

-   您只能通过MaxCompute SQL操作外部表，不能通过MaxCompute MapReduce操作外部表。
-   若需要底层通过HTTPS方式获取数据，您可以将命令`set odps.sql.unstructured.data.oss.use.https=true;`与SQL语句一起提交执行。

上述示例命令会提交一个作业，调用内置CSV Extractor，从OSS读取并处理数据。返回结果如下。

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

## 外部表分区自动补全（MSCK REPAIR TABLE）

MaxCompute支持根据数据所在的OSS文件路径自动补全外部表分区，语法格式如下。

```
MSCK [REPAIR] TABLE external_table_name [ADD PARTITIONS];
```

语法说明：

-   将数据导入OSS时，OSS文件路径需要符合格式`oss://xxx/table-location/ptname1=ptvalue1/ptname2=ptvalue2/xxx`。
-   建立外部表时，需要指定分区的结构。
-   执行`MSCK [REPAIR] TABLE external_table_name [ADD PARTITIONS];`时，SQL会自动解析OSS的目录结构，自动识别分区，并为外部表添加分区信息。

示例

1.  上传数据至OSS，OSS文件路径如下。

    ![OSS](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/1192659951/p84543.png)

2.  执行如下命令创建外部表orc\_pt\_v0，外部表中需要指定分区pt的结构。

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

3.  执行如下命令自动补全外部表orc\_pt\_v0的分区。

    ```
    MSCK REPAIR TABLE orc_pt_v0 ADD PARTITIONS;
    --该场景中，MSCK语法等价于执行如下三条语句
    ALTER TABLE orc_pt_v0 ADD PARTITION (pt=1);
    ALTER TABLE orc_pt_v0 ADD PARTITION (pt=10);
    ALTER TABLE orc_pt_v0 ADD PARTITION (pt=100);
    ```


## 读取GZIP格式压缩的CSV或TSV数据

MaxCompute只支持通过内置Extractor读取OSS上以GZIP格式压缩的CSV或TSV数据。与读取非压缩文件相比，主要区别是SERDEPROPERTIES指定的属性。

**说明：** 如果OSS上的数据文件为归档文件，请登录[OSS](/intl.zh-CN/产品简介/什么是对象存储OSS.md)，解冻文件。

创建外部表的命令示例如下。

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

SERDEPROPERTIES支持的属性项如下。

|属性名|属性值|默认值|说明|
|:--|:--|:--|:-|
|odps.text.option.gzip.input.enabled|-   True
-   False

|False|打开或关闭读压缩。|
|odps.text.option.gzip.output.enabled|-   True
-   False

|False|打开或关闭写压缩。|
|odps.text.option.header.lines.count|非负整数|0|跳过文本文件前N行。|
|odps.text.option.null.indicator|字符串|空字符串|指定文本中的哪种字符串会被解析为SQL中的NULL。例如`\N`指代NULL，则`a,\N,b`会解析为`a, NULL, b`。|
|odps.text.option.ignore.empty.lines|-   True
-   False

|True|指定是否忽略空行。|
|odps.text.option.encoding|-   UTF-8
-   UTF-16
-   US-ASCII
-   GBK

|UTF-8|指定文本的字符编码。|
|odps.text.option.delimiter|单个字符|英文逗号（,）|指定文本的列分隔符。|
|odps.text.option.use.quote|-   True
-   False

|False|当CSV某个字段中包含换行（CRLF）、双引号（需要在`"`前再加`"`转义）或英文逗号（分隔符）时，整个字段必须用双引号（" "）括起来作为列分隔符。该参数指定是否识别CSV的列分隔符`"`。|

**说明：**

当关联OSS压缩数据的外部表时，如果您需要同时执行读或写操作，在创建外部表时，请将odps.text.option.gzip.input.enabled和odps.text.option.gzip.output.enabled属性设置为True。

