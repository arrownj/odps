---
keyword: [排序操作, 分组查询, 嵌套查询]
---

# SELECT语法介绍

本文为您介绍MaxCompute SELECT语法格式及使用SELECT语法执行嵌套查询、排序操作、分组查询等操作的注意事项。

## SELECT语法格式

```
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
FROM table_reference
[WHERE where_condition]
[GROUP BY col_list]
[ORDER BY order_condition]
[DISTRIBUTE BY distribute_condition [SORT BY sort_condition] ]
[LIMIT number]
```

## 使用限制

-   当使用SELECT语句时，屏显最多只能显示10000行结果。当SELECT语句作为子句时则无此限制，SELECT子句会将全部结果返回给上层查询。
-   SELECT语句查询分区表时禁止全表扫描。

    2018年01月10日20点后创建的新项目执行SQL时，默认情况下，针对该Project里的分区表不允许全表扫描。在查询分区表数据时必须指定分区，由此减少SQL的不必要I/O，从而减少计算资源的浪费以及按量付费模式下不必要的计算费用。

    如果您需要对分区表进行全表扫描，可以在对分区表全表扫描的SQL语句前加上命令`set odps.sql.allow.fullscan=true;`，并和SQL语句一起提交执行。假设`sale_detail`表为分区表，需要同时执行如下语句进行全表查询。

    ```
    set odps.sql.allow.fullscan=true;
    SELECT * FROM sale_detail;
    ```

    如果整个项目都需要全表扫描，执行如下命令打开开关。

    ```
    setproject odps.sql.allow.fullscan=true;
    ```


## 列表达式（select\_expr）

SELECT操作从表中读取数据，列表达式有以下几种形式：

-   用列名指定要读取的列。例如，读取表`sale_detail`的列`shop_name`。

    ```
    SELECT shop_name FROM sale_detail;
    ```

-   用`*`代表所有的列。读取表`sale_detail`中所有的列。

    ```
    SELECT * FROM sale_detail;
    ```

    在`WHERE`中可以指定过滤的条件。

    ```
    SELECT * FROM sale_detail WHERE shop_name LIKE 'hang%';
    ```

-   `select_expr`支持正则表达式。示例如下：

    -   `SELECT `abc.*` FROM t;`选出`t`表中所有列名以`abc`开头的列。
    -   `SELECT `(ds)?+.+` FROM t;`选出`t`表中列名不为`ds`的所有列。
    -   `SELECT `(ds|pt)?+.+` FROM t;`选出`t`表中排除`ds`和`pt`两列的其它列。
    -   `SELECT `(d.*)?+.+` FROM t;`选出`t`表中排除列名以`d`开头的其它列。
    **说明：**

    在排除多个列时，如果col2是col1的前缀，则需保证col1写在col2的前面（较长的col写在前面）。例如，一个表有2个分区无需被查询，一个分区名为`ds`，另一个为`dshh`，由于前者是后者的前缀，则正确表达式为`SELECT `(dshh|ds)?+.+` FROM tbl;`；错误表达式为`SELECT `(ds|dshh)?+.+` FROM tbl;`。

-   DISTINCT去重。您可以在选取的列名前使用`DISTINCT`去掉重复字段，只返回一个值；而使用`ALL`会返回字段中所有重复的值。不指定此选项时，默认值为`ALL`。

    示例如下。

    ```
    --查询表sale_detail中region列数据，如果有重复值时仅显示一条。
    SELECT DISTINCT region FROM sale_detail;
    +------------+
    | region     |
    +------------+
    | shanghai   |
    +------------+
    --DISTINCT多列时，DISTINCT的作用域是SELECT的列集合，不是单个列。
    SELECT DISTINCT region, sale_date FROM sale_detail;
    +------------+------------+
    | region     | sale_date  |
    +------------+------------+
    | shanghai   | 20191110   |
    +------------+------------+
    ```


## 查询的目标表信息（table\_reference）

