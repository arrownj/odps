---
keyword: [MULTI INSERT, 多路输出]
---

# 多路输出（MULTI INSERT）

MaxCompute SQL支持您在一条SQL语句中通过`insert into`或`insert overwrite`操作将数据插入不同的目标表或者分区中，实现多路输出。

执行`insert into`和`insert overwrite`操作前需要具备目标表的修改权限（Alter）及源表的元信息读取权限（Describe）。授权操作请参见[授权](/cn.zh-CN/管理/安全管理详解/用户及授权管理/授权.md)。

本文中的命令您可以在如下工具平台执行：

-   [MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)
-   [MaxCompute控制台（查询编辑器）](/cn.zh-CN/工具及下载/查询编辑器.md)
-   [DataWorks控制台](https://workbench.data.aliyun.com/console)
-   [MaxCompute Studio](/cn.zh-CN/工具及下载/MaxCompute Studio/开发SQL程序/开发及提交SQL脚本.md)

## 功能介绍

在使用MaxCompute SQL处理数据时，`multi insert`操作可以将数据插入不同的目标表或分区中，实现多路输出。

## 使用限制

`multi insert`操作的使用限制如下：

-   单条`multi insert`语句中最多可以写255路输出。超过255路，会上报语法错误。
-   对于同一张分区表的不同分区，不能同时有`insert overwrite`和`insert into`操作，否则返回报错。
-   单条`multi insert`语句中，对于分区表，同一个目标分区不允许出现多次。
-   单条`multi insert`语句中，对于非分区表，该表不能出现多次。

## 命令格式

```
from <from_statement>
insert overwrite | into table <table_name1> [partition (<pt_spec1>)]
<select_statement1>
insert overwrite | into table <table_name2> [partition (<pt_spec2>)]
<select_statement2>
...;
```

-   from\_statement：必填。`from`子句，代表数据来源。例如，源表名称。
-   table\_name：必填。需要插入数据的目标表名称。
-   pt\_spec：可选。需要插入数据的目标分区信息，此参数不允许使用函数等表达式，只能是常量。格式为`(partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...)`。插入多个分区时，例如pt\_spec1和pt\_spec2，目标分区不允许出现多次，即pt\_spec1和pt\_spec2的分区信息不相同。
-   select\_statement：必填。`select`子句，从源表中查询需要插入的数据。

## 使用示例

-   示例1：将表sale\_detail的数据插入到表sale\_detail\_multi的2010年及2011年中国的销售记录中。命令示例如下：

    ```
    --创建表sale_detail_multi。
    create table sale_detail_multi like sale_detail;
    
    --开启全表扫描，仅此Session有效。将表sale_detail中的数据插入到表sale_detail_multi。
    set odps.sql.allow.fullscan=true; 
    from sale_detail
    insert overwrite table sale_detail_multi partition (sale_date='2010', region='china' )
    select shop_name, customer_id, total_price
    insert overwrite table sale_detail_multi partition (sale_date='2011', region='china' )
    select shop_name, customer_id, total_price;
    
    --开启全表扫描，仅此Session有效。执行`select`语句查看表sale_detail_multi中的数据。
    set odps.sql.allow.fullscan=true;
    select * from sale_detail_multi;
    
    --返回结果。
    +------------+-------------+-------------+------------+------------+
    | shop_name  | customer_id | total_price | sale_date  | region     |
    +------------+-------------+-------------+------------+------------+
    | s1         | c1          | 100.1       | 2010       | china      |
    | s2         | c2          | 100.2       | 2010       | china      |
    | s3         | c3          | 100.3       | 2010       | china      |
    | s1         | c1          | 100.1       | 2011       | china      |
    | s2         | c2          | 100.2       | 2011       | china      |
    | s3         | c3          | 100.3       | 2011       | china      |
    +------------+-------------+-------------+------------+------------+
    ```

-   示例2：同一目标分区不允许出现多次，否则会返回报错。**错误命令示例**如下：

    ```
    from sale_detail
    insert overwrite table sale_detail_multi partition (sale_date='2010', region='china' )
    select shop_name, customer_id, total_price
    insert overwrite table sale_detail_multi partition (sale_date='2010', region='china' )
    select shop_name, customer_id, total_price;
    ```

-   示例3：不允许同时向同一张表的不同分区执行`insert overwrite`和`insert into`操作，否则会返回报错。**错误命令示例**如下：

    ```
    from sale_detail
    insert overwrite table sale_detail_multi partition (sale_date='2010', region='china' )
    select shop_name, customer_id, total_price
    insert into table sale_detail_multi partition (sale_date='2011', region='china' )
    select shop_name, customer_id, total_price;
    ```


