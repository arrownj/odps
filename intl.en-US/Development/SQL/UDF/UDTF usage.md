---
keyword: [Java UDTF, Python UDTF]
---

# UDTF usage

This topic describes the usage and limits of Java user-defined table functions \(UDTFs\) and Python UDTFs.

## Usage

The following statements provide examples to show typical use cases of UDTFs in SQL:

```
select user_udtf(col0, col1, col2) as (c0, c1) from my_table; 
select user_udtf(col0, col1, col2) as (c0, c1) from (select * from my_table distribute by key sort by key) t;
select reduce_udtf(col0, col1, col2) as (c0, c1) from (select col0, col1, col2 from (select map_udtf(a0, a1, a2, a3) as (col0, col1, col2) from my_table) t1 distribute by col0 sort by col0, col1) t2;
```

## Limits

When you use UDTFs, note the following limits:

-   A `SELECT` clause cannot contain other expressions.

    ```
    select value, user_udtf(key) as mycol ...
    ```

-   A UDTF cannot be nested.

    ```
    select user_udtf1(user_udtf2(key)) as mycol...
    ```

-   A `SELECT` clause cannot be used with a `GROUP BY`, `DISTRIBUTE BY`, or `SORT BY` clause.

    ```
    select user_udtf(key) as mycol ... group by mycol;
    ```


