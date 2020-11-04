---
keyword: JOIN
---

# MaxCompute SQL中的关联操作

本文为您介绍MaxCompute SQL中的关联（JOIN）操作。

## 概述

JOIN类型如下所示。

|类型|说明|
|:-|:-|
|INNER JOIN|输出符合关联条件的数据。|
|LEFT JOIN|输出左表的所有记录，以及右表中符合关联条件的数据。右表中不符合关联条件的行，输出NULL。|
|RIGHT JOIN|输出右表的所有记录，以及左表中符合关联条件的数据。左表中不符合关联条件的行，输出NULL。|
|FULL JOIN|输出左表和右表的所有记录，对于不符合关联条件的数据，未关联的另一侧输出NULL。|
|LEFT SEMI JOIN|对于左表中的一条数据，如果右表存在符合关联条件的行，则输出左表。|
|LEFT ANTI JOIN|对于左表中的一条数据，如果右表中不存在符合关联条件的数据，则输出左表。|

SQL语句中，同时存在JOIN和WHERE子句时，如下所示。

```
(SELECT * FROM A WHERE {subquery_where_condition} A) A
JOIN
(SELECT * FROM B WHERE {subquery_where_condition} B) B
ON {on_condition}
WHERE {where_condition}
```

计算顺序如下：

1.  子查询中的WHERE子句（即`{subquery_where_condition}`）。
2.  JOIN子句中的关联条件（即`{on_condition}`）。
3.  JOIN结果集中的WHERE子句（即`{where_condition}`）。

因此，对于不同的JOIN类型，过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`中时，查询结果可能一致，也可能不一致。详情请参见[场景说明](#section_omw_m9j_d6i)。

## 示例数据

-   表A

    建表语句如下。

    ```
    CREATE TABLE A AS SELECT * FROM VALUES (1, 20180101),(2, 20180101),(2, 20180102) t (key, ds);
    ```

    示例数据如下。

    |key|ds|
    |:--|:-|
    |1|20180101|
    |2|20180101|
    |2|20180102|

-   表B

    建表语句如下。

    ```
    CREATE TABLE B AS SELECT * FROM VALUES (1, 20180101),(3, 20180101),(2, 20180102) t (key, ds);
    ```

    示例数据如下。

    |key|ds|
    |:--|:-|
    |1|20180101|
    |3|20180101|
    |2|20180102|

-   表A和表B的笛卡尔乘积如下。

    |a.key|a.ds|b.key|b.ds|
    |:----|:---|:----|:---|
    |1|20180101|1|20180101|
    |1|20180101|3|20180101|
    |1|20180101|2|20180102|
    |2|20180101|1|20180101|
    |2|20180101|3|20180101|
    |2|20180101|2|20180102|
    |2|20180102|1|20180101|
    |2|20180102|3|20180101|
    |2|20180102|2|20180102|


## 场景说明

-   INNER JOIN

    INNER JOIN对左右表执行笛卡尔乘积，然后输出满足ON表达式的行。

    结论：过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`中时，查询结果是一致的。

    -   情况1：过滤条件在子查询`{subquery_where_condition}`中。

        ```
        SELECT A.*, B.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        结果如下。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|

    -   情况2：过滤条件在JOIN的关联条件`{on_condition}`中。

        ```
        SELECT A.*, B.*
        FROM A JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        笛卡尔积结果为9条，满足关联条件的结果只有1条，如下。

        |a.key|a.ds|b.key|b.ds|
        |1|20180101|1|20180101|

    -   情况3：过滤条件在JOIN结果集的WHERE子句中。

        ```
        SELECT A.*, B.*
        FROM A JOIN B
        ON a.key = b.key
        WHERE A.ds='20180101' and B.ds='20180101';
        ```

        笛卡尔积的结果为9条，满足关联条件的结果有3条，如下。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180102|2|20180102|
        |2|20180101|2|20180102|

        对上述结果执行JOIN结果集中的过滤条件`A.ds='20180101' and B.ds='20180101'`，结果只有1条，如下。

        |a.key|a.ds|b.key|b.ds|
        |-----|----|-----|----|
        |1|20180101|1|20180101|

-   LEFT JOIN

    LEFT JOIN对左右表执行笛卡尔乘积，输出满足ON表达式的行。对于左表中不满足ON表达式的行，输出左表，右表输出NULL。

    结论：过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`中时，查询结果不一致。

    -   左表的过滤条件在`{subquery_where_condition}`和`{where_condition}`中时，查询结果是一致的。
    -   右表的过滤条件在`{subquery_where_condition}`和`{on_condition}`中时，查询结果是一致的。
    -   情况1：过滤条件在子查询`{subquery_where_condition}`中。

        ```
        SELECT A.*, B.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        LEFT JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        结果如下。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|NULL|NULL|

    -   情况2：过滤条件在JOIN的关联条件`{on_condition}`中。

        ```
        SELECT A.*, B.*
        FROM A JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        笛卡尔积的结果有9条，满足关联条件的结果只有1条。左表输出剩余不满足关联条件的两条记录，右表输出NULL。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|NULL|NULL|
        |2|20180102|NULL|NULL|

    -   情况3：过滤条件在JOIN结果集的WHERE子句中。

        ```
        SELECT A.*, B.*
        FROM A JOIN B
        ON a.key = b.key
        WHERE A.ds='20180101' and B.ds='20180101';
        ```

        笛卡尔积的结果为9条，满足ON条件的结果有3条。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|2|20180102|
        |2|20180102|2|20180102|

        对上述结果执行JOIN结果集中的过滤条件`A.ds='20180101' and B.ds='20180101'`，结果只有1条。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|

-   RIGHT JOIN

    RIGHT JOIN和LEFT JOIN是类似的，只是左右表的区别。

    -   过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`时，查询结果不一致。
    -   右表的过滤条件，在`{subquery_where_condition}`和`{where_condition}`中时，查询结果一致。
    -   左表的过滤条件，放在`{subquery_where_condition}`和`{on_condition}`中时，查询结果一致。