`table_reference`为查询的目标表信息。除了支持已存在的目标表名称还支持使用嵌套子查询，命令示例如下。

```
SELECT * FROM (SELECT region FROM sale_detail) t WHERE region = 'shanghai';
```

## WHERE子句过滤（where\_condition）

`WHERE`子句支持的过滤条件请参见下表。

|过滤条件|描述|
|:---|:-|
|\>、<、=、\>=、<=、<\>|关系操作符。|
|LIKE、RLIKE|LIKE和RLIKE的Source和Pattern参数均仅接受STRING类型。|
|IN、NOT IN|如果在IN\|NOT IN条件后加子查询，子查询只能返回一列值，且返回值的数量不能超过1000。|
|BETWEEN…AND|限定查询范围。|

**示例**

-   在SELECT语句的`WHERE`子句中，您可以指定分区范围，只扫描表的指定部分，避免全表扫描，命令示例如下。

    ```
    SELECT sale_detail.* 
    FROM sale_detail
    WHERE sale_detail.sale_date >= '2008'
    AND sale_detail.sale_date <= '2014';
    ```

    **说明：** 您可以通过`EXPLAIN SELECT`语句查看分区裁剪是否生效。普通的UDF或`JOIN`的分区条件写法都有可能导致分区裁剪不生效，详情请参见[分区剪裁合理性评估](/cn.zh-CN/最佳实践/SQL/分区剪裁合理性评估.md)。

-   UDF支持分区裁剪。支持的方式是将UDF语句先当作一个小作业执行，再将执行的结果替换到原来UDF出现的位置。实现的方式有以下两种：
    -   在编写UDF的时候，UDF类上加入Annotation。

        ```
        @com.aliyun.odps.udf.annotation.UdfProperty(isDeterministic=true)
        ```

        **说明：** `com.aliyun.odps.udf.annotation.UdfProperty`定义在odps-sdk-udf.jar文件中。您需要把引用的odps-sdk-udf版本提高到0.30.x或以上。

    -   在SQL语句前增加设置Flag语句`set odps.sql.udf.ppr.deterministic = true;`，此时SQL中所有的UDF均被视为`deterministic`。该操作执行的原理是进行执行结果回填，但是结果回填最多回填1000个分区。因此，如果UDF类加入Annotation，则可能会导致出现超过1000个回填结果的报错。此时您如果需要忽视此错误，可以通过设置Flag`set odps.sql.udf.ppr.to.subquery = false;`全局关闭此功能。关闭后，UDF分区裁剪也会失效。
-   `BETWEEN…AND`查询示例如下。

    ```
    SELECT sale_detail.* 
    FROM sale_detail 
    WHERE sale_detail.sale_date BETWEEN '2008' AND '2014';
    ```


## GROUP BY分组查询

通常，`GROUP BY`和[聚合函数](/cn.zh-CN/开发/SQL及函数/内建函数/聚合函数.md)配合使用。当`SELECT`中包含聚合函数时，有以下规则：

-   在SQL解析中，`GROUP BY`操作先于`SELECT`操作，因此`GROUP BY`的取值是`SELECT`输入表的列名或者由输入表的列构成的表达式，不允许是`SELECT`语句的输出列的别名。
-   `GROUP BY`的值既是输入表的列或表达式，又是`SELECT`的输出列时，取值为输入表的列名。
-   当SQL语句设置了Flag，即`set hive.groupby.position.alias=true;`，`GROUP BY`中的整型常量会被当做SELECT的列序号处理。

    ```
    set hive.groupby.position.alias=true;--与下一条SQL语句一起执行。
    SELECT region, SUM(total_price) FROM sale_detail GROUP BY 1;-- 1代表select的列中第一列即region，以region值分组，返回每一组的region值(组内唯一)及销售额总量。
    ```


**示例**

