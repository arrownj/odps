---
keyword: [insert overwrite, insert into]
---

# 更新表或静态分区数据（INSERT INTO \| INSERT OVERWRITE）

MaxCompute支持通过`insert into`或`insert overwrite`操作向目标表或静态分区中插入、更新数据。

执行`insert into`和`insert overwrite`操作前需要具备目标表的修改权限（Alter）及源表的元信息读取权限（Describe）。授权操作请参见[授权](/cn.zh-CN/管理/安全管理详解/用户及授权管理/授权.md)。

本文中的命令您可以在如下工具平台执行：

-   [MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)
-   [MaxCompute控制台（查询编辑器）](/cn.zh-CN/工具及下载/查询编辑器.md)
-   [DataWorks控制台](https://workbench.data.aliyun.com/console)
-   [MaxCompute Studio](/cn.zh-CN/工具及下载/MaxCompute Studio/开发SQL程序/开发及提交SQL脚本.md)

## 功能介绍

在使用MaxCompute SQL处理数据时，`insert into`或`insert overwrite`操作可以将`select`查询的结果保存至目标表中。二者的区别是：

-   `insert into`：直接向表或静态分区中插入数据。您可以在`insert`语句中直接指定分区值，将数据插入指定的分区。如果您需要插入少量测试数据，可以配合[VALUES](/cn.zh-CN/开发/SQL及函数/INSERT语句/VALUES.md)使用。
-   `insert overwrite`：先清空表中的原有数据，再向表或静态分区中插入数据。

    **说明：**

    -   MaxCompute的`insert`语法与通常使用的MySQL或Oracle的`insert`语法有差别。在`insert into | insert overwrite`后需要加`table`关键字，非直接使用`table_name`。
    -   在反复对同一个分区执行`insert overwrite`操作时，您通过`desc`命令查看到的数据分区Size会不同。这是因为从同一个表的同一个分区`select`出来再`insert overwrite`回相同分区时，文件切分逻辑发生变化，从而导致数据的Size发生变化。数据的总长度在`insert overwrite`前后是不变的，您不必担心存储计费会产生问题。
    -   并发写入场景，MaxCompute会根据ACID保障并发写入操作。关于ACID的具体语义，请参见[ACID语义](/cn.zh-CN/产品简介/基本概念/ACID语义.md)。

向动态分区插入数据的操作请参见[更新动态分区数据（DYNAMIC PARTITION）](/cn.zh-CN/开发/SQL及函数/INSERT语句/输出到动态分区（DYNAMIC PARTITION）.md)。

## 使用限制

执行`insert into`和`insert overwrite`操作更新表或静态分区数据的使用限制如下：

-   `insert into`：不支持向聚簇表中追加数据。
-   `insert overwrite`：不支持指定插入列，只能使用`insert into`。

## 命令格式

```
insert {into|overwrite} table <table_name> [partition (<pt_spec>)] [(<col_name> [,<col_name> ...)]
<select_statement>
from <from_statement>
[zorder by <zcol_name> [, <zcol_name> ...]];
```

-   table\_name：必填。需要插入数据的目标表名称。
-   pt\_spec：可选。需要插入数据的分区信息，不允许使用函数等表达式，只能是常量。格式为`(partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...)`。
-   col\_name：可选。需要插入数据的目标表的列名称。`insert overwrite`不支持指定`[(<col_name> [,<col_name> ...)]`。
-   select\_statement：必填。`select`子句，从源表中查询需要插入目标表的数据。

    **说明：**

    -   源表与目标表的对应关系依赖于`select`子句中列的顺序，而不是表与表之间列名的对应关系。
    -   如果目标表是静态分区，向某个分区插入数据时，分区列不允许出现在`select`子句中。
-   from\_statement：必填。`from`子句，表示数据来源。例如，源表名称。
-   zorder by <zcol\_name\> \[, <zcol\_name\> ...\]：可选。向表或分区写入数据时，支持根据指定的一列或多列（select\_statement对应表中的列），把排序列数据相近的行排列在一起，提升查询时的过滤性能，在一定程度上降低存储成本。需要注意的是，`order by x, y`会严格地按照先x后y的顺序对数据进行排序，`zorder by x, y`会把相近的<x, y\>尽量排列在一起。当SQL查询语句的过滤条件中包含排序列时，`order by`后的数据仅对包含x的表达式有较好的过滤效果，`zorder by`后的数据对包含x或同时包含x、y的表达式均有较好的过滤效果，列压缩比例更高。

    **说明：**

    -   使用`zorder by`子句写入数据时，会占用较多资源，比不排序花费时间更多。
    -   目标表为聚簇表时，不支持`zorder by`子句。

## 使用示例

-   示例1：执行`insert into`命令向分区表`sale_detail`中追加数据。命令示例如下：

    ```
    --创建一张分区表sale_detail。
    create table if not exists sale_detail
    (
    shop_name     string,
    customer_id   string,
    total_price   double
    )
    partitioned by (sale_date string, region string);
    
    --向源表增加分区。
    alter table sale_detail add partition (sale_date='2013', region='china');
    
    --向源表追加数据。其中：`insert into table table_name`可以简写为`insert into table_name`，但`insert overwrite table table_name`不可以省略`table`关键字。
    insert into sale_detail partition (sale_date='2013', region='china') values ('s1','c1',100.1),('s2','c2',100.2),('s3','c3',100.3);
    
    --开启全表扫描，仅此Session有效。执行`select`语句查看表sale_detail中的数据。
    set odps.sql.allow.fullscan=true; 
    select * from sale_detail;
    
    --返回结果。
    +------------+-------------+-------------+------------+------------+
    | shop_name  | customer_id | total_price | sale_date  | region     |
    +------------+-------------+-------------+------------+------------+
    | s1         | c1          | 100.1       | 2013       | china      |
    | s2         | c2          | 100.2       | 2013       | china      |
    | s3         | c3          | 100.3       | 2013       | china      |
    +------------+-------------+-------------+------------+------------+
    ```

-   示例2：执行`insert overwrite`命令更新表`sale_detail_insert`中的数据。命令示例如下：

    ```
    --创建目标表sale_detail_insert，与sale_detail有相同的结构。
    create table sale_detail_insert like sale_detail;
    
    --给目标表增加分区。
    alter table sale_detail_insert add partition (sale_date='2013', region='china');
    
    --从源表sale_detail中取出数据插入目标表sale_detail_insert。注意不需要声明目标表字段，也不支持重排目标表字段顺序。
    --对于静态分区目标表，分区字段赋值已经在partition()部分声明，不需要在select_statement中包含，只要按照目标表普通列顺序查出对应字段，按顺序映射到目标表即可。动态分区表则需要在`select`中包含分区字段，详情请参见[更新动态分区数据（DYNAMIC PARTITION）](/cn.zh-CN/开发/SQL及函数/INSERT语句/输出到动态分区（DYNAMIC PARTITION）.md)。
    insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
      select 
      shop_name, 
      customer_id,
      total_price 
      from sale_detail
      zorder by customer_id, total_price;
    
    --开启全表扫描，仅此Session有效。执行`select`语句查看表sale_detail_insert中的数据。
    set odps.sql.allow.fullscan=true;
    select * from sale_detail_insert;
    
    --返回结果。
    +------------+-------------+-------------+------------+------------+
    | shop_name  | customer_id | total_price | sale_date  | region     |
    +------------+-------------+-------------+------------+------------+
    | s1         | c1          | 100.1       | 2013       | china      |
    | s2         | c2          | 100.2       | 2013       | china      |
    | s3         | c3          | 100.3       | 2013       | china      |
    +------------+-------------+-------------+------------+------------+
    ```

-   示例3：执行`insert overwrite`命令更新表`sale_detail_insert`中的数据，调整`select`子句中列的顺序。源表与目标表的对应关系依赖于`select`子句中列的顺序，而不是表与表之间列名的对应关系。命令示例如下：

    ```
    insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
        select customer_id, shop_name, total_price from sale_detail;                      
    ```

    返回结果如下：

    ```
    +------------+-------------+-------------+------------+------------+
    | shop_name  | customer_id | total_price | sale_date  | region     |
    +------------+-------------+-------------+------------+------------+
    | c1         | s1          | 100.1       | 2013       | china      |
    | c2         | s2          | 100.2       | 2013       | china      |
    | c3         | s3          | 100.3       | 2013       | china      |
    +------------+-------------+-------------+------------+------------+
    ```

    在创建`sale_detail_insert`表时，列的顺序为`shop_name string、customer_id string、total_price bigint`，而从`sale_detail`向`sale_detail_insert`插入数据的顺序为`customer_id、shop_name、total_price`。此时，会将`sale_detail.customer_id`的数据插入`sale_detail_insert.shop_name`，将`sale_detail.shop_name`的数据插入`sale_detail_insert.customer_id`。

-   示例4：向某个分区插入数据时，分区列不允许出现在`select`子句中。如下语句会返回报错，`sale_date`和`region`为分区列，不允许出现在静态分区的`select`子句中。错误命令示例如下：

    ```
    insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
       select shop_name, customer_id, total_price, sale_date, region from sale_detail;
    ```

-   示例5：`partition`的值只能是常量，不可以为表达式。错误命令示例如下：

    ```
    insert overwrite table sale_detail_insert partition (sale_date=datepart('2016-09-18 01:10:00', 'yyyy') , region='china')
       select shop_name, customer_id, total_price from sale_detail;
    ```


