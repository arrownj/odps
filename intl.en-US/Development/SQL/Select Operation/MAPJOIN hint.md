---
keyword: MAPJOIN
---

# MAPJOIN hint

This topic describes how to explicitly specify MAPJOIN in a SELECT statement to join a large table with one or more small tables. MAPJOIN speeds up your query.

## Background information

A JOIN operation contains three stages: map, shuffle, and reduce. In most cases, tables are joined in the reduce stage.

MAPJOIN joins tables in the map stage instead of the reduce stage. This reduces data transmission time and system resource consumption and optimizes jobs.

In the map stage, MAPJOIN loads all data in the specified tables into the memory of the program that executes the JOIN operation. The tables specified for MAPJOIN must be small tables, and the total memory occupied by the table data cannot exceed 512 MB.

## Use method

To use MAPJOIN, you must specify the hint `/* + MAPJOIN(table) */` in a SELECT statement.

For example, sale\_detail is a large table whose alias is b, and shop is a small table whose alias is a.

-   The following statement is used to perform a common JOIN operation:

    ```
    SELECT  a.shop_name,
            b.customer_id,
            b.total_price
    FROM shop a JOIN sale_detail b
    ON a.shop_name = b.shop_name;
    ```

-   The following statement is used to perform a JOIN operation with MAPJOIN specified:

    ```
    SELECT /* + MAPJOIN(a) */
            a.shop_name,
            b.customer_id,
            b.total_price
    FROM shop a JOIN sale_detail b
    ON a.shop_name = b.shop_name;
    ```


## Usage notes

-   When you reference a small table or a subquery, you must reference the alias of the table or subquery.
-   MAPJOIN supports small tables in subqueries.
-   The left table in a LEFT OUTER JOIN operation must be a large table.
-   The right table in a RIGHT OUTER JOIN operation must be a large table.
-   Either the left or right table in an INNER JOIN operation can be a large table.
-   MAPJOIN cannot be used in a FULL OUTER JOIN operation.
-   For MAPJOIN, you can use non-equi joins or combine conditions by using OR. You can calculate the Cartesian product by leaving the ON condition unspecified or by using `MAPJOIN ON 1 = 1`, such as `SELECT /* + MAPJOIN(a) */ a.id FROM shop a JOIN table_name b ON 1=1`. However, this calculation method may increase the data volume.
-   MaxCompute allows you to specify a maximum of 128 small tables for MAPJOIN. If you specify more than 128 small tables, a syntax error is returned. Separate small tables with commas \(,\), such as `/* +MAPJOIN(a,b,c) */`.
-   The total memory occupied by small tables cannot exceed 512 MB. MaxCompute compresses your data before storage. As a result, the data volume of small tables sharply increases after they are loaded into the memory. 512 MB indicates the maximum data volume after small tables are loaded into the memory.

## Example

In a MaxCompute SQL statement, you cannot use non-equi joins or the OR logical operator in the ON condition for a common JOIN operation. However, you can use them for a JOIN operation with MAPJOIN specified.

```
SELECT /* + MAPJOIN(a) */
        a.total_price,
        b.total_price
FROM shop a JOIN sale_detail b
ON a.total_price < b.total_price OR a.total_price + b.total_price < 500;
```

