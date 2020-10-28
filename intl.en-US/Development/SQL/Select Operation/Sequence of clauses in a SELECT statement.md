---
keyword: sequence for executing SELECT statements
---

# Sequence of clauses in a SELECT statement

Clauses in a SELECT statement that is written in compliance with the SELECT syntax of MaxCompute are executed in a different logic sequence from the clauses in a standard SELECT statement.

## Example 1

```
SELECT  key
        ,MAX(value)
FROM    src t
WHERE   value > 0
GROUP BY key
HAVING  SUM(value) > 100
ORDER BY key
LIMIT   100
;
```

The clauses in the preceding example are executed in the following sequence: `FROM->WHERE->GROUY BY->HAVING->SELECT->ORDER BY->LIMIT`.

-   The `ORDER BY` clause can only be executed to sort columns that are returned by the `SELECT` clause rather than the columns in the source table that are returned by the `FROM` clause.
-   The `HAVING` clause can be used together with the `GROUP BY key` clause and aggregate functions.
-   If a statement contains the `GROUP BY key` clause, the `SELECT` clause is executed to extract data from the columns that are returned by the `GROUP BY key` clause and aggregate functions instead of the columns in the source table that are returned by the `FROM` clause.

To avoid confusion, MaxCompute allows you to write query clauses based on the execution sequence.

```
FROM    src t
WHERE   value > 0
GROUP BY key
HAVING  SUM(value) > 100
SELECT  key
        ,MAX(value)
ORDER BY key
LIMIT   100
;
```

## Example 2

```
SELECT  shop_name
        ,total_price
        ,region
FROM    sale_detail
WHERE   total_price > 150
DISTRIBUTE BY region
SORT BY region
;
```

The clauses in the preceding example are executed in the following sequence: `FROM->WHERE->SELECT->DISTRIBUTE BY->SORT BY`.

