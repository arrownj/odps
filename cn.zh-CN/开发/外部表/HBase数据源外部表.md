---
keyword: [加载数据至HBase, HTable.put, bulkload]
---

# HBase数据源外部表

如果您需要通过MaxCompute将数据加载至HBase的表中，可以基于HBase提供的HTable.put在线接口或bulkload实现。本文为您介绍如何通过HTable.put在线接口或bulkload将数据加载至HBase的表中。

-   执行本文操作前，您需要先开通MaxCompute和HBase间的网络连接，详情请参见[t1964315.md\#]()。
-   HBase中的表需要提前通过HBase客户端创建好。详情请参见[Hive读写HBase指南]()或[使用HBase Shell](https://help.aliyun.com/document_detail/164447.html)。
-   如果您通过bulkload方式将数据加载至HBase的表中，需要打开HBase的HDFS（Hadoop Distributed File System）端口，详情请参见[访问HBase HDFS]()。

## 使用限制

-   由于只有华北2（北京）、华东2（上海）、华北3（张家口）、华东1（杭州）四个区域开通了Private Access网络连通方案（VPC专线方案），只有以上四个区域可以创建HBase数据源外部表，其他区域暂不支持。
-   HBase数据源外部表仅支持HBase标准版，暂不支持HBase增强版（Lindorm）和HBase Serverless版。

## 通过在线接口加载数据至HBase

MaxCompute的外部表功能支持您通过Hive提供的HBaseStorageHandler读写HBase中的表，其中写操作是通过HTable.put在线接口实现。

1.  登录[MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)，执行`CREATE EXTERNAL TABLE`语句创建一个映射HBase中表Schema的外部表。

    建表DDL语句如下：

    ```
    --打开外部表VPC支持。
    set odps.sql.external.net.vpc=true;     
    --打开Hive兼容模式。      
    set odps.sql.hive.compatible = true;  
    --创建外部表。        
    CREATE EXTERNAL TABLE IF NOT EXISTS <mc_hbase_external>
    (
      <col1_name> <data_type>,
      <col2_name> <data_type>
    )
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'       --处理HBase数据源的Handler。
    WITH SERDEPROPERTIES ('mcfed.hbase.table.name'='<hbase_table_name>',
                          'mcfed.hbase.columns.mapping'='<:key,col-family:col-name1,col-family:col-name2,...>',
                          'odps.external.net.vpc'='<true>',
                          'odps.vpc.id'='<VPC ID>',
                          'odps.vpc.access.ips'='<VPC ip:port,VPC ipBegin-ipEnd:port|network_segment:port>',
                          'odps.vpc.region'='<RegionID>')
    location '<hbase://VPC ip:port>'                           
    TBLPROPERTIES (
    'hbase.table.name'='<hbase_table_name>',
    'hbase.columns.mapping'='<:key,col-family:col-name1,col-family:col-name2,...>',
    'mcfed.zookeeper.session.timeout'='<value>',
    'mcfed.hbase.client.retries.number'='<value>',
    'mcfed.hbase.zookeeper.quorum'='<VPC ip1,VPC ip2...>',
    'mcfed.hbase.zookeeper.property.clientPort'='2181');
    ```

    -   mc\_hbase\_external：待创建外部表的名称。
    -   col\_name：外部表的列名称。
    -   data\_type：列的数据类型。
    -   WITH SERDEPROPERTIES：
        -   'mcfed.hbase.table.name'='<hbase\_table\_name\>'：MaxCompute所映射的HBase的表名称。
        -   'mcfed.hbase.columns.mapping'='<:key,col-family:col-name1,col-family:col-name2,...\>'：待创建外部表Schema映射的HBase的列。
            -   RowKey（`:key`）必须放在最前面。
            -   列格式为`col-family:col-name1,col-family:col-name2`。一个作业只能写一个HFile文件，即col-family只能有一个。
        -   'odps.external.net.vpc'='true'：表明外部表的数据源处于VPC网络。
        -   'odps.vpc.id'='<VPC ID\>'：VPC网络ID号。
        -   'odps.vpc.access.ips'='<VPC ip:port,VPC ipBegin-ipEnd:port\|network\_segment:port\>'：映射的HBase表所属VPC网络IP或IP网段、端口。例如`192.0.2.1-192.0.2.10:9000, 192.0.2.20:70050`或`192.0.2.20/16:2181`。
            -   对于分布式的数据源，请确认需要访问的前端IP端口以及DataNode的IP端口。HBase外部表包括Zookeeper、HMaster、Region Server的IP和端口（2181、16000、16020）。
            -   由于该字段支持IP地址段，且映射IP会耗费资源，默认一次映射的IP数不能超过2000个。
        -   'odps.vpc.region'='<RegionID\>'：VPC网络所在的区域ID。详情请参见[RegionID](/cn.zh-CN/开发/常用命令/项目空间操作.md)。
    -   hbase://VPC ip:port：HBase上配置的Zookeeper Master。
    -   TBLPROPERTIES：
        -   'hbase.table.name'='<hbase\_table\_name\>'：MaxCompute所映射的HBase的表名称。必须提前通过HBase客户端建好，例如`hbase(main):001:0> create 'test', 'cf'`。
        -   'hbase.columns.mapping'='<:key,col-family:col-name1,col-family:col-name2,...\>'：待创建外部表Schema映射的HBase的列。

            **说明：** SERDEPROPERTIES和TBLPROPERTIES里都需要指定hbase.table.name和hbase.columns.mapping，且SERDEPROPERTIES以`mcfed`开头，TBLPROPERTIES没有`mcfed`，产品后续会改进，只保留其中一个。

        -   'mcfed.zookeeper.session.timeout'='<value\>'：Zookeeper的超时时间。
        -   'mcfed.hbase.client.retries.number'='<value\>'：HBase客户端连接重试次数。
        -   'mcfed.hbase.zookeeper.quorum'='<VPC ip1,VPC ip2...\>'：HBase的Zookeeper的IP。
        -   'mcfed.hbase.zookeeper.property.clientPort'='2181'：HBase的Zookeeper的端口。默认为2181。

            **说明：** mcfed.zookeeper和mcfed.hbase开头的配置是Zookeeper的相关配置。

    示例如下：

    ```
    set odps.sql.external.net.vpc=true;
    set odps.sql.hive.compatible = true;
    CREATE EXTERNAL TABLE IF NOT EXISTS test_cluster_hbase_external
    (
      id  string,
      cfa string
    )
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES ('mcfed.hbase.table.name'='test',
                          'mcfed.hbase.columns.mapping'=':key,cf:a',
                          'odps.external.net.vpc'='true',
                          'odps.vpc.id'='vpc-uf630ldmi9l0rrr01****',
                          'odps.vpc.access.ips'='192.0.2.0-192.0.2.10:2181',
                          'odps.vpc.region'='cn-shanghai')
    location 'hbase://192.0.2.0:2181'
    TBLPROPERTIES (
    'hbase.table.name'='test', 
    'hbase.columns.mapping'=':key,cf:a',
    'mcfed.zookeeper.session.timeout'='30', 
    'mcfed.hbase.client.retries.number'='1',
    'mcfed.hbase.zookeeper.quorum'='192.0.2.0,192.0.2.1,192.0.2.2',
    'mcfed.hbase.zookeeper.property.clientPort'='2181');
    ```

2.  执行`INSERT OVERWRITE`语句向HBase中的表写入数据。

    命令示例如下：

    ```
    INSERT OVERWRITE TABLE test_cluster_hbase_external SELECT '2', 'abc';
    ```

3.  执行`SELECT`语句验证写入的数据。

    命令示例如下：

    ```
    SELECT * FROM cluster_test_hbase_external;
    ```


## 通过bulkload加载数据至HBase

当您需要向HBase导入大批量数据时，可使用HBase bulkload方式。该方式会先生成HBase的底层存储文件HFile，然后直接将这些HFile文件移动至HBase存储目录下。与调用HTable.put在线接口加载数据方式相比，因为绕过了Hbase Region Server以及WAL（Write-ahead Logging），所以处理效率更快并且对HBase运行影响更小。

1.  登录[MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)，执行`CREATE EXTERNAL TABLE`语句创建一个映射HBase中表Schema的外部表。

    建表DDL语句如下：

    ```
    set odps.sql.external.net.vpc=true;    --打开外部表VPC支持。
    set odps.sql.hive.compatible=true;     --打开Hive兼容模式。
    CREATE EXTERNAL TABLE IF NOT EXISTS <mc_hfile_hbase_external>
    (
     <col1_name> <data_type>,
     <col2_name> <data_type>
    )
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'  --处理HBase数据源的Handler。
    WITH SERDEPROPERTIES ('mcfed.hbase.table.name'='<hbase_table_name>',
                          'mcfed.hbase.columns.mapping'='<:key,col-family:col-name1,col-family:col-name2,...>', 
                          'odps.external.net.vpc'='true',
                          'odps.vpc.id'='<VPC ID>',
                          'odps.vpc.access.ips'='<VPC ip:port,VPC ipBegin-ipEnd:port|network_segment:port>',
                          'odps.vpc.region'='<RegionID>')
    location '<hbase://VPC ip:port>'   
    TBLPROPERTIES (
    'hbase.table.name'='<hbase_table_name>',
    'hbase.columns.mapping'='<:key,col-family:col-name1,col-family:col-name2,...>',
    'hfile.temp.path'='<hdfs://hbase-cluster/hfiletmp>',
    'hadoop.user.name'='<admin>',
    'mcfed.fs.defaultFS'='<hdfs://hbase-cluster>',
    'mcfed.dfs.nameservices'='<hbase-cluster>',
    'mcfed.dfs.ha.namenodes.hbase-cluster'='nn1,nn2',
    'mcfed.dfs.namenode.rpc-address.hbase-cluster.nn1'='j63g09343.sqa.eu95:8020',
    'mcfed.dfs.namenode.rpc-address.hbase-cluster.nn2'='j63g09355.sqa.eu95:8020',
    'mcfed.dfs.client.failover.proxy.provider.hbase-cluster'='org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider',
    'mcfed.zookeeper.session.timeout'='<value>',
    'mcfed.hbase.client.retries.number'='<value>',
    'mcfed.hbase.zookeeper.quorum'='<VPC ip1,VPC ip2...>',
    'mcfed.hbase.zookeeper.property.clientPort'='2181'
    ```

    -   mc\_hbase\_external：待创建外部表的名称。
    -   col\_name：外部表的列名称。
    -   data\_type：列的数据类型。
    -   WITH SERDEPROPERTIES：
        -   'mcfed.hbase.table.name'='<hbase\_table\_name\>'：MaxCompute所映射的HBase的表名称。
        -   'mcfed.hbase.columns.mapping'='<:key,col-family:col-name1,col-family:col-name2,...\>'：待创建外部表Schema映射的HBase的列。
            -   RowKey（`:key`）必须放在最前面。
            -   列格式为`col-family:col-name1,col-family:col-name2`。一个作业只能写一个HFile文件，即col-family只能有一个。
        -   'odps.external.net.vpc'='true'：表明外部表的数据源处于VPC网络。
        -   'odps.vpc.id'='<VPC ID\>'：VPC网络ID号。
        -   'odps.vpc.access.ips'='<VPC ip:port,VPC ipBegin-ipEnd:port\|network\_segment:port\>'：映射的HBase表所属VPC网络IP或IP网段、端口。例如`192.0.2.1-192.0.2.10:9000, 192.0.2.20:70050`或`192.0.2.20/16:2181`。
            -   对于分布式的数据源，请确认需要访问的前端IP端口以及DataNode的IP端口。HBase外部表包括Zookeeper、HMaster、Region Server的IP和端口（2181、16000、16020）。
            -   由于该字段支持IP地址段，且映射IP会耗费资源，默认一次映射的IP数不能超过2000个。
        -   'odps.vpc.region'='<RegionID\>'：VPC网络所在的区域ID。详情请参见[RegionID](/cn.zh-CN/开发/常用命令/项目空间操作.md)。
    -   hbase://VPC ip:port：HBase上配置的Zookeeper Master。
    -   TBLPROPERTIES：
        -   'hbase.table.name'='<hbase\_table\_name\>'：MaxCompute所映射的HBase的表名称。必须提前通过HBase客户端建好，例如`hbase(main):001:0> create 'test', 'cf'`。
        -   'hbase.columns.mapping'='<:key,col-family:col-name1,col-family:col-name2,...\>'：待创建外部表Schema映射的HBase的列。

            **说明：** SERDEPROPERTIES和TBLPROPERTIES里都需要指定hbase.table.name和hbase.columns.mapping，且SERDEPROPERTIES以`mcfed`开头，TBLPROPERTIES没有`mcfed`，产品后续会改进，只保留其中一个。

        -   hfile.temp.path：HFile临时文件的存储目录，请确保该目录只用于MaxCompute写HFile文件。当指定该配置项时，MaxCompute会将数据先写入该目录，然后运行HBase BulkLoad命令，将HFile文件导入HBase。
        -   hadoop.user.name：指定在HDFS中写HFile文件的用户。
        -   mcfed.fs.defaultFS及mcfed.dfs开头的配置项是HDFS HA的相关配置，您可参考DDL语句填写。
        -   'mcfed.zookeeper.session.timeout'='<value\>'：Zookeeper的超时时间。
        -   'mcfed.hbase.client.retries.number'='<value\>'：HBase客户端连接重试次数。
        -   'mcfed.hbase.zookeeper.quorum'='ip\_address:port\|ip\_address\_range:port\|host\_name'：HBase的Zookeeper的IP。您也可以配置为Zookeeper的Host名称。
        -   'mcfed.hbase.zookeeper.property.clientPort'='2181'：HBase的Zookeeper的端口。默认为2181。

            **说明：** mcfed.zookeeper和mcfed.hbase开头的配置是Zookeeper的相关配置。

    示例如下：

    ```
    set odps.sql.external.net.vpc=true;
    set odps.sql.hive.compatible=true;
    CREATE EXTERNAL TABLE IF NOT EXISTS test_hfile_hbase_external
    (
    id string, cfa string, cfb string
    )
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES ('mcfed.hbase.table.name'='htest',
                          'mcfed.hbase.columns.mapping'=':key,cf:a,cf:b',
                          'odps.external.net.vpc'='true',
                          'odps.vpc.id'='vpc-uf630ldmi9l0rrr01****',
                          'odps.vpc.access.ips'='192.0.2.0-192.0.2.10:8020,192.0.2.15-192.0.2.20:2181',
                          'odps.vpc.region'='cn-shanghai')
    location 'hbase://192.0.2.0:2181'
    TBLPROPERTIES (
    'hbase.table.name'='htest',
    'hbase.columns.mapping'=':key,cf:a,cf:b',
    'hfile.temp.path'='hdfs://hbase-cluster/hfiletmp',
    'hadoop.user.name'='admin',
    'mcfed.fs.defaultFS'='hdfs://hbase-cluster',
    'mcfed.dfs.nameservices'='hbase-cluster',
    'mcfed.dfs.ha.namenodes.hbase-cluster'='nn1,nn2',
    'mcfed.dfs.namenode.rpc-address.hbase-cluster.nn1'='j63g09343.sqa.eu95:8020',
    'mcfed.dfs.namenode.rpc-address.hbase-cluster.nn2'='j63g09355.sqa.eu95:8020',
    'mcfed.dfs.client.failover.proxy.provider.hbase-cluster'='org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider',
    'mcfed.zookeeper.session.timeout'='30',
    'mcfed.hbase.client.retries.number'='1',
    'mcfed.hbase.zookeeper.quorum'='j63g11301.sqa.eu95,j63g11306.sqa.eu95,j63g09367.sqa.eu95',
    'mcfed.hbase.zookeeper.property.clientPort'='2181');
    ```

2.  执行`INSERT OVERWRITE`语句向HBase中的表写入数据并做全局排序。

    -   对MaxCompute源表做全局排序，沿用MaxCompute现有的Range聚簇表机制，即Optimizer生成执行计划时，发现是写入HBase中的表，则将其视作Range聚簇表，执行计划将会对源表做全局排序。命令示例如下。

        ```
        INSERT OVERWRITE TABLE test_hfile_hbase_external SELECT uuid(), a_name,a_age FROM hy_test1;
        ```

        生成的执行计划如下，源表（hy\_test1）数据按照RowKey全局排序，即M1。

        ![执行计划](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/2222543061/p175412.png)

    -   执行`INSERT OVERWRITE`语句后，MaxCompute会自动将数据按HFile格式写入到HDFS。HFile临时路径即hfile.temp.path参数配置的内容。

        每个Fuxi Instance对应HBase外部表的一个region，即上图的R2\_1。这一步将已排好序的数据按照HFile格式写入到HDFS中。

        **说明：** 需要注意的是写HFile有两个限制：

        -   第一列必须是RowKey，即hbase.columns.mapping中的`:key`。
        -   其余列必须归属于一个col-family，这样确保每个Instance只会写入一个HFile。
    -   将HFile移动到HBase存储目录并通知HBase Region Server加载。

        当HBase外部表所有region的HFile都成功写入HDFS后，最后一步即执行bulkload，即将写好的HFile移动到HBase的存储目录中并通知HBase Region Server加载，这一步由MaxCompute自动完成。

3.  执行`SELECT`语句验证写入的数据。

    命令示例如下。

    ```
    SELECT * FROM cluster_test_hbase_external;
    ```


