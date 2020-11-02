---
keyword: RDS
---

# RDS数据源外部表

RDS（Relational Database Service）是阿里云用户主要使用的服务，需要通过内网域名访问。如果您需要通过MaxCompute将数据加载至RDS的表中，可参考该文档进行操作。本文为您介绍如何在VPC网络环境下基于RDS数据源创建外部表并写入数据。

执行本文操作前，您需要先开通MaxCompute和云数据库RDS间的网络连接，详情请参见[网络开通流程](/cn.zh-CN/开发/外部表/网络开通流程.md)。

## 注意事项

-   只有华北2（北京）、华东2（上海）、华北3（张家口）和华东1（杭州）区域开通了SmartNAT或Private Access网络连通方案（VPC专线方案），仅这四个区域可以创建RDS数据源外部表，其他区域暂不支持。
-   暂不支持PrivateZone域名。

## 使用RDS数据源创建MaxCompute的外部表并加载数据（VPC环境）

使用RDS数据源创建MaxCompute外部表的步骤如下：

1.  登录RDS数据库，执行建表语句并插入数据。操作详情请参见[通过DMS登录RDS数据库](/cn.zh-CN/RDS MySQL 数据库/数据库连接/通过DMS登录RDS数据库.md)。

    建表示例如下：

    ```
    CREATE TABLE `rds_mc_external` (
      `id` int(11) DEFAULT NULL,
      `name` varchar(32) DEFAULT NULL
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    INSERT INTO `rds_mc_external`(`id` ,`name` ) VALUES(1,"张三");
    INSERT INTO `rds_mc_external`(`id` ,`name` ) VALUES(1,"zhangsan");
    ```

2.  登录[MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)，设置MaxCompute需要访问的外部表所属RegionID和VPC ID。

    命令语法如下：

    ```
    --注册RDS数据源的VPC属性。
    setproject odps.security.outbound.destination=<RegionID>_<VPC ID>[*]; 
    ```

    -   RegionID：VPC网络所在的区域ID。详情请参见[RegionID](/cn.zh-CN/开发/常用命令/项目空间操作.md)。
    -   VPC ID：VPC网络ID号。
    -   \[\*\]：通配符，表示VPC网络下的所有IP和端口都加入MaxCompute白名单。
    命令示例如下：

    ```
    setproject odps.security.outbound.destination=cn-beijing_vpc-2ze7cqx2bqodp9ri1****[*];
    ```

