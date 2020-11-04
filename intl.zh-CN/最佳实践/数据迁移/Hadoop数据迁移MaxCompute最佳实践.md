---
keyword: [Hadoop数据迁移MaxCompute, DataWorks数据同步]
---

# Hadoop数据迁移MaxCompute最佳实践

本文为您介绍如何通过DataWorks数据同步功能，迁移HDFS数据至MaxCompute，或从MaxCompute迁移数据至HDFS。无论您使用Hadoop还是Spark，均可以与MaxCompute进行双向同步。

-   开通MaxCompute并创建项目

    本文以在华东1（杭州）区域创建项目bigdata\_DOC为例。详情请参见[开通MaxCompute](/intl.zh-CN/准备工作/开通MaxCompute.md)。

-   搭建Hadoop集群

    进行数据迁移前，您需要保证Hadoop集群环境正常。本文使用阿里云EMR服务自动化搭建Hadoop集群，详情请参见[创建集群](/intl.zh-CN/快速入门/创建集群.md)。

    本文使用的EMR Hadoop版本信息如下：

    -   EMR版本：EMR-3.11.0
    -   集群类型：HADOOP
    -   软件信息：HDFS2.7.2 / YARN2.7.2 / Hive2.3.3 / Ganglia3.7.2 / Spark2.2.1 / HUE4.1.0 / Zeppelin0.7.3 / Tez0.9.1 / Sqoop1.4.6 / Pig0.14.0 / ApacheDS2.0.0 / Knox0.13.0
    Hadoop集群使用经典网络，区域为华东1（杭州），主实例组ECS计算资源配置公网及内网IP，高可用选择为否（非HA模式）。


1.  数据准备

    1.  Hadoop集群创建测试数据
        1.  进入EMR控制台界面，在项目下新建作业doc。本例中HIVE建表语句如下。 关于EMR上新建作业更多信息请参见[作业编辑](/intl.zh-CN/数据开发/作业编辑.md)。

            ```
            CREATE TABLE IF NOT
            EXISTS hive_doc_good_sale(
               create_time timestamp,
               category STRING,
               brand STRING,
               buyer_id STRING,
               trans_num BIGINT,
               trans_amount DOUBLE,
               click_cnt BIGINT
               )
               PARTITIONED BY (pt string) ROW FORMAT
            DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n';
            ```

        2.  单击**运行**，出现`Query executed successfully`提示，则说明成功在EMR Hadoop集群上创建了表hive\_doc\_good\_sale。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8593359951/p73398.png)

        3.  插入测试数据。您可以选择从OSS或其他数据源导入测试数据，也可以手动插入少量的测试数据。本文中手动插入数据如下。

            ```
            insert into
            hive_doc_good_sale PARTITION(pt =1 ) values('2018-08-21','外套','品牌A','lilei',3,500.6,7),('2018-08-22','生鲜','品牌B','lilei',1,303,8),('2018-08-22','外套','品牌C','hanmeimei',2,510,2),(2018-08-22,'卫浴','品牌A','hanmeimei',1,442.5,1),('2018-08-22','生鲜','品牌D','hanmeimei',2,234,3),('2018-08-23','外套','品牌B','jimmy',9,2000,7),('2018-08-23','生鲜','品牌A','jimmy',5,45.1,5),('2018-08-23','外套','品牌E','jimmy',5,100.2,4),('2018-08-24','生鲜','品牌G','peiqi',10,5560,7),('2018-08-24','卫浴','品牌F','peiqi',1,445.6,2),('2018-08-24','外套','品牌A','ray',3,777,3),('2018-08-24','卫浴','品牌G','ray',3,122,3),('2018-08-24','外套','品牌C','ray',1,62,7) ;
            ```

        4.  完成插入数据后，您可以执行`select * from hive_doc_good_sale where pt =1;`语句，检查Hadoop集群表中是否已存在数据可以用于迁移。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8593359951/p11596.png)

    2.  利用DataWorks新建目标表
        1.  登录[DataWorks控制台](https://workbench.data.aliyun.com/console)，单击相应工作空间后的**进入数据开发**。
        2.  进入DataStudio（数据开发）页面，右键单击工作流程，选择**新建** \> **MaxCompute** \> **表**。
        3.  在新建表对话框中，填写表名，并单击**提交**。
        4.  进入新建表页面，选择**DDL模式**。
        5.  在**DDL模式**对话框中输入建表语句，单击**生成表结构**，并确认操作。本示例的建表语句如下所示。

            ```
            CREATE TABLE IF NOT EXISTS hive_doc_good_sale(
               create_time string,
               category STRING,
               brand STRING,
               buyer_id STRING,
               trans_num BIGINT,
               trans_amount DOUBLE,
               click_cnt BIGINT
               )
               PARTITIONED BY (pt string) ;
            ```

            在建表过程中，需要考虑HIVE数据类型与MaxCompute数据类型的映射，当前数据映射关系请参见[数据类型映射表](/intl.zh-CN/开发/SQL及函数/附录/数据类型映射表.md)。

            上述步骤同样可通过odpscmd命令行工具完成，命令行工具安装和配置请参见[安装并配置客户端](/intl.zh-CN/准备工作/安装并配置客户端.md)。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8593359951/p11598.png)

            **说明：** 考虑到部分HIVE与MaxCompute数据类型的兼容问题，建议在odpscmd客户端上执行以下命令。

            ```
            set odps.sql.type.system.odps2=true;
            set odps.sql.hive.compatible=true;
            ```

        6.  单击**提交到生产环境**，完成表的创建。
        7.  完成建表后，单击左侧导航栏中的**表管理**，即可查看当前创建的MaxCompute表。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8593359951/p11599.png)

