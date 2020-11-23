---
keyword: [INSERT OVERWRITE, INSERT INTO]
---

# 更新表数据（INSERT OVERWRITE and INSERT INTO）

本文向您介绍如何使用INSERT OVERWRITE和INSERT INTO两种命令更新表数据。

## INSERT命令

-   **命令格式**

    ```
    INSERT OVERWRITE|INTO TABLE table_name [PARTITION (partcol1=val1, partcol2=val2 ...)] [(col1,col2 ...)]
    select_statement
    FROM from_statement
    [ZORDER BY zcol1 [, zcol2 ...]];
    ```

-   **功能说明**

    在使用MaxCompute SQL处理数据时，`INSERT OVERWRITE|INTO`用于将计算的结果保存至目标表中。

    -   INSERT INTO：直接向表或表的分区中追加数据。不支持`INSERT INTO`到聚簇表。如果您需要插入少量测试数据，可以配合[VALUES](/intl.zh-CN/开发/SQL及函数/INSERT语句/VALUES.md)使用。
    -   INSERT OVERWRITE：先清空表中的原有数据，再向表或分区中插入数据。`INSERT OVERWRITE`不支持指定插入列的功能，只能用`INSERT INTO`。

        **说明：**

        -   MaxCompute的`INSERT`语法与通常使用的MySQL或Oracle的`INSERT`语法有差别。在`INSERT OVERWRITE|INTO`后需要加`TABLE`关键字，非直接使用`table_name`
        -   在反复对同一个分区执行`INSERT OVERWRITE`操作时，您通过`DESC`命令查看到的数据分区Size会不同。这是因为从同一个表的同一个分区`SELECT`出来再`INSERT OVERWRITE`回相同分区时，文件切分逻辑发生变化，从而导致数据的Size发生变化。数据的总长度在`INSERT OVERWRITE`前后是不变的，您不必担心存储计费会存在问题。
-   **参数说明**
    -   table\_name：需要插入数据的目标表名称。
    -   PARTITION \(partcol1=val1, partcol2=val2 ...\)：需要插入数据的分区名称，此参数不允许使用函数等表达式，只能是常量。
    -   \[\(col1,col2 ...\)\]：需要插入目标表的列名称。`INSERT OVERWRITE`不支持。
    -   select\_statement：SELECT子句，从源表中查询需要插入的数据。

        **说明：**

        -   源表与目标表的对应关系依赖于在SELECT子句中列的顺序，而不是表与表之间列名的对应关系。
        -   如果目标表是静态分区，向某个分区插入数据时，分区列不允许出现在SELECT子句中。
    -   from\_statement：FROM子句，代表数据来源。例如，源表名称。
    -   \[ZORDER BY zcol1 \[, zcol2 ...\]\]向表或分区写入数据时，支持根据指定的一列或多列（select\_statement对应表中的列），把排序列数据相近的行排列在一起，提升查询时的过滤性能，在一定程度上降低存储成本。需要注意的是，`ORDER BY x, y`会严格地按照先x后y的顺序对数据进行排序，`ZORDER BY x, y`会把相近的<x, y\>尽量排列在一起。当SQL查询语句的过滤条件中包含排序列时，ORDER BY后的数据仅对包含x的表达式有较好的过滤效果，ZORDER BY后的数据对包含x或同时包含x、y的表达式均有较好的过滤效果，列压缩比例更高。

        **说明：**

        -   使用ZORDER BY子句语句写入数据时，会占用较多资源，比不排序花费时间更多。
        -   目标表为聚簇表时，不支持ZORDER BY子句。

## 示例

计算`sale_detail`表中不同地区的销售额并存入表`sale_detail_insert`中。