3.  在MaxCompute客户端创建映射RDS数据源的外部表。您可以通过如下两种方式进行操作：

    -   创建MaxCompute外部表的字段和RDS中表的字段列名完全对应。
        1.  在MaxCompute客户端创建外部表，表字段与RDS中表的字段列名完全对应。DDL建表语句如下：

            ```
            --打开外部表VPC支持，必须携带。
            set odps.sql.external.net.vpc=true;
            --创建外部表。
            CREATE EXTERNAL TABLE <mc_vpc_rds_external> (
              <col1_name> <data_type>,
              <col2_name> <data_type>
            )
            STORED BY 'com.aliyun.odps.jdbc.JdbcStorageHandler'  --处理JDBC连接类数据源的Handler。
             WITH SERDEPROPERTIES (
             'odps.external.net.vpc'='true',
             'odps.vpc.id'='<VPC ID>',
             'odps.vpc.access.ips'='<realm_name:port>',
             'odps.vpc.region'='<RegionID>'
             )
            location '<jdbc:mysql://realm_name:port/rds_database_name?useSSL=false&user=user_name&password=password_value@&table=rds_mc_external>' 
            TBLPROPERTIES(
             'mcfed.mapreduce.jdbc.input.query'='<SELECT * FROM rds_database_name.rds_mc_external>'
            )
            ;
            ```

            -   mc\_vpc\_rds\_external：待创建外部表的名称。
            -   col\_name：外部表的列名称。
            -   data\_type：列的数据类型。
            -   WITH SERDEPROPERTIES：
                -   'odps.external.net.vpc'='true'：表明外部表的数据源处于VPC网络。
                -   'odps.vpc.id'='<VPC ID\>'：VPC网络ID号。
                -   'odps.vpc.access.ips'='<realm\_name:port\>'：RDS数据源外部表所属VPC域名及端口。
                -   'odps.vpc.region'='<RegionID\>'：VPC网络所在的区域ID。详情请参见[RegionID](/cn.zh-CN/开发/常用命令/项目空间操作.md)。
            -   jdbc:mysql://realm\_name:port/rds\_database\_name?useSSL=false&user=user\_name&password=password\_value@&table=rds\_mc\_external：连接RDS数据源表的连接字符串。user\_name和password\_value是RDS数据库的账号和密码。
            -   TBLPROPERTIES：
                -   'mcfed.mapreduce.jdbc.input.query'='<SELECT \* FROM rds\_database\_name.rds\_mc\_external\>'：新建外部表字段必须与RDS数据源表字段一致。
        2.  向新建的MaxCompute表中插入数据。

            命令示例如下：

            ```
            --打开外部表VPC支持，必须携带。
            set odps.sql.external.net.vpc=true;
            --插入数据。
            INSERT INTO table mc_vpc_rds_external values(2,"zhagnsan");
            ```

        3.  查询数据插入结果。

            命令示例如下：

            ```
            --打开外部表VPC支持，必须携带。
            set odps.sql.external.net.vpc=true;
            --查询数据插入结果。
            SELECT * FROM mc_vpc_rds_external;
            ```

    -   创建MaxCompute外部表的字段和RDS表指定的字段列名进行映射。
        1.  在MaxCompute客户端创建外部表，表字段与RDS表中指定的字段列名进行映射。建表DDL语句如下：

            ```
            --打开外部表VPC支持，必须携带。
            set odps.sql.external.net.vpc=true;
            --创建外部表。
            CREATE EXTERNAL TABLE <mc_vpc_rds_external_mapping> (
              <col1_name> <data_type>,
              <col2_name> <data_type>
            )
            STORED BY 'com.aliyun.odps.jdbc.JdbcStorageHandler'    --处理JDBC连接类数据源的Handler。
             WITH SERDEPROPERTIES (
             'odps.external.net.vpc'='true',
             'odps.vpc.id'='<VPC ID>',
             'odps.vpc.access.ips'='<realm_name:port>',
             'odps.vpc.region'='<RegionID>'
             )
            location '<jdbc:mysql://realm_name:port/rds_database_name?useSSL=false&user=user_name&password=password_value@&table=rds_mc_external>'
            TBLPROPERTIES(
             'odps.federation.jdbc.colmapping'='<新建外部表列名:RDS表列名>',
             'mcfed.mapreduce.jdbc.input.query'='<SELECT * FROM rds_database_name.rds_mc_external>'
            )
            ;
            ```

            -   mc\_vpc\_rds\_external\_mapping：待创建外部表的名称。
            -   col\_name：外部表的列名称。
            -   data\_type：列的数据类型。
            -   WITH SERDEPROPERTIES：
                -   'odps.external.net.vpc'='true'：表明外部表的数据源处于VPC网络。
                -   'odps.vpc.id'='<VPC ID\>'：VPC网络ID号。
                -   'odps.vpc.access.ips'='<realm\_name:port\>'：RDS数据源外部表所属VPC域名及端口。
                -   'odps.vpc.region'='<RegionID\>'：VPC网络所在的区域ID。
            -   TBLPROPERTIES：
                -   'odps.federation.jdbc.colmapping'='<新建外部表列名:RDS表列名\>'：MaxCompute外部表字段与RDS数据源表字段的映射关系。对应方法为`外部表字段1:数据源表字段1,...外部表字段n:数据源表字段n`。
                -   'mcfed.mapreduce.jdbc.input.query'='<SELECT \* FROM rds\_database\_name.rds\_mc\_external\>'：新建外部表字段必须与RDS数据源表字段一致。
        2.  向新建的MaxCompute表中插入数据。

            命令示例如下：

            ```
            --打开外部表VPC支持，必须携带。
            set odps.sql.external.net.vpc=true;
            --插入数据。
            INSERT INTO TABLE mc_vpc_rds_external_mapping VALUES(4,"lisi");
            ```

        3.  查询数据插入结果。

            命令示例如下：

            ```
            --打开外部表VPC支持，必须携带。
            set odps.sql.external.net.vpc=true;
            --查询数据插入结果。
            SELECT * FROM mc_vpc_rds_external_mapping;
            ```


