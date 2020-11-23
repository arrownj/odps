---
keyword: [Java UDTF, Python UDTF]
---

# UDTF使用说明

本文为您介绍Java UDTF和Python UDTF的使用方式和使用限制。

## 使用方式

UDTF在SQL中的使用方式如下。

```
select user_udtf(col0, col1, col2) as (c0, c1) from my_table; 
select user_udtf(col0, col1, col2) as (c0, c1) from (select * from my_table distribute by key sort by key) t;
select reduce_udtf(col0, col1, col2) as (c0, c1) from (select col0, col1, col2 from (select map_udtf(a0, a1, a2, a3) as (col0, col1, col2) from my_table) t1 distribute by col0 sort by col0, col1) t2;
```

UDTF的详细使用说明请参见[Java UDTF](/intl.zh-CN/开发/SQL及函数/UDF/Java UDF.md)和[Python UDTF](/intl.zh-CN/开发/SQL及函数/UDF/Python 2 UDF.md)。

## 使用限制

UDTF的使用限制如下：

-   同一个`select`子句中不允许有其他表达式。

    ```
    select value, user_udtf(key) as mycol ...
    ```

-   UDTF不能嵌套使用。

    ```
    select user_udtf1(user_udtf2(key)) as mycol...
    ```

-   不支持在同一个`select`子句中与`group by`、`distribute by`、`sort by`联用。

    ```
    select user_udtf(key) as mycol ... group by mycol;
    ```


