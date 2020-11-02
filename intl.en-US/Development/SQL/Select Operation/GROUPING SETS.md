---
keyword: [GROUPING SETS, result grouping]
---

# GROUPING SETS

In some cases, you need to execute the UNION ALL clause multiple times to aggregate and analyze data from multiple dimensions. For example, you need to aggregate column a, aggregate column b, and then aggregate columns a and b. To address this issue, you can use the GROUPING SETS clause.

`GROUPING SETS` is an extension of the `GROUP BY` clause in a `SELECT` statement. The GROUPING SETS clause allows you to group your results in multiple ways, without the need to use multiple `SELECT` statements. This allows the MaxCompute engine to generate more efficient execution plans with higher performance.

**Note:** Most examples in this topic are demonstrated by using MaxCompute Studio. We recommend that you [install MaxCompute Studio](/intl.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md) before you proceed with subsequent operations.

## Example

1.  Prepare data.

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

2.  Use one of the following methods to group data:

    -   Execute multiple `SELECT` statements.

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

    -   Execute a single SELECT statement with the `GROUPING SETS` clause.

        ```
        SELECT os,device, city ,COUNT(*)
        FROM requests
        GROUP BY os, device, city GROUPING SETS((os, device), (city), ());
        ```

        The following result is returned:

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

    **Note:** NULL is used as placeholders for the expressions that are not used in GROUPING SETS. This way, you can perform UNION operations on the result sets, such as the city column in rows 1 to 5.


## CUBE and ROLLUP

CUBE and ROLLUP are special `GROUPING SETS` functions. `CUBE` lists all possible combinations of specific columns as grouping sets. `ROLLUP` aggregates data by level to generate grouping sets.

Example:

```
GROUP BY CUBE(a, b, c)  
-- Equivalent to the following statement:  
GROUPING SETS((a,b,c),(a,b),(a,c),(b,c),(a),(b),(c),())

GROUP BY ROLLUP(a, b, c)
-- Equivalent to the following statement:  
GROUPING SETS((a,b,c),(a,b),(a), ())

GROUP BY CUBE ( (a, b), (c, d) ) 
-- Equivalent to the following statement: 
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b       ),
    (       c, d ),
    (            )
)

GROUP BY ROLLUP ( a, (b, c), d ) 
-- Equivalent to the following statement:
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b, c    ),
    ( a          ),
    (            )
)

GROUP BY a, CUBE (b, c), GROUPING SETS ((d), (e)) 
-- Equivalent to the following statement: 
GROUP BY GROUPING SETS (
    (a, b, c, d), (a, b, c, e),
    (a, b, d),    (a, b, e),
    (a, c, d),    (a, c, e),
    (a, d),       (a, e)
)

GROUP BY grouping sets((b), (c),rollup(a,b,c)) 
-- Equivalent to the following statement: 
GROUP BY GROUPING SETS (
    (b), (c),
    (a,b,c), (a,b), (a), ()
 )
```

## GROUPING and GROUPING\_ID

NULL is used as placeholders in the results of `GROUPING SETS`. As a result, the NULL placeholders cannot be distinguished from the NULL values. To address this issue, MaxCompute provides the `GROUPING` function. `GROUPING` allows you to specify the name of a column as a parameter. If specific rows are aggregated based on such a column, 0 is returned, which indicates that NULL is a value. Otherwise, 1 is returned, which indicates that NULL is a placeholder in `GROUPING SETS`.

`GROUPING_ID` allows you to specify the names of one or more columns as parameters. The results of `GROUPING` in these columns are formed into integers by using bitmap.

Example:

```
SELECT a,b,c ,COUNT(*),
GROUPING(a) ga, GROUPING(b) gb, GROUPING(c) gc, GROUPING_ID(a,b,c) groupingid
FROM VALUES (1,2,3) as t(a,b,c)
GROUP BY CUBE(a,b,c);
```

The following result is returned:

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

By default, the columns that are not used in GROUP BY are filled with NULL. You can use the GROUPING function to obtain more useful data.

```
SELECT 
  IF(GROUPING(os) == 0, os, 'ALL') as os,
  IF(GROUPING(device) == 0, device, 'ALL') as device, 
  IF(GROUPING(city) == 0, city, 'ALL') as city ,
  COUNT(*) as count
FROM requests
GROUP BY os, device, city GROUPING SETS((os, device), (city), ());
```

The following figure shows the returned result.

![Returned result](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0209824061/p98799.png)

MaxCompute also provides a parameterless function named GROUPING\_ID for Hive queries.

```
SELECT     
a,b,c ,COUNT(*),GROUPING_ID
FROM VALUES (1,2,3) as t(a,b,c)
GROUP BY a, b, c GROUPING SETS ((a,b,c), (a));
```

GROUPING\_ID does not contain input parameters and parentheses \(\(\)\). GROUPING\_ID is equivalent to GROUPING\_ID\(a,b,c\) in MaxCompute. The order of the parameters in this function is the same as that in GROUP BY.

We recommend that you use this function for Hive 2.3.0 and later.

