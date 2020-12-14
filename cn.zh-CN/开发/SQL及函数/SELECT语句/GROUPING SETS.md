---
keyword: [GROUPING SETS, 结果分组]
---

# GROUPING SETS

对于经常需要对数据进行多维度的聚合分析的场景，您既需要对a列做聚合，也要对b列做聚合，同时要按照a、b两列同时做聚合，因此需要多次使用UNION ALL。使用GROUPING SETS可以快速解决此类问题。

MaxCompute中的`GROUPING SETS`是对`SELECT`语句中`GROUP BY`子句的扩展，允许您采用多种方式对结果分组，而不必使用多个`SELECT`语句来实现这一目的。这样能够使MaxCompute的引擎给出更有效的执行计划，从而提高执行性能。

**说明：** 本文中大部分示例采用MaxCompute Studio进行展示，建议您[安装MaxCompute Studio](/cn.zh-CN/工具及下载/MaxCompute Studio/工具安装与版本信息/安装IntelliJ IDEA.md)后，再进行后续操作。

## 实现示例

1.  准备数据。

    ```
    create table requests LIFECYCLE 20 as
    select * from values
        (1, 'windows', 'PC', 'Beijing'),
        (2, 'windows', 'PC', 'Shijiazhuang'),
        (3, 'linux', 'Phone', 'Beijing'),
        (4, 'windows', 'PC', 'Beijing'),
        (5, 'ios', 'Phone', 'Shijiazhuang'),
        (6, 'linux', 'PC', 'Beijing'),
        (7, 'windows', 'Phone', 'Shijiazhuang')
    as t(id, os, device, city);
    ```

2.  对数据进行分组。您可以通过以下两种方式进行分组：

    -   使用多个`SELECT`语句进行分组。

        ```
        SELECT NULL, NULL, NULL, COUNT(*)
        FROM requests
        UNION ALL
        SELECT os, device, NULL, COUNT(*)
        FROM requests GROUP BY os, device
        UNION ALL
        SELECT null, null, city, COUNT(*)
        FROM requests GROUP BY city;
        ```

    -   使用`GROUPING SETS`进行分组。

        ```
        SELECT os,device, city ,COUNT(*)
        FROM requests
        GROUP BY os, device, city GROUPING SETS((os, device), (city), ());
        ```

        执行结果如下。

        ```
        +----+--------+------+------------+
        | os | device | city | cnt        |
        +----+--------+------+------------+
        | NULL | NULL   | NULL | 7          |
        | NULL | NULL   | Beijing | 4          |
        | NULL | NULL   | Shijiazhuang | 3          |
        | ios | Phone  | NULL | 1          |
        | linux | PC     | NULL | 1          |
        | linux | Phone  | NULL | 1          |
        | windows | PC     | NULL | 3          |
        | windows | Phone  | NULL | 1          |
        +----+--------+------+------------+
        ```

    **说明：** 分组集中不使用的表达式，会使用NULL充当占位符，使得这些结果集可以做UNION操作。例如结果第1-5行的city列。


## CUBE and ROLLUP函数

CUBE和ROLLUP可以认为是特殊的`GROUPING SETS`。CUBE会枚举指定列的所有可能组合作为`GROUPING SETS`，而ROLLUP会以按层级聚合的方式产生`GROUPING SETS`。

示例如下。

```
GROUP BY CUBE(a, b, c)  
--等价于以下语句。  
GROUPING SETS((a,b,c),(a,b),(a,c),(b,c),(a),(b),(c),())

GROUP BY ROLLUP(a, b, c)
--等价于以下语句。  
GROUPING SETS((a,b,c),(a,b),(a), ())

GROUP BY CUBE ( (a, b), (c, d) ) 
--等价于以下语句。 
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b       ),
    (       c, d ),
    (            )
)

GROUP BY ROLLUP ( a, (b, c), d ) 
--等价于以下语句。
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b, c    ),
    ( a          ),
    (            )
)

GROUP BY a, CUBE (b, c), GROUPING SETS ((d), (e)) 
--等价于以下语句。 
GROUP BY GROUPING SETS (
    (a, b, c, d), (a, b, c, e),
    (a, b, d),    (a, b, e),
    (a, c, d),    (a, c, e),
    (a, d),       (a, e)
)

GROUP BY grouping sets((b), (c),rollup(a,b,c)) 
--等价于以下语句。 
GROUP BY GROUPING SETS (
    (b), (c),
    (a,b,c), (a,b), (a), ()
 )
```

## GROUPING和GROUPING\_ID函数

`GROUPING SETS`结果中使用NULL充当占位符，导致您会无法区分占位符NULL与数据中真正的NULL。因此，MaxCompute为您提供了`GROUPING`函数。`GROUPING`函数接受一个列名作为参数，如果结果对应行使用了参数列做聚合，返回0，此时意味着NULL来自输入数据。否则返回1，此时意味着NULL是`GROUPING SETS`的占位符。

MaxCompute还提供了`GROUPING_ID`函数，此函数接受一个或多个列名作为参数。结果是将参数列的`GROUPING`结果按照BitMap的方式组成整数。

示例如下。

```
SELECT a,b,c ,COUNT(*),
GROUPING(a) ga, GROUPING(b) gb, GROUPING(c) gc, GROUPING_ID(a,b,c) groupingid
FROM VALUES (1,2,3) as t(a,b,c)
GROUP BY CUBE(a,b,c);
```

执行结果如下。

```
+------------+------------+------------+------------+------------+------------+------------+------------+
| a          | b          | c          | _c3        | ga         | gb         | gc         | groupingid |
+------------+------------+------------+------------+------------+------------+------------+------------+
| NULL       | NULL       | NULL       | 1          | 1          | 1          | 1          | 7          |
| NULL       | NULL       | 3          | 1          | 1          | 1          | 0          | 6          |
| NULL       | 2          | NULL       | 1          | 1          | 0          | 1          | 5          |
| NULL       | 2          | 3          | 1          | 1          | 0          | 0          | 4          |
| 1          | NULL       | NULL       | 1          | 0          | 1          | 1          | 3          |
| 1          | NULL       | 3          | 1          | 0          | 1          | 0          | 2          |
| 1          | 2          | NULL       | 1          | 0          | 0          | 1          | 1          |
| 1          | 2          | 3          | 1          | 0          | 0          | 0          | 0          |
+------------+------------+------------+------------+------------+------------+------------+------------+
```

默认情况，GROUP BY列表中不被使用的列，会被填充为NULL。您可以通过GROUPING函数输出更有实际意义的值。

```
SELECT 
  IF(GROUPING(os) == 0, os, 'ALL') as os,
  IF(GROUPING(device) == 0, device, 'ALL') as device, 
  IF(GROUPING(city) == 0, city, 'ALL') as city ,
  COUNT(*) as count
FROM requests
GROUP BY os, device, city GROUPING SETS((os, device), (city), ());
```

返回结果如下。

![返回结果](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/1782659951/p98799.png)

MaxCompute还提供了无参数的GROUPING\_\_ID函数，用于兼容Hive查询。

```
SELECT     
a,b,c ,COUNT(*),GROUPING__ID
FROM VALUES (1,2,3) as t(a,b,c)
GROUP BY a, b, c GROUPING SETS ((a,b,c), (a));
```

GROUPING\_\_ID既无输入参数，也无括号。此表达方式在MaxCompute种等价于GROUPING\_ID\(a,b,c\)，参数与GROUP BY的顺序一致。

MaxCompute和Hive 2.3.0及以上版本兼容该函数，在Hive 2.3.0以下版本中该函数输出不一致，因此并不推荐您使用此函数。

