---
keyword: JOIN
---

# JOIN operations in MaxCompute SQL

This topic describes the JOIN operations that MaxCompute SQL supports.

## Overview

The following table describes the JOIN operations that MaxCompute SQL supports.

|Operation|Description|
|:--------|:----------|
|INNER JOIN|Returns the rows that have matching column values in both the left table and the right table based on the join condition.|
|LEFT JOIN|Returns all the rows from the left table and matched rows from the right table based on the join condition. If a row in the left table has no matching rows in the right table, NULL values are returned in the columns from the right table in the result set.|
|RIGHT JOIN|Returns all the rows from the right table and matched rows from the left table based on the join condition. If a row in the right table has no matching rows in the left table, NULL values are returned in the columns from the left table in the result set.|
|FULL JOIN|Returns all the rows in both the left table and the right table whether the join condition is met or not. In the result set, NULL values are returned in the columns from the table that lacks a matching row in the other table.|
|LEFT SEMI JOIN|Returns only the rows in the left table that have a matching row in the right table.|
|LEFT ANTI JOIN|Returns only the rows in the left table that have no matching rows in the right table.|

The ON clause and the WHERE clause can be used in the same SQL statement. For example, consider the following SQL statement:

```
(SELECT * FROM A WHERE {subquery_where_condition} A) A
JOIN
(SELECT * FROM B WHERE {subquery_where_condition} B) B
ON {on_condition}
WHERE {where_condition}
```

The conditions in the preceding SQL statement are evaluated in the following order:

1.  The `{subquery_where_condition}` condition in the WHERE clause of the subqueries
2.  The `{on_condition}` condition in the ON clause
3.  The `{where_condition}` condition in the WHERE clause after the JOIN clause

