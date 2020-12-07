---
keyword: [set, show flags]
---

# SET操作

MaxCompute支持在Session级设置MaxCompute系统变量，本文为您介绍如何设置及查看MaxCompute系统变量，影响MaxCompute的行为。

set操作相关命令如下。

|类型|功能|角色|操作入口|
|--|--|--|----|
|[set](#section_937_f6z_num)|对当前Session设置MaxCompute系统变量。|具备项目空间操作权限的用户。|本文中的命令您可以在如下工具平台执行：-   [MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)
-   [MaxCompute控制台（查询编辑器）](/cn.zh-CN/工具及下载/查询编辑器.md)
-   [DataWorks控制台](https://workbench.data.aliyun.com/console)
-   [MaxCompute Studio](/cn.zh-CN/工具及下载/MaxCompute Studio/开发SQL程序/开发及提交SQL脚本.md) |
|[show flags](#section_hwo_tsw_dgi)|显示set命令设置的属性。|

## set

对当前Session设置MaxCompute系统变量。MaxCompute也支持设置Project级的属性，详情请参见[设置项目空间属性](/cn.zh-CN/开发/常用命令/项目空间操作.md)。

-   命令格式

    ```
    set <KEY>=<VALUE>
    ```

-   参数说明
    -   KEY：属性名称。
    -   VALUE：属性值。

        Session级的常用属性如下。

        |属性名称|属性描述|取值范围|
        |:---|:---|:---|
        |console.sql.result.instancetunnel|InstanceTunnel开关。详情请参见[Tunnel命令使用说明](/cn.zh-CN/开发/数据上传下载/使用Tunnel命令上传下载数据/Tunnel命令使用说明.md)。|        -   True：打开
        -   False：关闭 |
        |odps.stage.mapper.mem|设置每个Map Worker的内存大小。|单位MB，默认值为1024 MB。|
        |odps.stage.reducer.mem|设置每个Reduce Worker的内存大小。|单位MB，默认值为1024 MB。|
        |odps.stage.joiner.mem|设置每个Join Worker的内存大小。|单位MB，默认值为1024 MB。|
        |odps.stage.mem|设置MaxCompute指定任务下所有Worker的内存大小。优先级低于odps.stage.mapper.mem、odps.stage.reducer.mem和odps.stage.joiner.mem属性。|单位MB，无默认值。|
        |odps.stage.mapper.split.size|修改每个Map Worker的输入数据量，即输入文件的分片大小，从而间接控制每个Map阶段下Worker的数量。|单位MB，默认值为256 MB。|
        |odps.stage.reducer.num|修改每个Reduce阶段的Worker数量。|无默认值。|
        |odps.stage.joiner.num|修改每个Join阶段的Worker数量。|无默认值。|
        |odps.stage.num|修改MaxCompute指定任务下所有Worker的并发数，优先级低于odps.stage.mapper.split.size、odps.stage.reducer.mem和odps.stage.joiner.num属性。|无默认值。|
        |odps.sql.reshuffle.dynamicpt|动态分区开关，以避免拆分动态分区时产生过多小文件。|        -   True：打开
        -   False：关闭
**说明：** 如果生成的动态分区个数很少，建议将值设为False，以避免数据倾斜。 |
        |odps.sql.type.system.odps2|2.0新数据类型开关。2.0数据类型详情请参见[2.0数据类型版本](/cn.zh-CN/开发/数据类型/2.0数据类型版本.md)。|        -   True：打开
        -   False：关闭 |
        |odps.sql.hive.compatible|Hive兼容模式开关。打开Hive兼容模式后，MaxCompute才支持Hive指定的各种语法，例如`inputRecordReader`、`outputRecordReader`和`Serde`。Hive兼容数据类型详情请参见[Hive兼容数据类型版本](/cn.zh-CN/开发/数据类型/Hive兼容数据类型版本.md)。|        -   True：打开
        -   False：关闭 |
        |odps.sql.executionengine.coldata.deep.buffer.size.max|设置MaxCompute在写表过程中，为复杂数据类型的列预先申请的缓存大小，以便提高写入性能。如果输出的表中的复杂数据类型过多或输出表中含有的某个单独的复杂类型变量大小过大，可以设置该参数。

        -   如果输出的表有3个列的Schema是复杂数据类型，例如列类型为（STRING、MAP、STRUCT、ARRAY或BINARY），则默认情况下MaxCompute将会为写表操作预留64 MB×3大小的内存。每一列预先申请的缓存将会用来存放这一列`batch row count`行的数据。
        -   如果预估表的复杂类型变量占用的空间比较小，建议调小此值。例如，如果每个复杂类型变量大小不会超过1024 Byte，同时`batch row count`值使用的是默认值（1024），则可以将属性值设置为1024×1024，示例如下。

            ```
set odps.sql.executionengine.coldata.deep.buffer.size.max=1048576;
            ```

        -   如果您预先知道每个复杂类型的值都在7 MB~8 MB间，同时指定了`batch row count`为32，则该值可以被调整为8 MB×32。
        -   如果输出的表有复杂类型，或`MAPJOIN`小表有复杂类型，调整该值会影响到作业执行过程中使用的内存。根据前面的计算方法，值设的过大有可能导致OOM（Out Of Memory）内存溢出。
|单位Byte，默认值为67108864 Byte。|
        |odps.task.quota.preference.tag|指定作业的Quota组（即MaxCompute管家中的配额组）。使用包年包月资源的项目空间可以通过该属性指定作业使用某个具体的二级Quota组。如果提交作业时设置的Quota Tag和某个Quota组属性中的Quota Tag相等，作业就会被优先调度到这个Quota组中。否则，会被调度到所属项目空间指定的Quota组中。执行如下语句进行设置。

        ```
set odps.task.quota.preference.tag = tag_name;
        ```

|tag\_name为MaxCompute管家中配额组的**标签**。配额组只能取作业所属项目空间Owner所在区域的配额组。tag\_name只允许使用字母、数字和下划线（\_）。|

-   使用示例

    ```
    --调整每个Mapper读取数据的大小为256 MB。
    set odps.stage.mapper.split.size=256;
    ```


## show flags

显示set命令设置的属性。命令格式如下：

```
show flags;
```

