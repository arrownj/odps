---
keyword: MAPJOIN
---

# MAPJOIN HINT

当一个大表和一个或多个小表JOIN时，您可以在SELECT语句中显式指定MAPJOIN以提升查询性能。

## 背景信息

通常情况下，JOIN操作在Reduce阶段执行表连接。整个JOIN过程包含Map、Shuffle和Reduce三个阶段。

MAPJOIN在Map阶段执行表连接，而非等到Reduce阶段才执行表连接。这样就节省了大量数据传输的时间以及系统资源，从而起到了优化作业的作用。

MAPJOIN在Map阶段会将指定表的数据全部加载在内存中。因此指定的表仅能为小表，且表被加载到内存后占用的总内存不得超过512 MB。

在大表和一个或多个小表JOIN的场景下，MAPJOIN会将您指定的小表全部加载到执行JOIN操作的程序的内存中，在Map阶段完成表连接从而加快JOIN的执行速度。

## 使用方法

您需要在SELECT语句中使用Hint提示`/* + MAPJOIN(table) */`才会执行MAPJOIN。

假设表sale\_detail为大表，别名为b，表shop为小表，别名为a。

-   普通JOIN操作，查询语句如下。

    ```
    SELECT  a.shop_name,
            b.customer_id,
            b.total_price
    FROM shop a JOIN sale_detail b
    ON a.shop_name = b.shop_name;
    ```

-   使用MAPJOIN，查询语句如下。

    ```
    SELECT /* + MAPJOIN(a) */
            a.shop_name,
            b.customer_id,
            b.total_price
    FROM shop a JOIN sale_detail b
    ON a.shop_name = b.shop_name;
    ```


## 使用说明

-   使用MAPJOIN时，在引用小表或子查询时，需要引用别名。
-   MAPJOIN支持小表为子查询。
-   LEFT OUTER JOIN的左表必须是大表。
-   RIGHT OUTER JOIN的右表必须是大表。
-   INNER JOIN的左表或右表均可以作为大表。
-   FULL OUTER JOIN不能使用MAPJOIN。
-   在MAPJOIN中，可以使用不等值连接或者OR连接多个条件。您可以通过不写ON语句而通过`MAPJOIN ON 1 = 1`的形式，实现笛卡尔乘积的计算，例如`SELECT /* + MAPJOIN(a) */ a.id FROM shop a JOIN table_name b ON 1=1`，但此操作可能带来数据量膨胀问题。
-   MaxCompute在MAPJOIN中最多支持指定128张小表，否则报语法错误。MAPJOIN中多个小表用逗号隔开，例如`/*+MAPJOIN(a,b,c)*/`。
-   如果使用MAPJOIN，则小表占用的总内存不得超过512 MB。由于MaxCompute是压缩存储，因此小表在被加载到内存后，数据大小会急剧膨胀。此处的512 MB是指加载到内存后的空间大小。

## 示例

MaxCompute SQL不支持在普通JOIN的ON条件中使用不等值表达式、OR逻辑等复杂的JOIN条件，但是在MAPJOIN中可以进行上述操作。

```
SELECT /* + MAPJOIN(a) */
        a.total_price,
        b.total_price
FROM shop a JOIN sale_detail b
ON a.total_price < b.total_price OR a.total_price + b.total_price < 500;
```

MAPJOIN多个小表示例请参见[如何用MAPJOIN缓存多张小表？](/cn.zh-CN/常见问题/SQL/SQL语句.md)。

