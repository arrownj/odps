---
keyword: SET操作
---

# SET操作

本文向您介绍如何使用set命令设置MaxCompute或用户自定义的系统变量。

## SET

-   命令格式

    ```
    SET <KEY>=<VALUE>
    ```

-   功能说明

    对当前Session设置MaxCompute系统变量或者影响MaxCompute的行为。

    **说明：** Project级别设置请参见[Setproject](/cn.zh-CN/开发/常用命令/项目空间操作.md)。

-   参数说明
    -   KEY：需要修改的属性名称。
    -   VALUE：属性值，详情请参见下表。

        |属性名称|属性描述|取值范围|
        |:---|:---|:---|
        |odps.sql.allow.fullscan|设置是否允许对分区表进行全表扫描。|True（允许）、False（禁止）。|
        |odps.stage.mapper.mem|设置每个Map Worker的内存大小。|单位MB，默认值1024 MB。|
        |odps.stage.reducer.mem|设置每个Reduce Worker的内存大小。|单位MB，默认值1024 MB。|
        |odps.stage.joiner.mem|设置每个Join Worker的内存大小。|单位MB，默认值1024 MB。|
        |odps.stage.mem|设置MaxCompute 指定任务下所有Worker的内存大小。优先级低于以上三个属性。|单位MB，无默认值。|
        |odps.stage.mapper.split.size|修改每个Map Worker的输入数据量，即输入文件的分片大小，从而间接控制每个Map阶段下Worker的数量。|单位MB，默认值256 MB。|
        |odps.stage.reducer.num|修改每个Reduce阶段的Worker数量。|无默认值。|
        |odps.stage.joiner.num|修改每个Join阶段的Worker数量。|无默认值。|
        |odps.stage.num|修改MaxCompute指定任务的所有阶段的Worker的并发度，优先级低于以上三者。|无默认值。|
        |odps.sql.reshuffle.dynamicpt|设置动态分区，以避免拆分动态分区时产生过多小文件。|True（允许）、False（禁止），默认值为True。**说明：** 如果生成的动态分区个数很少，建议将值设为False，以避免数据倾斜。 |
        |odps.sql.type.system.odps2|是否打开新数据类型开关。SQL操作中涉及到[新数据类型](/cn.zh-CN/开发/数据类型/数据类型版本说明.md)（TINYINT、SMALLINT、INT、FLOAT、VARCHAR、TIMESTAMP和BINARY等）时需要设置为True。此设置为Session级别的设置。|True（允许）、False（禁止），默认为False。|
        |odps.sql.hive.compatible|是否打开Hive兼容模式。打开后，MaxCompute才可以支持Hive指定的各种语法，例如inputRecordReader、outputRecordReader和Serde等。此设置为Session级别的设置。|True（允许）、False（禁止），默认为False。|
        |odps.sql.executionengine.coldata.deep.buffer.size.max|修改MaxCompute写表过程中为单个复杂类型的列预先申请缓存大小，以便提高写入性能。|        -   使用场景
            -   输出的表中的复杂数据类型过多。
            -   输出的表中含有的某个单独的复杂类型变量Size过大。
        -   使用说明
            -   默认值为67108864（64 MB），默认单位为Byte。
            -   如果输出的表有3个列的Schema是复杂类型，例如列类型为（STRING、MAP、STRUCT、ARRAY或BINARY），则默认情况下MaxCompute将会为写表操作预留64 MB×3大小的内存。每一列预先申请的缓存将会用来存放这一列`batch row count`行的数据。
            -   如果预先可知道表的复杂类型变量占用的空间比较小，则建议调小此值。例如，如果每个复杂类型变量大小不会超过1024 Byte，同时`batch row count`值使用的是默认值（1024），则可以将Flag设置为1024×1024，示例如下。

                ```
SET odps.sql.executionengine.coldata.deep.buffer.size.max=1048576;
                ```

            -   如果您预先知道每个复杂类型的值都在7 MB到8 MB之间，同时指定了`batch row count`为32，则这个值可以被调整为8 MB×32。
            -   如果任务的输出带有复杂类型，或者任务的`MAPJOIN`小表带有复杂类型，这个值的调整会影响到任务执行过程中使用的内存。根据前面的计算方法，值设的过大有可能导致任务OOM内存溢出。 |
        |odps.task.quota.preference.tag|指定作业的Quota组（即MaxCompute管家中的配额组）。使用包年包月资源的Project可以通过该属性指定作业使用某个具体的二级Quota组。如果提交作业时设置的Quota Tag和某个Quota组属性中的Quota Tag相等。此作业就会被优先调度到这个Quota组中。否则，会被调度到所属Project指定的Quota组中。执行如下语句进行设置。

        ```
SET odps.task.quota.preference.tag = tag_name;
        ```

|tag\_name为MaxCompute管家中配额组的**标签**。配额组只能取作业所属Project Owner所在区域的配额组。tag\_name只允许使用字母、数字和下划线（\_）。|

-   示例
    -   调整MaxCompute写表过程中为单个复杂类型的列预先申请缓存大小，以便提高写入性能。

        ```
        SET odps.sql.executionengine.coldata.deep.buffer.size.max=1048576;
        ```

    -   调整每个Mapper读取数据的大小为256 MB。

        ```
        SET odps.stage.mapper.split.size=256
        ```


## SHOW FLAGS

-   命令格式

    ```
    SHOW FLAGS;
    ```

-   功能说明

    显示SET设置的参数。

    **说明：** 该命令仅对Session级别生效，Project级别Flag设置请参见[项目空间操作](/cn.zh-CN/开发/常用命令/项目空间操作.md)。