2.  数据同步

    1.  新建自定义资源组

        由于MaxCompute项目所处的网络环境与Hadoop集群中的数据节点（data node）网络通常不可达，您可以通过自定义资源组的方式，将DataWorks的同步任务运行在Hadoop集群的Master节点上（Hadoop集群内Master节点和数据节点通常可达）。

        1.  查看Hadoop集群数据节点
            1.  登录[EMR控制台](https://emr.console.aliyun.com/)，单击**集群管理**。
            2.  选择集群名称，并在左侧导航栏上单击**主机列表**。

                您也可以通过单击上图中Master节点的ECS ID，进入ECS实例详情页。然后单击远程连接进入ECS，执行`hadoop dfsadmin –report`命令查看data node。

                ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8593359951/p11601.png)

                本示例的data node只具有内网地址，很难与DataWorks默认资源组互通，所以需要设置自定义资源组，将master node设置为执行DataWorks数据同步任务的节点。

        2.  新建任务资源组
            1.  在DataWorks控制台上，进入**数据集成** \> **自定义资源组管理**页面，单击右上角的**新增自定义资源组**。

                **说明：** 目前仅专业版及以上版本方可使用此入口。

            2.  添加服务器时，需要输入ECS UUID和机器IP等信息（对于经典网络类型，需要输入服务器名称。对于专有网络类型，需要输入服务器UUID）。目前仅DataWorks V2.0华东2（上海）支持添加经典网络类型的调度资源，对于其他区域，无论您使用的是经典网络还是专有网络类型，在添加调度资源组时都请选择专有网络类型。

                机器IP需要填写master node公网IP（内网IP有可能不可达）。ECS的UUID需要进入master node管理终端，通过命令dmidecode \| grep UUID获取（如果您的hadoop集群并非搭建在EMR环境上，也可以通过该命令获取）。

                ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8593359951/p11603.png)

            3.  添加服务器后，需要保证master node与DataWorks网络可达。如果您使用的是ECS服务器，需要设置服务器安全组。
                -   如果您使用的内网IP互通，请参见[添加安全组]()。
                -   如果您使用的是公网IP，可以直接设置安全组公网出入方向规则。本文中设置公网入方向放通所有端口（实际应用场景中，为了您的数据安全，强烈建议设置详细的放通规则）。

                    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8593359951/p11604.png)

            4.  完成上述步骤后，按照提示安装自定义资源组agent。当前状态显示为**可用**时，则新增自定义资源组成功。

                ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9593359951/p11605.png)

                如果状态为**不可用**，您可以登录master node，执行`tail –f/home/admin/alisatasknode/logs/heartbeat.log`命令查看DataWorks与master node之间心跳报文是否超时。

                ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9593359951/p11606.png)

    2.  新建数据源

        DataWorks新建工作空间后，默认数据源odps\_first。因此只需要添加Hadoop集群数据源。更多详情请参见[t1695542.md\#]()。

        1.  进入数据集成页面，单击左侧导航栏中的**数据源**。
        2.  在**数据源管理**页面，单击右上角的**新增数据源**。
        3.  在新增数据源页面中，选择数据源类型为**HDFS**。
        4.  填写HDFS数据源的各配置项。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9593359951/p11608.png)

            |配置|说明|
            |:-|:-|
            |**数据源名称**|数据源名称必须以字母、数字、下划线组合，且不能以数字和下划线开头。|
            |**数据源描述**|对数据源进行简单描述，不得超过80个字符。|
            |**适用环境**|可以选择**开发**或**生产**环境。 **说明：** 仅标准模式工作空间会显示此配置。 |
            |**DefaultFS**|对于EMR Hadoop集群而言，如果Hadoop集群为HA集群，则此处地址为`hdfs://emr-header-1的IP:8020`。如果Hadoop集群为非HA集群，则此处地址为`hdfs://emr-header-1的IP:9000`。 本实验中的emr-header-1与DataWorks通过公网连接，因此此处填写公网IP并放通安全组。 |

        5.  完成配置后，单击**测试连通性**。
        6.  测试连通性通过后，单击**完成**。

            **说明：** 如果EMR Hadoop集群设置网络类型为专有网络，则不支持连通性测试。

    3.  配置数据同步任务
        1.  进入数据开发页面，在左侧菜单栏顶部选择**新建** \> **数据集成** \> **离线同步**。
        2.  在新建节点对话框中，输入**节点名称**，单击**提交**。
        3.  成功创建数据同步节点后，单击工具栏中的**转换脚本**按钮。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9593359951/p51387.png)

        4.  单击提示对话框中的**确认**，即可进入脚本模式进行开发。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9593359951/p51388.png)

        5.  单击工具栏中的**导入模板**按钮。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9593359951/p51389.png)

        6.  在导入模板对话框中，选择**来源类型**、**数据源**、**目标类型**及**数据源**，单击**确认**。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9593359951/p51390.png)

        7.  新建同步任务完成后，通过导入模板已生成了基本的读取端配置。此时您可以继续手动配置数据同步任务的读取端数据源，以及需要同步的表信息等。本示例的代码如下所示，更多详情请参见[HDFS Reader]()。

            ```
            {
              "configuration": {
                "reader": {
                  "plugin": "hdfs",
                  "parameter": {
                    "path": "/user/hive/warehouse/hive_doc_good_sale/", 
                    "datasource": "HDFS1",
                    "column": [
                      {
                        "index": 0,
                        "type": "string"
                      },
                      {
                        "index": 1,
                        "type": "string"
                      },
                      {
                        "index": 2,
                        "type": "string"
                      },
                      {
                        "index": 3,
                        "type": "string"
                      },
                      {
                        "index": 4,
                        "type": "long"
                      },
                      {
                        "index": 5,
                        "type": "double"
                      },
                      {
                        "index": 6,
                        "type": "long"
                      }
                    ],
                    "defaultFS": "hdfs://121.199.11.138:9000",
                    "fieldDelimiter": ",",
                    "encoding": "UTF-8",
                    "fileType": "text"
                  }
                },
                "writer": {
                  "plugin": "odps",
                  "parameter": {
                    "partition": "pt=1",
                    "truncate": false,
                    "datasource": "odps_first",
                    "column": [
                      "create_time",
                      "category",
                      "brand",
                      "buyer_id",
                      "trans_num",
                      "trans_amount",
                      "click_cnt"
                    ],
                    "table": "hive_doc_good_sale"
                  }
                },
                "setting": {
                  "errorLimit": {
                    "record": "1000"
                  },
                  "speed": {
                    "throttle": false,
                    "concurrent": 1,
                    "mbps": "1",
                  }
                }
              },
              "type": "job",
              "version": "1.0"
            }
            ```

            其中，path参数为数据在Hadoop集群中存放的位置。您可以在登录Master Node后，执行`hdfs dfs –ls /user/hive/warehouse/hive_doc_good_sale`命令确认。对于分区表，您可以不指定分区，DataWorks数据同步会自动递归到分区路径。

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9593359951/p11611.png)

        8.  完成配置后，单击**运行**。如果提示任务运行成功，则说明同步任务已完成。如果运行失败，可以通过日志进行排查。