-   FULL JOIN

    FULL JOIN对左右表执行笛卡尔乘积，然后输出满足关联条件的行。对于左右表中不满足关联条件的行，输出有数据表的行，无数据的表输出NULL。

    结论：过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`时，查询结果不一致。

    -   情况1：过滤条件在子查询`{subquery_where_condition}`中。

        ```
        SELECT A.*, B.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        FULL JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        结果如下。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|NULL|NULL|
        |NULL|NULL|3|20180101|

    -   情况2：过滤条件在JOIN的关联条件`{on_condition}`中。

        ```
        SELECT A.*, B.*
        FROM A FULL JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        笛卡尔积的结果有9条，满足关联条件的结果只有1条。对于左表不满足关联条件的两条记录输出左表数据，右表输出NULL。对于右表不满足关联条件的两条记录输出右表数据，左表输出NULL。

        |a.key|a.ds|b.key|b.ds|
        |-----|----|-----|----|
        |1|20180101|1|20180101|
        |2|20180101|NULL|NULL|
        |2|20180102|NULL|NULL|
        |NULL|NULL|3|20180101|
        |NULL|NULL|2|20180102|

    -   情况3：过滤条件在JOIN结果集的WHERE子句中。

        ```
        SELECT A.*, B.*
        FROM A FULL JOIN B
        ON a.key = b.key
        WHERE A.ds='20180101' and B.ds='20180101';
        ```

        笛卡尔积的结果有9条，满足关联条件的结果有3条。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|2|20180102|
        |2|20180102|2|20180102|

        对于不满足关联条件的表输出数据，另一表输出NULL。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|2|20180102|
        |2|20180102|2|20180102|
        |NULL|NULL|3|20180101|

        对上述结果执行JOIN结果集中的过滤条件`A.ds='20180101' and B.ds='20180101'`，结果只有1条。

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|

-   LEFT SEMI JOIN

    LEFT SEMI JOIN将左表的每一条记录，和右表进行匹配。如果匹配成功，则输出左表。如果匹配不成功，则跳过。由于只输出左表，所以JOIN后的WHERE条件中不涉及右表。

    结论：过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`中时，查询结果是一致的。

    -   情况1：过滤条件在子查询\{subquery\_where\_condition\}中。

        ```
        SELECT A.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        LEFT SEMI JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        结果如下。

        |a.key|a.ds|
        |:----|:---|
        |1|20180101|

    -   情况2：过滤条件在JOIN的关联条件`{on_condition}`中。

        ```
        SELECT A.*
        FROM A LEFT SEMI JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        结果如下。

        |a.key|a.ds|
        |:----|:---|
        |1|20180101|

    -   情况3：过滤条件在JOIN结果集的WHERE子句中。

        ```
        SELECT A.*
        FROM A LEFT SEMI JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key
        WHERE A.ds='20180101';
        ```

        符合关联条件的结果如下。

        |a.key|a.ds|
        |:----|:---|
        |1|20180101|

        对上述结果执行JOIN结果集中的过滤条件`A.ds='20180101'`，结果如下。

        |a.key|a.ds|
        |-----|----|
        |1|20180101|

-   LEFT ANTI JOIN

    LEFT ANTI JOIN将左表的每一条记录，和右表进行匹配。如果右表中的记录不匹配，则输出左表。由于只输出左表，所以JOIN后的WHERE条件中不能涉及右表。LEFT ANTI JOIN常常用来实现NOT EXISTS语义。

    结论：过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`中时，查询结果不一致。

    -   左表的过滤条件在`{subquery_where_condition}`和`{where_condition}`中时，查询结果是一致的。
    -   右表的过滤条件在`{subquery_where_condition}`和`{on_condition}`中时，查询结果是一致的。
    -   情况1：过滤条件在子查询`{subquery_where_condition}`中。

        ```
        SELECT A.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        LEFT ANTI JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        结果如下。

        |a.key|a.ds|
        |:----|:---|
        |2|20180101|

    -   情况2：过滤条件在JOIN的关联条件`{on_condition}`中。

        ```
        SELECT A.*
        FROM A LEFT ANTI JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        结果如下。

        |a.key|a.ds|
        |:----|:---|
        |2|20180101|
        |2|20180102|

    -   情况3：过滤条件在JOIN结果集的WHERE子句中。

        ```
        SELECT A.*
        FROM A LEFT ANTI JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key
        WHERE A.ds='20180101';
        ```

        左表中符合关联条件的数据如下。

        |a.key|a.ds|
        |:----|:---|
        |2|20180101|
        |2|20180102|

        对上述结果执行JOIN结果集中的过滤条件`A.ds='20180101'`，结果如下。

        |a.key|a.ds|
        |:----|:---|
        |2|20180101|


## 注意事项

-   INNER JOIN/LEFT SEMI JOIN左右表的过滤条件不受限制。
-   LEFT JOIN/LEFT ANTI JOIN左表的过滤条件需放在`{subquery_where_condition}`或`{where_condition}`中，右表的过滤条件需放在`{subquery_where_condition}`或`{on_condition}`中。
-   RIGHT JOIN和LEFT JOIN相反，右表的过滤条件需放在`{subquery_where_condition}`或`{where_condition}`中，左表的过滤条件需放在`{subquery_where_condition}`或`{on_condition}`。
-   FULL OUTER JOIN的过滤条件只能放在`{subquery_where_condition}`中。