```
--创建一张分区表sale_detail，并插入分区和数据。注意源表建表语句中，分区列不需要在普通列中重复定义。
CREATE TABLE IF NOT EXISTS sale_detail
(
shop_name     string,
customer_id   string,
total_price   double
)
PARTITIONED BY (sale_date STRING,region STRING);

--向源表增加分区。
ALTER TABLE sale_detail ADD PARTITION (sale_date='2013', region='china');

--向源表增加数据，其中INSERT INTO TABLE table_name可以简写为INSERT INTO table_name，但INSERT OVERWRITE TABLE table_name不可以省略TABLE关键字。
INSERT INTO sale_detail PARTITION (sale_date='2013', region='china') VALUES ('s1','c1',100.1),('s2','c2',100.2),('s3','c3',100.3);

--创建目标表sale_detail_insert，与源表有相同的结构。
CREATE TABLE sale_detail_insert LIKE sale_detail;

--给目标表增加分区。
ALTER TABLE sale_detail_insert ADD PARTITION (sale_date='2013', region='china');

--从源表sale_detail中取出数据插入目标表sale_detail_insert。
--注意不需要声明目标表字段，也不支持重排目标表字段顺序。
--对于静态分区目标表，分区字段赋值已经在PARTITION()部分声明，不需要在select_statement中包含，只要按照目标表普通列顺序查出对应字段，按顺序映射到目标表即可。动态分区表则需要在SELECT中包含分区字段，详情请参见[输出到动态分区（DYNAMIC PARTITION）](/intl.zh-CN/开发/SQL及函数/INSERT语句/输出到动态分区（DYNAMIC PARTITION）.md)。
INSERT OVERWRITE TABLE sale_detail_insert PARTITION (sale_date='2013', region='china')
  SELECT 
  shop_name, 
  customer_id,
  total_price 
  FROM sale_detail
  ZORDER BY customer_id, total_price;
```

您需要注意：

-   源表与目标表的对应关系依赖于在`SELECT`子句中列的顺序，而不是表与表之间列名的对应关系。命令示例如下。

    ```
    INSERT OVERWRITE TABLE sale_detail_insert PARTITION (sale_date='2013', region='china')
        SELECT customer_id, shop_name, total_price FROM sale_detail;                      
    ```

    在创建`sale_detail_insert`表时，列的顺序为`shop_name STRING、customer_id STRING、total_price BIGINT`，而从`sale_detail`向`sale_detail_insert`插入数据的顺序为`customer_id、shop_name、total_price`。此时，会将`sale_detail.customer_id`的数据插入`sale_detail_insert.shop_name`，将`sale_detail.shop_name`的数据插入`sale_detail_insert.customer_id`。

-   向某个分区插入数据时，分区列不允许出现在`SELECT`子句中。如下语句会返回报错，`sale_date`和`region`为分区列，不允许出现在静态分区的INSERT语句中。

    ```
    INSERT OVERWRITE TABLE sale_detail_insert PARTITION (sale_date='2013', region='china')
       SELECT shop_name, customer_id, total_price, sale_date, region FROM sale_detail;
    ```

-   `PARTITION`的值只能是常量，不可以为表达式。如下为错误用法。

    ```
    INSERT OVERWRITE TABLE sale_detail_insert PARTITION (sale_date=datepart('2016-09-18 01:10:00', 'yyyy') , region='china')
       SELECT shop_name, customer_id, total_price FROM sale_detail;
    ```


## 使用动态分区注意事项

如果您需要更新表数据到动态分区，请您注意：

-   `INSERT INTO PARTITION`时，如果分区不存在，会自动创建分区。
-   多个`INSERT INTO PARTITION`作业并发时，如果分区不存在，会自动创建分区，但只会成功创建一个分区。
-   如果不能控制`INSERT INTO PARTITION`作业并发，建议您通过`ALTER TABLE`命令提前创建分区，详情请参见[分区和列操作](/intl.zh-CN/开发/SQL及函数/DDL语句/分区和列操作.md)

更多动态分区详情请参见[输出到动态分区（DYNAMIC PARTITION）](/intl.zh-CN/开发/SQL及函数/INSERT语句/输出到动态分区（DYNAMIC PARTITION）.md)。