## 使用RDS数据源创建MaxCompute的外部表并加载数据（SmartNAT环境）

SmartNAT环境不需要配置开通VPC，但需要将RDS默认的VPC添加到项目空间的Flag（`odps.security.outbound.destination`）中，并在执行外部表SQL语句前增加VPC相关的Flag（`set odps.sql.external.net.vpc=true;`）。建表同样支持两种方法，使用方法与MaxCompute创建VPC环境下RDS数据源的外部表相同。

## TBLPROPERTIES属性说明

RDS外部表的TBLPROPERTIES还支持如下属性。

|属性名称|功能|
|----|--|
|`odps.federation.jdbc.condition`|从RDS获取数据时，会添加该筛选条件。设置`odps.federation.jdbc.condition`与`SELECT * FROM text_test_jdbc_write_external WHERE condition`两者的区别：

-   `odps.federation.jdbc.condition`会把过滤（Filter）操作下推到数据库，由SQL计算引擎完成。
-   `SELECT * FROM text_test_jdbc_write_external WHERE condition`会把数据完全读入MaxCompute，由MaxCompute完成过滤操作。

例如，假设RDS外部表中有100行数据，如果您通过`odps.federation.jdbc.condition`设置在MySQL上做筛选，MaxCompute通过外部表读取到的数据只有10条。如果您通过`SELECT * FROM text_test_jdbc_write_external WHERE condition`筛选，MaxCompute会从MySQL读取100条数据，然后MaxCompute在运行时筛选出其中10条数据。 |
|`odps.federation.jdbc.insert.type`|向MySQL写数据的操作类型。只支持SimpleInsert、InsertOnDuplicateKeyUpdate、ReplaceInto三种。MaxCompute的INSERT语句会分别解析为如下三类SQL语句以更新数据库：-   `INSERT INTO sqlTable xxx VALUES xxx;`
-   `INSERT INTO sqlTable xxx VALUES xxx on duplicate key update col1=values(col1), col2=values(col2);`
-   `REPLACE INTO sqlTable xxx VALUES xxx;`

如果不设置，默认为SimpleInsert。 |
|`odps.federation.jdbc.auto.commit`|设置建立连接后进行读写时，是否自动提交（auto-commit）实例。每个运行实例结束时会执行一次Commit操作。|
|`odps.federation.jdbc.insert.batch.size`|设置写入数据时，每次插入数据量的大小。默认为1000。每个运行实例结束时会执行一次插入操作。|
|`odps.federation.jdbc.input.colconvert`|在读取外部表时，可以通过该属性使用RDS内置的一些函数。例如，RDS外部表中有列col\_geometry，类型是GEOMETRY。MaxCompute外部表指明列为odps\_geo，类型是STRING。指定该属性值为odps\_geo:astext\(col\_geometry\)，即可将RDS的GEOMETRY类型数据转换为STRING读入到MaxCompute。|
|`odps.federation.jdbc.output.colconvert`|在向外部表写入数据时，您可以通过该属性使用RDS内置的一些函数。例如，RDS外部表有列col\_geometry，类型是GEOMETRY。MaxCompute外部表指明列为odps\_geo，类型是STRING。指定该属性值为odps\_geo:GEOMETRYFROMTEXT\(?\)"。即可将MaxCompute的STRING类型的数据在RDS转换为GEOMETRY并写入到RDS。|

