---
keyword: [Tunnel, Tunnel Upload, Tunnel Download]
---

# Tunnel操作

MaxCompute通过Tunnel实现上传下载数据功能。本文为您介绍如何通过Tunnel上传、下载数据。

Tunnel操作详情请参见[Tunnel命令参考](/cn.zh-CN/开发/数据上传下载/使用Tunnel命令上传下载数据/Tunnel命令参考.md)。Tunnel操作常用命令如下。

|类型|功能|角色|操作入口|
|--|--|--|----|
|[上传数据](#section_51k_3og_7yg)|将本地文件的数据上传至MaxCompute的表中，以追加模式导入。|具备修改表权限（Alter）的用户。|本文中的命令您需要在[MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)执行。|
|[下载数据](#section_ypo_l7f_h7k)|将MaxCompute表数据或指定Instance的执行结果下载至本地。|具备读取表数据权限（Select）的用户。|

## 上传数据

将本地文件的数据上传至MaxCompute的表中，以追加模式导入。上传数据至MaxCompute不收取费用。

-   使用限制
    -   支持文件或目录（指一级目录）的上传，每一次上传只支持数据上传到一张表或表的一个[分区](/cn.zh-CN/开发/SQL及函数/DDL语句/分区和列操作.md)。
    -   分区表一定要指定上传的分区，多级分区一定要指定到末级分区。
-   命令格式

    ```
    Tunnel upload <path> [<project_name>.]<table_name>[/<pt_spc>];
    ```

-   参数说明
    -   path：必填。上传数据文件的存放路径以及文件名称。默认上传的数据文件为TXT格式。
    -   project\_name：可选。目标表所属项目空间名称。跨项目空间访问表时需要指定该参数。
    -   table\_name：必填。目标表名。
    -   pt\_spc：可选。需要指定至最末级分区。格式为`partition_col1=col1_value1, partition_col2=col2_value1...`。
-   使用示例
    -   示例1：将log.txt中的数据上传至当前项目空间的表test\_table中。

        ```
        Tunnel upload log.txt test_table;
        ```

    -   示例2：将log.txt中的数据上传至项目空间test\_project的表test\_table（二级分区表）中的`p1="b1",p2="b2"`分区。

        ```
        Tunnel upload log.txt test_project.test_table/p1="b1",p2="b2";
        ```


## 下载数据

将MaxCompute表数据或指定Instance的执行结果下载至本地。MaxCompute仅对公网的数据下载，按照下载的数据大小进行计费，一次下载费用=下载数据量（GB）×下载价格（0.8元/GB）。

-   使用限制
    -   只支持下载到单个文件，每一次下载只支持下载一张表或一个分区到一个文件。
    -   分区表一定要指定下载的分区，多级分区一定要指定到末级分区。
-   命令格式

    ```
    Tunnel download [<project_name>.]<table_name>[/<pt_spc>] <path>;
    ```

-   参数说明
    -   project\_name：可选。目标表所属项目空间名称。跨项目空间访问表时需要指定该参数。
    -   table\_name：必填。目标表表名。
    -   pt\_spc：可选。需要指定至最末级分区。格式为`partition_col1=col1_value1, partition_col2=col2_value1...`。
    -   path：必填。下载数据文件的存放路径以及文件名称。默认下载的数据文件为TXT格式。
-   使用示例

    ```
    --将test_project.test_table表（二级分区表）中的数据下载到test_table.txt文件中。
    Tunnel download test_project.test_table/p1="b1",p2="b2" test_table.txt;
    ```


