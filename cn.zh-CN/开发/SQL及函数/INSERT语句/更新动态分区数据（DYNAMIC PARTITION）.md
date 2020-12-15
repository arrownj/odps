---
keyword: [insert into, insert overwrite, 动态分区, 输出到动态分区]
---

# 更新动态分区数据（DYNAMIC PARTITION）

MaxCompute支持通过`insert into`或`insert overwrite`操作向动态分区中插入数据。

执行`insert into`和`insert overwrite`操作前需要具备目标表的修改权限（Alter）及源表的元信息读取权限（Describe）。授权操作请参见[授权](/cn.zh-CN/管理/安全管理详解/用户及授权管理/授权.md)。

本文中的命令您可以在如下工具平台执行：

-   [MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)
-   [MaxCompute控制台（查询编辑器）](/cn.zh-CN/工具及下载/查询编辑器.md)
-   [DataWorks控制台](https://workbench.data.aliyun.com/console)
-   [MaxCompute Studio](/cn.zh-CN/工具及下载/MaxCompute Studio/开发SQL程序/开发及提交SQL脚本.md)

## 功能介绍

在使用MaxCompute SQL处理数据时，`insert into`或`insert overwrite`语句中不直接指定分区值，只指定分区列名（分区字段）。分区列的值在`select`子句中提供，系统自动根据分区列的值将数据插入到相应分区。

向静态分区插入数据的操作请参见[更新表或静态分区数据（INSERT INTO \| INSERT OVERWRITE）](/cn.zh-CN/开发/SQL及函数/INSERT语句/更新表或静态分区数据（INSERT INTO | INSERT OVERWRITE）.md)。

## 使用限制

通过`insert into`和`insert overwrite`操作向动态分区中插入数据的使用限制如下：

-   `insert into`最多可以生成10000个动态分区，`insert overwrite`最多可以生成60000个动态分区。
-   分布式环境下，在使用动态分区功能的SQL语句中，单个进程最多只能输出512个动态分区，否则会运行异常。
-   动态生成的分区值不允许为NULL，也不支持特殊字符和中文，否则会报错`FAILED: ODPS-0123031:Partition exception - invalid dynamic partition value: province=xxx`。
-   聚簇表不支持动态分区。

## 注意事项

如果您需要更新表数据到动态分区，需要注意：

-   `insert into partition`时，如果分区不存在，会自动创建分区。
-   多个`insert into partition`作业并发时，如果分区不存在，优先执行成功的作业会自动创建分区，但只会成功创建一个分区。
-   如果不能控制`insert into partition`作业并发，建议您通过`alter table`命令提前创建分区，详情请参见[分区和列操作](/cn.zh-CN/开发/SQL及函数/DDL语句/分区和列操作.md)。
-   如果目标表有多级分区，在执行`insert`操作时，允许指定部分分区为静态分区，但是静态分区必须是高级分区。
-   向动态分区插入数据时，动态分区列必须在`select`列表中，否则会执行失败。

## 命令格式

```
insert {into|overwrite} table <table_name> partition (<ptcol_name>[, <ptcol_name> ...]) 
<select_statement> from <from_statement>;
```

-   table\_name：必填。需要插入数据的目标表名。
-   ptcol\_name：必填。目标表分区列的名称。
-   select\_statement：必填。`select`子句，从源表中查询需要插入目标表的数据。

    如果目标表只有一级动态分区，则`select`子句的最后一个字段值即为目标表的动态分区值。源表`select`的值和输出分区的值的关系是由字段顺序决定，并不是由列名称决定的。当源表的字段与目标表字段顺序不一致时，建议您按照目标表顺序在select\_statement语句中指定字段。

-   from\_statement：必填。`from`子句，表示数据来源。例如，源表名称。

## 使用示例

-   示例1：将源表中的数据插入到目标表中。在运行SQL语句之前，您无法得知会产生哪些分区。只有在语句运行结束后，才能通过region字段产生的值确定产生的分区。命令示例如下：

    ```
    --创建目标表total_revenues。
    create table total_revenues (revenue double) partitioned by (region string);
    
    --将源表sale_detail中的数据插入到目标表total_revenues。源表信息请参见[更新表或静态分区数据（INSERT INTO \| INSERT OVERWRITE）](/cn.zh-CN/开发/SQL及函数/INSERT语句/更新表或静态分区数据（INSERT INTO | INSERT OVERWRITE）.md)。
    insert overwrite table total_revenues partition(region)
    select total_price as revenue, region from sale_detail;
    
    --执行`show partitions`语句查看表total_revenues的分区。
    show partitions total_revenues;
       
    --返回结果。
    region=china  
    
    --开启全表扫描，仅此Session有效。执行`select`语句查看表sale_detail_dypart中的数据。  
    set odps.sql.allow.fullscan=true; 
    select * from sale_detail_dypart;    
    
    --返回结果。
    +------------+------------+
    | revenue    | region     |
    +------------+------------+
    | 100.1      | china      |
    | 100.2      | china      |
    | 100.3      | china      |
    +------------+------------+        
    ```

-   示例2：将源表中的数据插入到目标表中。多级分区，指定一级分区sale\_date。命令示例如下：

    ```
    --创建目标表sale_detail_dypart。 
    create table sale_detail_dypart like sale_detail; 
    
    --指定一级分区，将数据插入目标表。
    insert overwrite table sale_detail_dypart partition (sale_date='2013', region)
    select shop_name,customer_id,total_price,region from sale_detail;
    
    --开启全表扫描，仅此Session有效。执行`select`语句查看表sale_detail_dypart中的数据。
    set odps.sql.allow.fullscan=true; 
    select * from sale_detail_dypart;
    
    --返回结果。
    +------------+-------------+-------------+------------+------------+
    | shop_name  | customer_id | total_price | sale_date  | region     |
    +------------+-------------+-------------+------------+------------+
    | s1         | c1          | 100.1       | 2013       | china      |
    | s2         | c2          | 100.2       | 2013       | china      |
    | s3         | c3          | 100.3       | 2013       | china      |
    +------------+-------------+-------------+------------+------------+
    ```

-   示例3：动态分区中，select\_statement字段和目标表动态分区的对应关系是由字段顺序决定，并不是由列名称决定的。命令示例如下：

    ```
    --将源表sale_detail中的数据插入到目标表sale_detail_dypart。
    insert overwrite table sale_detail_dypart partition (sale_date, region)
    select shop_name,customer_id,total_price,sale_date,region from sale_detail;
    
    --开启全表扫描，仅此Session有效。执行`select`语句查看表sale_detail_dypart中的数据。
    set odps.sql.allow.fullscan=true; 
    select * from sale_detail_dypart;
    
    --返回结果。决定目标表sale\_detail\_dypart动态分区的字段sale\_date为源表sale\_detail的字段sale\_date；决定目标表sale\_detail\_dypart动态分区的字段region为源表sale\_detail的字段region。
    +------------+-------------+-------------+------------+------------+
    | shop_name  | customer_id | total_price | sale_date  | region     |
    +------------+-------------+-------------+------------+------------+
    | s1         | c1          | 100.1       | 2013       | china      |
    | s2         | c2          | 100.2       | 2013       | china      |
    | s3         | c3          | 100.3       | 2013       | china      |
    +------------+-------------+-------------+------------+------------+
    
    --将源表sale_detail中的数据插入到目标表sale_detail_dypart，调整`select`字段顺序。
    insert overwrite table sale_detail_dypart partition (sale_date, region)
    select shop_name,customer_id,total_price,region,sale_date from sale_detail;
    
    --开启全表扫描，仅此Session有效。执行`select`语句查看表sale_detail_dypart中的数据。
    set odps.sql.allow.fullscan=true; 
    select * from sale_detail_dypart;
    
    --返回结果。决定目标表sale\_detail\_dypart动态分区的字段sale\_date为源表sale\_detail的字段region；决定目标表sale\_detail\_dypart动态分区的字段region为源表sale\_detail的字段sale\_date。
    +------------+-------------+-------------+------------+------------+
    | shop_name  | customer_id | total_price | sale_date  | region     |
    +------------+-------------+-------------+------------+------------+
    | s1         | c1          | 100.1       | china      | 2013       |
    | s2         | c2          | 100.2       | china      | 2013       |
    | s3         | c3          | 100.3       | china      | 2013       |
    +------------+-------------+-------------+------------+------------+
    ```

-   示例4：向动态分区插入数据时，动态分区列必须在`select`列表中，否则会执行失败。错误命令示例如下：

    ```
    insert overwrite table sale_detail_dypart partition (sale_date='2013', region)
    select shop_name,customer_id,total_price from sale_detail;
    ```

-   示例5：向动态分区插入数据时，不能仅指定低级子分区，而动态插入高级分区，否则会执行失败。错误命令示例如下：

    ```
    insert overwrite table sales partition (region='china', sale_date)
    select shop_name,customer_id,total_price,sale_date from sale_detail;
    ```

-   示例6：MaxCompute在向动态分区插入数据时，如果分区列的类型与对应`select`中列的类型不严格一致，会[隐式转换](/cn.zh-CN/开发/数据类型/数据类型版本说明.md)，命令示例如下：

    ```
    --创建源表src。
    create table src (c int, d string) partitioned by (e int);
    
    --向源表src增加分区。
    alter table src add if not exists partition (e=201312);
    
    --向源表src追加数据。
    insert into src partition (e=201312) values (1,100.1),(2,100.2),(3,100.3);
    
    --创建目标表parttable。
    create table parttable(a int, b double) partitioned by (p string);
    
    --将源表src数据插入目标表parttable。
    insert into parttable partition (p) select c, d, current_timestamp() from src;
    
    --查询目标表parttable。
    select * from parttable;
    
    --返回结果如下。
    +------------+------------+------------+
    | a          | b          | p          |
    +------------+------------+------------+
    | 1          | 100.1      | 2020-11-25 15:13:28.686 |
    | 2          | 100.2      | 2020-11-25 15:13:28.686 |
    | 3          | 100.3      | 2020-11-25 15:13:28.686 |
    +------------+------------+------------+
    ```

    **说明：** 如果您的数据是有序的，动态分区插入会把数据随机打散，导致压缩率较低。推荐您使用[Tunnel命令](/cn.zh-CN/开发/数据上传下载/使用Tunnel命令上传下载数据/Tunnel命令参考.md)上传数据到动态分区，以获取较好的压缩率。使用该命令的详细示例请参见[RDS迁移至MaxCompute实现动态分区](/cn.zh-CN/最佳实践/数据迁移/RDS迁移至MaxCompute实现动态分区.md)。


