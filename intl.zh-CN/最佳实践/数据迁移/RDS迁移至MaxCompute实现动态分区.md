---
keyword: [RDS, MaxCompute]
---

# RDS迁移至MaxCompute实现动态分区

本文为您介绍如何使用DataWorks数据集成同步功能自动创建分区，动态地将RDS中的数据迁移至MaxCompute大数据计算服务。

-   准备DataWorks环境
    1.  [开通MaxCompute](/intl.zh-CN/准备工作/开通MaxCompute.md)。
    2.  [开通DataWorks](https://common-buy.aliyun.com/)。
    3.  在DataWorks上完成创建业务流程，本例使用DataWorks简单模式。详情请参见[创建业务流程]()。
-   新增数据源
    -   新增MySQL数据源作为数据来源，详情请参见[配置MySQL数据源]()。
    -   新增MaxCompute数据源作为目标数据源接收RDS数据，详情请参见[配置MaxCompute数据源]()。

## 自动创建分区

准备工作完成后，需要将RDS中的数据定时每天同步到MaxCompute中，自动创建按天日期的分区。详细的数据同步任务的操作和配置请参见[DataWorks数据开发和运维]()。

1.  登录[DataWorks控制台](https://workbench.data.aliyun.com/console)。

2.  在MaxCompute上创建目标表。

    1.  在左侧导航栏，单击**工作空间列表**。

    2.  单击相应工作空间后的**进入数据开发**。

    3.  右键单击已创建的**业务流程**，选择**新建** \> **MaxCompute** \> **表**。

    4.  在**新建表**页面，选择引擎类型并输入**表名**。

    5.  在表的编辑页面，单击**DDL模式**。

    6.  在DDL模式对话框，输入如下建表语句，单击**生成表结构**。

        ```
        CREATE TABLE IF NOT EXISTS ods_user_info_d (
        uid STRING COMMENT '用户ID',
        gender STRING COMMENT '性别',
        age_range STRING COMMENT '年龄段',
        zodiac STRING COMMENT '星座'
        )
        PARTITIONED BY (
        dt STRING
        );                           
        ```

    7.  单击**提交到生产环境**。

3.  新建离线同步节点。

    1.  进入数据开发页面，右键单击指定业务流程，选择**新建** \> **数据集成** \> **离线同步**。

    2.  在**新建节点**对话框中，输入**节点名称**，并单击**提交**。

    3.  选择数据来源和数据去向。

        ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9693359951/p102236.png)

4.  配置分区参数。

    1.  在右侧导航栏上，单击**调度配置**。

    2.  在**基础属性**区域，设置**参数**。参数值默认为系统自带的时间参数`${bizdate}`，格式为yyyymmdd。

        **说明：** 默认**参数**值与**数据去向**中的**分区信息**值对应。调度执行迁移任务时，目标表的分区值会被自动替换为任务执行日期的前一天，默认情况下，您会在当前执行前一天的业务数据，这个日期也叫做业务日期。如果您需要使用当天任务运行的日期作为分区值，则需自定义参数值。

        自定义参数设置：用户可以自主选择某一天和格式配置，如下所示。

        -   后N年：`$[add_months(yyyymmdd,12*N)]`
        -   前N年：`$[add_months(yyyymmdd,-12*N)]`
        -   前N月：`$[add_months(yyyymmdd,-N)]`
        -   后N周：`$[yyyymmdd+7*N]`
        -   后N月：`$[add_months(yyyymmdd,N)]`
        -   前N周：`$[yyyymmdd-7*N]`
        -   后N天：`$[yyyymmdd+N]`
        -   前N天：`$[yyyymmdd-N]`
        -   后N小时：`$[hh24miss+N/24]`
        -   前N小时：`$[hh24miss-N/24]`
        -   后N分钟：`$[hh24miss+N/24/60]`
        -   前N分钟：`$[hh24miss-N/24/60]`
        **说明：**

        -   使用中括号（\[\]）编辑自定义变量参数的取值计算公式，例如 `key1=$[yyyy-mm-dd]`。
        -   默认情况下，自定义变量参数的计算单位为天。例如 `$[hh24miss-N/24/60]`表示`(yyyymmddhh24miss-(N/24/60 * 1天))`的计算结果，然后按 hh24miss 的格式取时分秒。
        -   使用add\_months的计算单位为月。例如 `$[add_months(yyyymmdd,12 N)-M/24/60]` 表示`(yyyymmddhh24miss-(12 * N * 1月))-(M/24/60 * 1天)`的结果，然后按 `yyyymmdd` 的格式取年月日。
5.  单击![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9993359951/p100471.png)图标运行代码。

6.  您可以在**运行日志**查看运行结果。


## 补数据实验

如果您的数据中存在大量运行日期之前的历史数据，需要实现自动同步和自动分区。您可以通过DataWorks的**运维中心**，选择当前的同步数据节点，使用**补数据**功能实现。

1.  在RDS端按照日期筛选出历史数据。

    您可以在同步节点**数据来源**区域设置**数据过滤**条件。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9693359951/p14401.png)