1.  单击左侧导航栏中的**临时查询**。
2.  选择**新建** \> **ODPS SQL**。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9593359951/p51391.png)

3.  编写并执行SQL语句，查看导入hive\_doc\_good\_sale的数据。SQL语句如下所示：

    ```
    --查看是否成功写入MaxCompute。
    select * from hive_doc_good_sale where pt=1;
    ```

    您也可以在odpscmd命令行工具中输入`select * FROM hive_doc_good_sale where pt =1;`，查询表结果。


如果您想实现MaxCompute数据迁移至Hadoop，步骤与上述步骤类似，不同的是同步脚本内的reader和writer对象需要对调，具体实现脚本如下。

```

{
  "configuration": {
    "reader": {
      "plugin": "odps",
      "parameter": {
      "partition": "pt=1",
      "isCompress": false,
      "datasource": "odps_first",
      "column": [
        "create_time",
        "category",
        "brand",
      "buyer_id",
      "trans_num",
      "trans_amount",
      "click_cnt"
    ],
    "table": "hive_doc_good_sale"
    }
  },
  "writer": {
    "plugin": "hdfs",
    "parameter": {
    "path": "/user/hive/warehouse/hive_doc_good_sale",
    "fileName": "pt=1",
    "datasource": "HDFS_data_source",
    "column": [
      {
        "name": "create_time",
        "type": "string"
      },
      {
        "name": "category",
        "type": "string"
      },
      {
        "name": "brand",
        "type": "string"
      },
      {
        "name": "buyer_id",
        "type": "string"
      },
      {
        "name": "trans_num",
        "type": "BIGINT"
      },
      {
        "name": "trans_amount",
        "type": "DOUBLE"
      },
      {
        "name": "click_cnt",
        "type": "BIGINT"
      }
    ],
    "defaultFS": "hdfs://47.99.162.100:9000",
    "writeMode": "append",
    "fieldDelimiter": ",",
    "encoding": "UTF-8",
    "fileType": "text"
    }
  },
  "setting": {
    "errorLimit": {
      "record": "1000"
  },
  "speed": {
    "throttle": false,
    "concurrent": 1,
    "mbps": "1",
  }
  }
},
"type": "job",
"version": "1.0"
}
```

您需要参见[HDFS Writer]()，在运行上述同步任务前，对Hadoop集群进行设置。在运行同步任务后，手动复制同步过去的文件。