```
--直接使用输入表列名region作为GROUP BY的列，即以region值分组。
SELECT region FROM sale_detail GROUP BY region;
--以region值分组，返回每一组的销售额总量。
SELECT SUM(total_price) FROM sale_detail GROUP BY region;
--以region值分组，返回每一组的region值（组内唯一）及销售额总量。
SELECT region, SUM(total_price) FROM sale_detail GROUP BY region;
--使用SELECT列的别名运行，返回报错。
SELECT region AS r FROM sale_detail GROUP BY r;
--必须使用列的完整表达式。
SELECT 2 + total_price AS r FROM sale_detail GROUP BY 2 + total_price;
--返回报错，SELECT的所有列中没有使用聚合函数的列，必须出现在GROUP BY中。
SELECT region, total_price FROM sale_detail GROUP BY region;
--没有使用聚合函数的列出现在GROUP BY中后，运行成功。
SELECT region, total_price FROM sale_detail GROUP BY region, total_price;     
```

## ORDER BY\|DISTRIBUTE BY\|SORT BY

-   `ORDER BY`

    功能说明：用于对所有数据按照指定列进行全局排序。

    -   对记录进行降序排序，需要使用`DESC`关键字。默认以升序排列。
    -   在使用`ORDER BY`排序时，NULL会被认为比任何值都小，这个行为与MySQL一致，但是与Oracle不一致。
    -   `ORDER BY`后面需要加上`SELECT`列的别名。当`SELECT`某列时，如果没有指定列的别名，则列名会被作为列的别名。
    -   当SQL语句设置了Flag，即`set hive.orderby.position.alias=true;`，ORDER BY中的整型常量会被当做SELECT的列序号处理。

        ```
        --设置Flag。
        set hive.orderby.position.alias=true;
        --创建表src。
        CREAT table src(key BIGINT，value BIGINT);
        --以value值分组，返回表src的所有信息。
        SELECT * FROM src ORDER BY 2 LIMIT 100;
        等同于
        SELECT * FROM src ORDER BY value LIMIT 100;
        ```

    -   OFFSET和ORDER BY LIMIT语句配合，可以指定跳过的行数。

        ```
        --将src按照key从小到大排序后，输出第11到第30行（OFFSET 10指定跳过前10行，LIMIT 20指定最多输出20行）。
        SELECT * FROM src ORDER BY key LIMIT 20 OFFSET 10；
        ```

    **示例**

    ```
    --查询表sale_detail的信息，并按照region升序排列前100条。
    SELECT * FROM sale_detail ORDER BY region LIMIT 100;
    --ORDER BY默认要求带LIMIT数据行数限制，没有LIMIT会返回报错。如您需要解除ORDER BY必须带LIMIT的限制，请参见[LIMIT NUMBER限制输出行数\>解除ORDER BY必须带LIMIT的限制](#section_low_fwz_gvu)。
    SELECT * FROM sale_detail ORDER BY region;
    --ORDER BY加列的别名。
    SELECT region AS r FROM sale_detail ORDER BY region LIMIT 100;
    SELECT region AS r FROM sale_detail ORDER BY r LIMIT 100;
    ```

-   `DISTRIBUTE BY`

    功能说明：用于对数据按照某几列的值做Hash分片，必须使用`SELECT`的输出列别名。

    **示例**

    ```
    --查询表sale_detail中的列region值并按照region值进行哈希分片。
    SELECT region FROM sale_detail DISTRIBUTE BY region;
    -- 列名即是别名，可以运行。
    SELECT region AS r FROM sale_detail DISTRIBUTE BY region;
    等同于
    SELECT region AS r FROM sale_detail DISTRIBUTE BY r;
    ```

