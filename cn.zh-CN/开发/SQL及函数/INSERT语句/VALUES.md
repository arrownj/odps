---
keyword: [values, 插入少量数据, VALUES TABLE]
---

# VALUES

如果需要向表中插入少量数据，您可以通过`insert … values`或`values table`操作向数据量小的表中插入数据。

执行`insert into`操作前需要具备目标表的修改权限（Alter）及源表的元信息读取权限（Describe）。授权操作请参见[授权](/cn.zh-CN/管理/安全管理详解/用户及授权管理/授权.md)。

本文中的命令您可以在如下工具平台执行：

-   [MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)
-   [MaxCompute控制台（查询编辑器）](/cn.zh-CN/工具及下载/查询编辑器.md)
-   [DataWorks控制台](https://workbench.data.aliyun.com/console)
-   [MaxCompute Studio](/cn.zh-CN/工具及下载/MaxCompute Studio/开发SQL程序/开发及提交SQL脚本.md)

## 功能介绍

MaxCompute支持您通过`insert … values`或`values table`操作向表中插入少量数据。

|功能|说明|
|--|--|
|`insert … values`|在业务测试阶段，您可以通过`insert … values`操作向表中插入数据执行简单测试：-   如果插入几条或十几条数据，您可以通过`insert … values`语句快速向测试表中写入数据。
-   如果插入几十条数据，您可以通过Tunnel上传一个TXT或CSV格式的数据文件导入数据，详情请参见[导入数据](/cn.zh-CN/快速入门/导入数据.md)。您还可以通过DataWorks的导入功能快速**导入**一个数据文件，详情请参见[界面功能点介绍]()。 |
|`values table`|如果您需要对插入的数据进行简单的运算，推荐使用MaxCompute的`values table`。`values table`可以在`insert`语句和任何DML语句中使用。功能如下：-   在没有任何物理表时，您可以模拟一个有任意数据的、多行的表，并进行任意运算。
-   取代`select * from`与`union all`组合的方式，构造常量表。
-   `values table`支持特殊形式。您可以不通过`from`子句，直接执行`select`，`select`表达式列表中不可以出现其它表中的数据。其底层实现为从一个1行0列的匿名VALUES表中进行选取操作。在测试UDF或其它函数时，您可以通过该方式免去手工创建DUAL表的过程。 |

## 使用限制

通过`insert … values`或`values table`操作向表中插入数据的使用限制如下：

-   不支持通过`insert overwrite`操作指定插入列，只能通过`insert into`操作指定插入列。
-   `values`后的取值必须是常量，不支持函数。

## 命令格式

```
--`insert … values`
insert into table <table_name>
[partition (<pt_spec>)][(<col1_name> ,<col2_name>,...)] 
values (<col1_value>,<col2_value>,...),(<col1_value>,<col2_value>,...),...

--`values table`
values (<col1_value>,<col2_value>,...),(<col1_value>,<col2_value>,...),<table_name> (<col1_name> ,<col2_name>,...)...
```

-   table\_name：必填。待插入数据的表名称。该表为已经存在的表。
-   pt\_spec：可选。需要插入数据的目标分区信息，格式为`(partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...)`。如果需要更新的表为分区表，您需要指定该参数。
-   col\_name：可选。需要插入数据的目标列名称。
-   col\_value：必填。目标表中列对应的列值。多个列值之间用英文逗号（,）分隔。该列值必须为常量，未指定列值时，默认值为NULL。

    **说明：**

    -   复杂数据类型无法构造对应的常量，例如ARRAY，您可以在`values`中使用ARRAY类型，请参见[示例3](#section_zzt_4ru_ar8)。
    -   通过`values`写入DATETIME或TIMESTAMP数据类型时，需要在`values`中指定类型名称，请参见[示例4](#section_zzt_4ru_ar8)。

## 使用示例

-   示例1：通过`insert … values`操作向特定分区内插入数据。命令示例如下：

    ```
    --创建分区表srcp。
    create table if not exists srcp (key string,value bigint) partitioned by (p string);
    
    --向分区表srcp添加分区。
    alter table srcp add if not exists partition (p='abc');
    
    --向表srcp的指定分区abc中插入数据。
    insert into table srcp partition (p='abc') values ('a',1),('b',2),('c',3);
    
    --查询表srcp。
    select * from srcp where p='abc';
    
    --返回结果。
    +------------+------------+------------+
    | key        | value      | p          |
    +------------+------------+------------+
    | a          | 1          | abc        |
    | b          | 2          | abc        |
    | c          | 3          | abc        |
    +------------+------------+------------+
    ```

-   示例2：通过`insert … values`操作向非特定分区内插入数据。命令示例如下：

    ```
    --创建分区表srcp。
    create table if not exists srcp (key string,value bigint) partitioned by (p string);
    
    --向表srcp中插入数据，不指定分区。
    insert into table srcp partition (p)(key,p) values ('d','20170101'),('e','20170101'),('f','20170101');
    
    --查询表srcp。
    select * from srcp where p='20170101';
    
    --返回结果。
    +------------+------------+------------+
    | key        | value      | p          |
    +------------+------------+------------+
    | d          | NULL       | 20170101   |
    | e          | NULL       | 20170101   |
    | f          | NULL       | 20170101   |
    +------------+------------+------------+
    ```

-   示例3：使用复杂数据类型构造常量，通过`insert`操作导入数据。命令示例如下：

    ```
    --创建分区表srcp。
    create table if not exists srcp (key string,value array<int>) partitioned by (p string);
    
    --向分区表srcp添加分区。
    alter table srcp add if not exists partition (p='abc');
    
    --向表srcp的指定分区abc中插入数据。
    insert into table srcp partition (p='abc') select 'a', array(1, 2, 3);
    
    --查询表srcp。
    select * from srcp where p='abc';
    
    --返回结果。
    +------------+------------+------------+
    | key        | value      | p          |
    +------------+------------+------------+
    | a          | [1,2,3]    | abc        |
    +------------+------------+------------+
    ```

-   示例4：通过`insert … values`操作写入DATETIME或TIMESTAMP数据类型。命令示例如下：

    ```
    --创建分区表srcp。
    create table if not exists srcp (key string, value timestamp) partitioned by (p string);
    
    --向分区表srcp添加分区。
    alter table srcp add if not exists partition (p='abc');
    
    --向表srcp的指定分区abc中插入数据。
    insert into table srcp partition (p='abc') values (datetime'2017-11-11 00:00:00',timestamp'2017-11-11 00:00:00.123456789');
    
    --查询表srcp。
    select * from srcp where p='abc';
    
    --返回结果。
    +------------+------------+------------+
    | key        | value      | p          |
    +------------+------------+------------+
    | 2017-11-11 00:00:00 | 2017-11-11 00:00:00.123 | abc        |
    +------------+------------+------------+
    ```

-   示例5：通过`values table`操作插入数据。命令示例如下：

    ```
    --创建分区表srcp。
    create table if not exists srcp (key string,value bigint) partitioned by (p string);
    
    --向表srcp中插入数据。
    insert into table srcp partition (p) select concat(a,b), length(a)+length(b),'20170102' from values ('d',4),('e',5),('f',6) t(a,b);
    
    --查询表srcp。
    select * from srcp where p='20170102';
    
    --返回结果。
    +------------+------------+------------+
    | key        | value      | p          |
    +------------+------------+------------+
    | d4         | 2          | 20170102   |
    | e5         | 2          | 20170102   |
    | f6         | 2          | 20170102   |
    +------------+------------+------------+
    ```

    `values (…), (…) t(a, b)`相当于定义了一个名为`t`，列为`a`和`b`，数据类型分别为STRING和BIGINT的表。列的类型需要从`values`列表中推导。

-   示例6：取代`select * from`与`union all`组合的方式，构造常量表。命令示例如下：

    ```
    select 1 c union all select 2 c;
    --等价于如下语句。
    select * from values (1), (2) t(c);
    
    --返回结果。
    +------------+
    | c          |
    +------------+
    | 1          |
    | 2          |
    +------------+
    ```

-   示例7：通过`values table`的特殊形式插入数据，不带`from`子句。命令示例如下：

    ```
    --创建分区表srcp。
    create table if not exists srcp (key string,value bigint) partitioned by (p string);
    
    --向表srcp中插入数据。
    insert into table srcp partition (p) select abs(-1), length('abc'), getdate();
    
    --查询表srcp。
    select * from srcp;
    
    --返回结果。
    +------------+------------+------------+
    | key        | value      | p          |
    +------------+------------+------------+
    | 1          | 3          | 2020-11-25 18:39:48 |
    +------------+------------+------------+
    ```