2.  执行补数据操作。详情请参见[补数据实例]()

3.  在运行的日志中查看对RDS数据的抽取结果。

    从运行结果中可以看到MaxCompute已自动创建分区。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9693359951/p21001.png)

4.  运行结果验证。在MaxCompute客户端执行如下命令，查看数据写入情况。

    ```
    SELECT count(*) from ods_user_info_d where dt = 20180913;
    ```


## Hash实现非日期字段分区

如果您的数据量较大，或没有按照日期字段对第一次全量的数据进行分区，而是按照省份等非日期字段分区，则此时数据集成操作将不能实现自动分区。这种情况下，您可以按照RDS中某个字段进行Hash，将相同的字段值自动存放到这个字段对应值的MaxCompute分区中。

1.  将数据全量同步到MaxCompute的一个临时表，创建一个SQL脚本节点。执行如下命令。

    ```
    drop table if exists ods_user_t;
    CREATE TABLE ods_user_t ( 
            dt STRING,
            uid STRING,
            gender STRING,
            age_range STRING,
            zodiac STRING);
    --将MaxCompute表中的数据存入临时表。
    insert overwrite table ods_user_t select dt,uid,gender,age_range,zodiac from ods_user_info_d;         
    ```

2.  创建同步任务的节点mysql\_to\_odps，即简单的同步任务。将RDS数据全量同步到MaxCompute，无需设置分区。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/0793359951/p33591.png)

3.  使用SQL语句进行动态分区到目标表，命令如下。

    ```
    drop table if exists ods_user_d;
    //创建一个ODPS分区表（最终目的表）。
        CREATE TABLE ods_user_d (
        uid STRING,
            gender STRING,
            age_range STRING,
            zodiac STRING
    )
    PARTITIONED BY (
        dt STRING
    );
    //执行动态分区SQL，按照临时表的字段dt自动分区，dt字段中相同的数据值，会按照这个数据值自动创建一个分区值。
    //例如dt中有些数据是20181025，会自动在ODPS分区表中创建一个分区，dt=20181025。
    //动态分区SQL如下。
    //可以注意到SQL中select的字段多写了一个dt，就是指定按照这个字段自动创建分区。
    insert overwrite table ods_user_d partition(dt)select dt,uid,gender,age_range,zodiac from ods_user_t;
    //导入完成后，可以把临时表删除，节约存储成本。
    drop table if exists ods_user_t;
    ```

    在MaxCompute中您可以通过SQL语句完成数据同步。详细的SQL语句介绍请参见[阿里云大数据利器MaxCompute学习之--分区表的使用](https://yq.aliyun.com/articles/81775?spm=a2c4e.11153940.blogcont182972.20.72856a07pHyeLb)。

4.  将三个节点配置成一个工作流，按顺序执行。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/0793359951/p21007.png)

5.  查看执行过程。您可以重点观察最后一个节点的动态分区过程。

    ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/0793359951/p103159.png)

6.  运行结果验证。在MaxCompute客户端执行如下命令，查看数据写入情况。

    ```
    SELECT count(*) from ods_user_d where dt = 20180913;
    ```