-   `SORT BY`

    功能说明：局部排序。您可以在字段后面加上关键字，ASC表示升序，DESC表示降序，不带关键字默认为升序。

    -   如果`SORT BY`语句前有`DISTRIBUTE BY`，`SORT BY`对`DISTRIBUTE BY`的结果进行局部排序。`DISTRIBUTE BY`控制map的输出在reduer中是如何划分的，如果不希望reducer的输出内容存在重叠，或需要对同一分组的数据一起处理，您可以使用`DISTRIBUTE BY`来保证同组数据分发到同一个reducer中，然后使用`SORT BY`对局部数据进行排序。命令示例如下。

        ```
        --查询表sale_detail中的列region值并按照region值进行哈希分片，然后对哈希分片结果进行局部排序。
        SELECT region FROM sale_detail DISTRIBUTE BY region SORT BY region;
        ```

    -   如果`SORT BY`语句前没有`DISTRIBUTE BY`，`SORT BY`对每个reduce中的数据进行排序，同样是执行一个局部排序过程。这可以保证每个reduce的输出数据都是有序的，从而增加存储压缩率，同时读取时如果有过滤，能够减少真正从磁盘读取的数据量，提高后续全局排序的效率。
    **示例**

    ```
    SELECT region FROM sale_detail SORT BY region;
    ```


**说明：**

-   `ORDER BY|DISTRIBUTE BY|SORT BY`的取值必须是`SELECT`语句的输出列，即列的别名。列的别名可以为中文。
-   在MaxCompute SQL解析中，`ORDER BY|DISTRIBUTE BY|SORT BY`在`SELECT`操作之后，因此它们的取值只能为`SELECT`语句的输出列。
-   `ORDER BY`不和`DISTRIBUTE BY`、`SORT BY`同时使用，同时`GROUP BY`也不和`DISTRIBUTE BY`、`SORT BY`同时使用。

## LIMIT NUMBER限制输出行数

`LIMIT number`中的`number`是常数，限制输出行数。

**解除`ORDER BY`必须带`LIMIT`的限制**

因为`ORDER BY`需要对单个执行节点做全局排序，所以默认带`LIMIT`限制，避免误用导致单点处理大量数据。如果您的使用场景确实需要`ORDER BY`放开`LIMIT`限制，可以通过如下两种方式实现：

-   Project级别：设置`setproject odps.sql.validate.orderby.limit=false;`关闭`ORDER BY`必须带`LIMIT`的限制。
-   Session级别：设置`setproject odps.sql.validate.orderby.limit=false;`关闭`ORDER BY`必须带`LIMIT`的限制，需要与SQL语句一起提交。

    **说明：** 如果关闭`ORDER BY`必须带`LIMIT`的限制，在单个执行节点有大量数据排序的情况下，资源消耗或处理时长等性能表现会受到影响。


**屏显限制说明**

当使用无`LIMIT`的`SELECT`语句或`LIMIT`的`number`数量超过设置的屏显上限时，如果您直接从屏显窗口查看结果，最多只能输出屏显上限设置的行数。

每个项目空间的屏显上限可能不同，您可以参考如下方法控制：

-   如果关闭了项目空间数据保护，修改odpscmd config.ini文件。

    设置odpscmd config.ini文件中的`use_instance_tunnel=true`，如果不配置`instance_tunnel_max_record`参数，则屏显行数不受限制；否则，屏显行数受`instance_tunnel_max_record`参数值限制。`instance_tunnel_max_record`参数值上限为10000行。Instance Tunnel详情请参见[Tunnel命令使用说明](/cn.zh-CN/开发/数据上传下载/使用Tunnel命令上传下载数据/Tunnel命令使用说明.md)。

-   如果开启了项目空间数据保护，屏显行数受`READ_TABLE_MAX_ROW`参数值限制，配置上限为10000行。

**说明：** 您可以执行`show SecurityConfiguration;`命令查看`ProjectProtection`属性配置。如果`ProjectProtection=true`，根据项目空间数据保护需求判断是否关闭数据保护机制。如果可以关闭，通过`set ProjectProtection=false;`命令关闭。`ProjectProtection`属性默认不开启。项目空间数据保护机制详情请参见[项目空间的数据保护](/cn.zh-CN/管理/安全管理详解/项目空间的数据保护.md)。