Therefore, a JOIN operation may return different results, depending on whether the filter conditions are specified in `{subquery_where_condition}`, `{on_condition}`, or `{where_condition}`. For more information, see [Case-by-case study](#section_omw_m9j_d6i).

## Test tables

-   Table A

    Execute the following statement to create Table A:

    ```
    CREATE TABLE A AS SELECT * FROM VALUES (1, 20180101),(2, 20180101),(2, 20180102) t (key, ds);
    ```

    Table A has the following three rows and is used as the left table for all JOIN operations in this topic.

    |key|ds|
    |:--|:-|
    |1|20180101|
    |2|20180101|
    |2|20180102|

-   Table B

    Execute the following statement to create Table B:

    ```
    CREATE TABLE B AS SELECT * FROM VALUES (1, 20180101),(3, 20180101),(2, 20180102) t (key, ds);
    ```

    Table B has the following three rows and is used as the right table for all JOIN operations in this topic.

    |key|ds|
    |:--|:-|
    |1|20180101|
    |3|20180101|
    |2|20180102|

-   Cartesian product of Table A and Table B

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


## Case-by-case study

-   INNER JOIN

    An INNER JOIN operation first takes the Cartesian product of the rows in Table A and Table B and returns rows that have matching column values in Table A and Table B based on the ON clause.

    Conclusion: An INNER JOIN operation returns the same results independently of whether the filter conditions are specified in `{subquery_where_condition}`, `{on_condition}`, or `{where_condition}`.

    -   Case 1: Specify the filter conditions in the `{subquery_where_condition}` clause, as shown in the following statement:

        ```
        SELECT A.*, B.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        The following table lists the results that the preceding statement returns.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|

    -   Case 2: Specify the filter conditions in the `{on_condition}` clause, as shown in the following statement:

        ```
        SELECT A.*, B.*
        FROM A JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        The Cartesian product of Table A and Table B contains nine rows, of which only one meets the join condition. The following table lists the results that the preceding statement returns.

        |a.key|a.ds|b.key|b.ds|
        |1|20180101|1|20180101|

    -   Case 3: Specify the filter conditions in the WHERE clause after the ON clause, as shown in the following statement:

        ```
        SELECT A.*, B.*
        FROM A JOIN B
        ON a.key = b.key
        WHERE A.ds='20180101' and B.ds='20180101';
        ```

        The Cartesian product of Table A and Table B contains nine rows, of which only three meet the join condition. The following table lists the result set.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180102|2|20180102|
        |2|20180101|2|20180102|

        The query processor then filters the preceding result set based on the `A.ds='20180101' and B.ds='20180101'` filter condition. The following table lists the results that the preceding statement returns.

        |a.key|a.ds|b.key|b.ds|
        |-----|----|-----|----|
        |1|20180101|1|20180101|

-   LEFT JOIN

    A LEFT JOIN operation first takes the Cartesian product of the rows in Table A and Table B and returns all the rows of Table A and rows in Table B that meet the join condition. If the join condition finds no matching rows in Table B for a row in Table A, the row in Table A is returned in the result set with NULL values in each column from Table B.

    Conclusion: A LEFT JOIN operation may return different results, depending on whether the filter conditions are specified in `{subquery_where_condition}`, `{on_condition}`, or `{where_condition}`:

    -   The operation returns the same results, regardless of whether the filter condition for Table A is specified in `{subquery_where_condition}` or `{where_condition}`.
    -   The operation returns the same results, regardless of whether the filter condition for Table B is specified in `{subquery_where_condition}` or `{on_condition}`.
    -   Case 1: Specify the filter conditions in the `{subquery_where_condition}` clause, as shown in the following statement:

        ```
        SELECT A.*, B.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        LEFT JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        The following table lists the results that the preceding statement returns.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|NULL|NULL|

    -   Case 2: Specify the filter conditions in the `{on_condition}` clause, as shown in the following statement:

        ```
        SELECT A.*, B.*
        FROM A JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        The Cartesian product of Table A and Table B contains nine rows, of which only one meets the join condition. The other two rows in Table A do not have matching rows in Table B. Therefore, NULL values are returned in the columns from Table B for the two rows in Table A. The following table lists the results that the preceding statement returns.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|NULL|NULL|
        |2|20180102|NULL|NULL|

    -   Case 3: Specify the filter conditions in the WHERE clause after the ON clause, as shown in the following statement:

        ```
        SELECT A.*, B.*
        FROM A JOIN B
        ON a.key = b.key
        WHERE A.ds='20180101' and B.ds='20180101';
        ```

        The Cartesian product of Table A and Table B contains nine rows, of which only three meet the join condition. The following table lists the result set.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|2|20180102|
        |2|20180102|2|20180102|

        The query processor then filters the preceding result set based on the `A.ds='20180101' and B.ds='20180101'` filter condition. The following table lists the results that the preceding statement returns.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|

-   RIGHT JOIN

    A RIGHT JOIN operation is similar to a LEFT JOIN operation, except that the two tables are used in a reversed manner. A RIGHT JOIN operation returns all the rows of Table B and rows in Table A that meet the join condition.

    -   Conclusion: A RIGHT JOIN operation may return different results, depending on whether the filter conditions are specified in `{subquery_where_condition}`, `{on_condition}`, or `{where_condition}`.
    -   The operation returns the same results, regardless of whether the filter condition for Table B is specified in `{subquery_where_condition}` or `{where_condition}`.
    -   The operation returns the same results, regardless of whether the filter condition for Table A is specified in `{subquery_where_condition}` or `{on_condition}`.
-   FULL JOIN

    A FULL JOIN operation takes the Cartesian product of the rows in Table A and Table B and returns all the rows in Table A and Table B, whether the join condition is met or not. In the result set, NULL values are returned in the columns from the table that lacks a matching row in the other table.

    Conclusion: A FULL JOIN operation may return different results, depending on whether the filter conditions are specified in `{subquery_where_condition}`, `{on_condition}`, or `{where_condition}`.

    -   Case 1: Specify the filter conditions in the `{subquery_where_condition}` clause, as shown in the following statement:

        ```
        SELECT A.*, B.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        FULL JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        The following table lists the results that the preceding statement returns.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|NULL|NULL|
        |NULL|NULL|3|20180101|

    -   Case 2: Specify the filter conditions in the `{on_condition}` clause, as shown in the following statement:

        ```
        SELECT A.*, B.*
        FROM A FULL JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        The Cartesian product of Table A and Table B contains nine rows, of which only one meets the join condition. In the result set, for the two rows in Table A that match no rows in Table B, NULL values are returned in the columns from Table B. For the two rows in Table B that match no rows in Table A, NULL values are returned in the columns from Table A. The following table lists the results that the preceding statement returns.

        |a.key|a.ds|b.key|b.ds|
        |-----|----|-----|----|
        |1|20180101|1|20180101|
        |2|20180101|NULL|NULL|
        |2|20180102|NULL|NULL|
        |NULL|NULL|3|20180101|
        |NULL|NULL|2|20180102|

    -   Case 3: Specify the filter conditions in the WHERE clause after the ON clause, as shown in the following statement:

        ```
        SELECT A.*, B.*
        FROM A FULL JOIN B
        ON a.key = b.key
        WHERE A.ds='20180101' and B.ds='20180101';
        ```

        The Cartesian product of Table A and Table B contains nine rows, of which only three meet the join condition.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|2|20180102|
        |2|20180102|2|20180102|

        The row in Table B that has no matching rows in Table A is returned in the result set, with NULL values in the columns from Table A for that row. The following table lists the result set.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|
        |2|20180101|2|20180102|
        |2|20180102|2|20180102|
        |NULL|NULL|3|20180101|

        The query processor then filters the preceding result set based on the `A.ds='20180101' and B.ds='20180101'` filter condition. The following table lists the results that the preceding statement returns.

        |a.key|a.ds|b.key|b.ds|
        |:----|:---|:----|:---|
        |1|20180101|1|20180101|

-   LEFT SEMI JOIN

    A LEFT SEMI JOIN operation returns only the rows in Table A that have a matching row in Table B. A LEFT SEMI JOIN operation does not return rows from Table B. Therefore, you cannot specify a filter condition for Table B in the WHERE clause after the ON clause.

    Conclusion: A LEFT SEMI JOIN operation returns the same results independently of whether the filter conditions are specified in `{subquery_where_condition}`, `{on_condition}`, or `{where_condition}`.

    -   Case 1: Specify the filter conditions in the \{subquery\_where\_condition\} clause, as shown in the following statement:

        ```
        SELECT A.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        LEFT SEMI JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        The following table lists the results that the preceding statement returns.

        |a.key|a.ds|
        |:----|:---|
        |1|20180101|

    -   Case 2: Specify the filter conditions in the `{on_condition}` clause, as shown in the following statement:

        ```
        SELECT A.*
        FROM A LEFT SEMI JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        The following table lists the results that the preceding statement returns.

        |a.key|a.ds|
        |:----|:---|
        |1|20180101|

    -   Case 3: Specify the filter conditions in the WHERE clause after the ON clause, as shown in the following statement:

        ```
        SELECT A.*
        FROM A LEFT SEMI JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key
        WHERE A.ds='20180101';
        ```

        The following table lists the result set.

        |a.key|a.ds|
        |:----|:---|
        |1|20180101|

        The query processor then filters the preceding result set based on the `A.ds='20180101'` filter condition. The following table lists the results that the preceding statement returns.

        |a.key|a.ds|
        |-----|----|
        |1|20180101|

-   LEFT ANTI JOIN

    A LEFT ANTI JOIN operation returns only the rows in Table A that have no matching rows in Table B. A LEFT ANTI JOIN operation does not return rows from Table B. Therefore, you cannot specify a filter condition for Table B in the WHERE clause after the ON clause. A LEFT ANTI JOIN operation is usually used to replace the NOT EXISTS syntax.

    Conclusion: A LEFT ANTI JOIN operation may return different results, depending on whether the filter conditions are specified in `{subquery_where_condition}`, `{on_condition}`, or `{where_condition}`.

    -   The operation returns the same results, regardless of whether the filter condition for Table A is specified in `{subquery_where_condition}` or `{where_condition}`.
    -   The operation returns the same results, regardless of whether the filter condition for Table B is specified in `{subquery_where_condition}` or `{on_condition}`.
    -   Case 1: Specify the filter conditions in the `{subquery_where_condition}` clause, as shown in the following statement:

        ```
        SELECT A.*
        FROM
        (SELECT * FROM A WHERE ds='20180101') A
        LEFT ANTI JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key;
        ```

        The following table lists the results that the preceding statement returns.

        |a.key|a.ds|
        |:----|:---|
        |2|20180101|

    -   Case 2: Specify the filter conditions in the `{on_condition}` clause, as shown in the following statement:

        ```
        SELECT A.*
        FROM A LEFT ANTI JOIN B
        ON a.key = b.key and A.ds='20180101' and B.ds='20180101';
        ```

        The following table lists the results that the preceding statement returns.

        |a.key|a.ds|
        |:----|:---|
        |2|20180101|
        |2|20180102|

    -   Case 3: Specify the filter conditions in the WHERE clause after the ON clause, as shown in the following statement:

        ```
        SELECT A.*
        FROM A LEFT ANTI JOIN
        (SELECT * FROM B WHERE ds='20180101') B
        ON a.key = b.key
        WHERE A.ds='20180101';
        ```

        The following table lists the result set.

        |a.key|a.ds|
        |:----|:---|
        |2|20180101|
        |2|20180102|

        The query processor then filters the preceding result set based on the `A.ds='20180101'` filter condition. The following table lists the results that the preceding statement returns.

        |a.key|a.ds|
        |:----|:---|
        |2|20180101|


## Usage notes

-   For an INNER JOIN operation or a LEFT SEMI JOIN operation, an SQL statement returns the same results, regardless of where you specify filter conditions for the left table and the right table.
-   For a LEFT JOIN operation or a LEFT ANTI JOIN operation, the filter condition for the left table functions the same whether it is specified in `{subquery_where_condition}` or `{where_condition}`. The filter condition for the right table functions the same whether it is specified in `{subquery_where_condition}` or `{on_condition}`.
-   For a RIGHT JOIN operation, the filter condition for the left table functions the same whether it is specified in `{subquery_where_condition}` or `{on_condition}`. The filter condition for the right table functions the same whether it is specified in `{subquery_where_condition}` or `{where_condition}`.
-   For a FULL OUTER JOIN operation, filter conditions can be specified only in `{subquery_where_condition}`.

