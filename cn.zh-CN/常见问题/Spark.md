---
keyword: Spark常见问题
---

# Spark

本文为您介绍使用Spark过程中的常见问题。

-   [如何自检项目工程？](#section_k7t_5pu_mi9)
-   [如何将开源Spark代码迁移至MaxCompute Spark？](#section_61j_ilf_ggw)
-   [如何通过Spark访问VPC环境内的服务？](#section_wh0_38j_yk0)
-   [spark-defaults.conf提供的ID和Key错误，如何处理？](#section_n65_5pm_odh)
-   [提示权限不足，如何处理？](#section_b2c_d1f_d4m)
-   [项目不支持运行Spark作业，如何处理？](#section_xvt_wy1_hso)
-   [运行Spark作业时，报错No space left on device，如何处理？](#section_dc8_ah9_s3s)
-   [如何把JAR包当成资源来引用？](#section_hy7_xdh_prm)
-   [运行Spark作业时，报错找不到表或视图，如何处理？](#section_ezx_8yw_ipl)
-   [运行Spark作业时，打印的中文乱码，如何处理？](#section_6c7_3fy_fmg)
-   [Spark调用公网第三方任务时报错，如何处理？](#section_xc2_mf2_mxd)
-   [运行Spark作业时，报错Shutdown hook called before final status was reported，是什么原因？](#section_1wz_aeq_cwh)
-   [运行Spark作业时，发生JAR包版本冲突类错误，如何处理？](#section_3xw_sdj_aco)
-   [运行Spark作业时，发生ClassNotFound类错误，如何处理？](#section_70b_pf1_9ku)
-   [运行Spark作业时，报错The task is not in release range，如何处理？](#section_os0_blo_y3h)
-   [运行Spark作业时，报错You have NO privilege，如何处理？](#section_my7_sct_rwk)
-   [运行Spark作业时，报错User signature dose not match，如何处理？](#section_621_fa9_uzw)
-   [运行Spark作业时，报错java.io.UTFDataFormatException，如何处理？](#section_eke_m69_c39)
-   [在DataWorks上运行ODPS Spark节点的步骤是什么？](#section_6wu_0mi_1w7)
-   [如何将Spark流式读取的DataHub数据写入MaxCompute？](#section_bph_o2v_581)
-   [MaxCompute Spark如何在本地进行调试？](#section_po3_856_zbc)
-   [如何通过Spark处理MaxCompute中的表数据？](#section_usz_6fq_blx)
-   [MaxCompute Spark支持原生Spark哪个版本？](#section_id3_trs_wk7)

## 如何自检项目工程？

建议您检查如下内容：

-   检查pom.xml。

    ```
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_${scala.binary.version}</artifactId>
        <version>${spark.version}</version>
        <scope>provided</scope> // spark-xxxx_${scala.binary.version} 依赖scope必须是provided。
    </dependency>
    ```

-   检查主类spark.master。

    ```
    val spark = SparkSession
          .builder()
          .appName("SparkPi")
          .config("spark.master", "local[4]") // 如果是以yarn-cluster方式提交，代码中如果有local[N]的配置，将会报错。
          .getOrCreate()
    ```

-   检查主类Scala代码。

    ```
    object SparkPi { // 必须是object，如果在IDEA创建文件的时候写为class，main函数是无法加载的。
      def main(args: Array[String]) {
        val spark = SparkSession
          .builder()
          .appName("SparkPi")
          .getOrCreate()
    ```

-   检查主类代码配置。

    ```
    val spark = SparkSession
          .builder()
          .appName("SparkPi")
          .config("key1", "value1")
          .config("key2", "value2")
          .config("key3", "value3")
          ...  // 如果执行local测试时，将MaxCompute配置在hard-code代码里，部分配置是无法生效的。
          .getOrCreate()
    ```

    **说明：** 建议您在使用yarn-cluster方式提交任务时，将配置项都写在spark-defaults.conf中。


## 如何将开源Spark代码迁移至MaxCompute Spark？

具体场景如下：

-   作业无需访问MaxCompute表和OSS。

    您可以直接运行已有JAR包，详情请参见[搭建开发环境](/cn.zh-CN/开发/Spark/搭建开发环境.md)。对于Spark或Hadoop的依赖必须设置为provided。

-   作业需要访问MaxCompute表。

    配置相关依赖后重新打包即可，详情请参见[搭建开发环境](/cn.zh-CN/开发/Spark/搭建开发环境.md)。

-   访问OSS所需要的包，打通网络请参见[Spark访问VPC实例](/cn.zh-CN/开发/Spark/Spark访问VPC实例.md)。

    配置相关依赖后重新打包即可，详情请参见[搭建开发环境](/cn.zh-CN/开发/Spark/搭建开发环境.md)。


## 如何通过Spark访问VPC环境内的服务？

详情请参见[Spark访问VPC实例](/cn.zh-CN/开发/Spark/Spark访问VPC实例.md)。

## spark-defaults.conf提供的ID和Key错误，如何处理？

-   问题现象：报错信息如下。

    ```
    Stack:
    com.aliyun.odps.OdpsException: ODPS-0410042:
    Invalid signature value - User signature dose not match
    ```

-   解决方法：请检查spark-defaults.conf提供的ID、Key和阿里云官网控制台[用户信息管理](https://account.aliyun.com/login/login.htm?oauth_callback=https://usercenter.console.aliyun.com/)中的**AccessKey ID**、**Access Key Secret**是否一致，如果不一致，请修改一致。

## 提示权限不足，如何处理？

-   问题现象：报错信息如下。

    ```
    Stack:
    com.aliyun.odps.OdpsException: ODPS-0420095: 
    Access Denied - Authorization Failed [4019], You have NO privilege 'odps:CreateResource' on {acs:odps:*:projects/*}
    ```

-   解决方法：需要项目空间所有者授予Resource的Read和Create权限。授权详情请参见[授权](/cn.zh-CN/管理/安全管理详解/用户及授权管理/授权.md)。

## 项目不支持运行Spark作业，如何处理？

-   问题现象：报错信息如下。

    ```
    Exception in thread "main" org.apache.hadoop.yarn.exceptions.YarnException: com.aliyun.odps.OdpsException: ODPS-0420095: Access Denied - The task is not in release range: CUPID
    ```

-   解决方法：您需要确认项目所在的区域是否已经提供了MaxCompute Spark服务。如果已经提供MaxCompute Spark服务，请检查Spark-defaults.conf配置信息，详情请参见[搭建开发环境](/cn.zh-CN/开发/Spark/搭建开发环境.md)。您也可以[提工单](https://selfservice.console.aliyun.com/ticket/createIndex)或加入钉钉群：21969532（MaxCompute Spark支持群）咨询。

## 运行Spark作业时，报错No space left on device，如何处理？

Spark使用网盘进行本地存储。Shuffle数据和BlockManager溢出的数据均存储在网盘上。网盘的大小通过spark.hadoop.odps.cupid.disk.driver.device\_size参数控制，默认为20 GB，最大为100 GB。如果调整到100 GB仍然报此错误，需要分析具体原因。常见原因为数据倾斜，在Shuffle或者Cache过程中数据集中分布在某些Block。此时可以缩小单个Executor的核数（spark.executor.cores），增加Executor的数量（spark.executor.instances）。

## 如何把JAR包当成资源来引用？

您可以通过参数`spark.hadoop.odps.cupid.resources`指定需要引用的资源。资源可以多个项目共享，建议您设置相关权限确保数据安全。示例如下。

```
spark.hadoop.odps.cupid.resources = projectname.xx0.jar,projectname.xx1.jar 
```

## 运行Spark作业时，报错找不到表或视图，如何处理？

-   问题现象：报错信息如下。

    ```
    Table or view not found:xxx
    ```

-   解决办法：请检查报错的表在项目空间中是否存在。
    -   如果不存在，请创建表。
    -   如果存在，请检查是否打开了Hive的catalog配置。如果是打开的，请去掉此配置。报错示例如下，需要去掉`enableHiveSupport()`。

        ```
        spark = SparkSession.builder.appName(app_name).enableHiveSupport().getOrCreate()
        ```


## 运行Spark作业时，打印的中文乱码，如何处理？

您需要添加如下配置。

```
"--conf" "spark.executor.extraJavaOptions=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8"
"--conf" "spark.driver.extraJavaOptions=-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8"
```

## Spark调用公网第三方任务时报错，如何处理？

Spark不能调用公网第三方任务，网络不通。

您可以在VPC中搭建Nginx反向代理，通过代理访问公网。Spark支持直接访问VPC，详情请参见[Spark访问VPC实例](/cn.zh-CN/开发/Spark/Spark访问VPC实例.md)。

## 运行Spark作业时，报错Shutdown hook called before final status was reported，是什么原因？

-   问题现象：报错示例如下。

    ```
    App Status: SUCCEEDED, diagnostics: Shutdown hook called before final status was reported.
    ```

-   问题原因：提交到集群的user main并没有通过AM（ApplicationMaster）申请集群资源。例如，用户没有新建SparkContext或用户在代码中设置spark.master为local。

## 运行Spark作业时，发生JAR包版本冲突类错误，如何处理？

-   问题现象：报错示例如下。

    ```
    User class threw exception: java.lang.NoSuchMethodError
    ```

-   解决方法：
    1.  在$SPARK\_HOME/jars路径下找出异常类所在的JAR。
    2.  执行如下命令定位第三方库的坐标以及版本。

        ```
        grep <异常类类名> $SPARK_HOME/jars/*.jar
        ```

    3.  在Spark作业根目录下，执行如下命令查看整个工程的所有依赖。

        ```
        mvn dependency:tree
        ```

    4.  找到对应的依赖后，执行如下命令排除冲突包。

        ```
        maven dependency exclusions
        ```

    5.  重新编译并提交代码。

## 运行Spark作业时，发生ClassNotFound类错误，如何处理？

-   问题现象：报错示例如下。

    ```
    java.lang.ClassNotFoundException: xxxx.xxx.xxxxx
    ```

-   解决方法：
    1.  执行如下命令查看您提交的JAR包里是否存在该类定义。

        ```
        jar -tf <作业JAR包> | grep <类名称>
        ```

    2.  检查pom.xml文件中的依赖是否正确。
    3.  使用Shade方式提交JAR包。

## 运行Spark作业时，报错The task is not in release range，如何处理？

-   问题现象：报错示例如下。

    ```
    The task is not in release range: CUPID
    ```

-   解决方法：请您[提工单](https://selfservice.console.aliyun.com/ticket/createIndex)，开启该区域的MaxCompute Spark服务。

## 运行Spark作业时，报错You have NO privilege，如何处理？

-   问题现象：报错示例如下。

    ```
    Tcom.aliyun.odps.OdpsException: ODPS-0420095: 
    Access Denied - Authorization Failed [4019], You have NO privilege 'odps:CreateResource' on {acs:odps:*:projects/*}
    ```

-   解决方法：用来提交Spark作业的主账号或子账号没有对应权限，需要授权，详情请参见[授权](/cn.zh-CN/管理/安全管理详解/用户及授权管理/授权.md)。

## 运行Spark作业时，报错User signature dose not match，如何处理？

-   问题现象：报错示例如下。

    ```
    Invalid signature value - User signature dose not match
    ```

-   解决方法：通常是spark-defaults.conf中的`spark.hadoop.odps.access.id`和`spark.hadoop.odps.access.key`填写有误导致。请您确认并修改AccessKey ID和AccessKey Secret信息。

## 运行Spark作业时，报错java.io.UTFDataFormatException，如何处理？

-   问题现象：报错示例如下。

    ```
    java.io.UTFDataFormatException: encoded string too long: 2818545 bytes 
    ```

-   解决方法：调整spark-defaults.conf中spark.hadoop.odps.cupid.disk.driver.device\_size参数的值。默认为20 GB，最大支持100 GB。

## 在DataWorks上运行ODPS Spark节点的步骤是什么？

1.  在本地Python环境中编辑Spark代码并打包。本地Python环境版本要求为Python 2.7。

    **说明：** 由于DataWorks仅支持2.7环境，因此要求本地环境为Python 2.7。

2.  上传资源包至DataWorks。详情请参见[创建MaxCompute资源]()。
3.  在DataWorks上创建ODPS Spark节点。详情请参见[创建ODPS Spark节点]()。
4.  编写代码并运行节点，在DataWorks控制台上即可查看运行结果。

## 如何将Spark流式读取的DataHub数据写入MaxCompute？

示例代码请参见[DataHub](https://github.com/aliyun/MaxCompute-Spark/tree/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/streaming/datahub?spm=a2c6h.13066369.0.0.4f006a4d4HgrLv)。

## MaxCompute Spark如何在本地进行调试？

您可以通过IntelliJ IDEA在本地进行调试。详情请参见[搭建开发环境](/cn.zh-CN/开发/Spark/搭建开发环境.md)

## 如何通过Spark处理MaxCompute中的表数据？

MaxCompute Spark支持Local、Cluster和DataWorks运行模式。三种模式的配置不同，详情请参见[运行模式](/cn.zh-CN/开发/Spark/运行模式.md)。

## MaxCompute Spark支持原生Spark哪个版本？

支持Spark 1.6.3和Spark 2.3.0两个版本。

## 如何通过Spark传入参数？

传参详情请参见[Spark on Dataworks](https://github.com/aliyun/MaxCompute-Spark/wiki/17.-Spark-on-Dataworks)。

