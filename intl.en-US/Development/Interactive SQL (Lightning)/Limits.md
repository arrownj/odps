---
keyword: [MaxCompute Lightning, limit]
---

# Limits

This topic describes the limits on the use of MaxCompute Lightning.

## Limits on DDL and DML statements

MaxCompute Lightning now supports only the SELECT statement on MaxCompute tables. It does not support the UPDATE, CREATE, DELETE, or INSERT statements on MaxCompute tables. These statements will be supported in later versions.

## Limits on queries

-   MaxCompute Lightning allows you to scan a maximum of 1,024 partitions from a partitioned table.
-   MaxCompute Lightning does not allow you to create or use views.
-   MaxCompute Lightning does not support the following data types: MAP, ARRAY, TINYINT, BINARY, TIMESTAMP, or DECIMAL with specified precision.
-   MaxCompute Lightning allows you to scan a maximum of 1 TB data from a table in a single query.
-   The size of a query statement that you commit cannot exceed 100 KB.
-   The lifecycle of a query is one hour.
-   A single query cannot access more than 100 GB of data.
-   The total number of the JOIN and GROUP BY keywords in a single query cannot be greater than 5.
-   The number of data records in the result of a single query cannot exceed 10,000. If the number exceeds this limit, the query result is truncated.
-   MaxCompute Lightning is compatible with PostgreSQL. However, PostgreSQL does not support the DATETIME data type. If your data is of the DATETIME type, it is mapped to the TIMESTAMP type in PostgreSQL for queries.

## Limits on UDFs

-   You cannot use MaxCompute UDFs in MaxCompute Lightning.
-   You cannot create or use PostgreSQL UDFs in MaxCompute Lightning. However, you can use the built-in functions of [PostgreSQL](https://www.postgresql.org/docs/8.2/static/functions.html).
-   You cannot use the built-in functions of MaxCompute.

## Limits on query concurrency

In a MaxCompute project, the number of concurrent MaxCompute Lightning queries cannot exceed 20.

