---
keyword: window functions
---

# Window functions

You can use window functions to flexibly analyze and process jobs in MaxCompute SQL.

**Note:**

-   Window functions can only be included in the `SELECT` clause.
-   A window function cannot contain nested window functions or aggregate functions.
-   Window functions cannot be used with aggregate functions of the same level.
-   A maximum of five window functions can be used in a MaxCompute SQL statement.

## Syntax

```
window_func() over (partition by [col1,col2…]
[order by [col1[asc|desc], col2[asc|desc]…]] windowing_clause)
```

-   `PARTITION BY`: partitioning columns.
-   Rows with the same partitioning column value are considered in the same window. A window can contain a maximum of 100 million rows of data. We recommend that the number of rows in a window not exceed 5 million. If the number of rows exceeds 5 million, an error is returned. This limit applies to all built-in window functions.

    **Note:** `GROUP BY` UID indicates that records with the same UID are considered in the same window. Therefore, the records with the same UID can contain a maximum of 100 million rows of data. We recommend that the number of rows in a window not exceed 5 million.

-   `ORDER BY`: the rule used to sort data in a window.

    **Note:** If some values in `ORDER BY` are the same, the sorting result may not be accurate. To reduce randomness, keep each value in `ORDER BY` unique.

-   You can use the following `ROWS` functions in `windowing_clause` to specify the partitioning method:

    -   `rows between x preceding|following and y preceding|following`: indicates a window range from the xth row preceding or following the current row to the yth row preceding or following the current row.
    -   `rows x preceding|following`: indicates a window range from the xth row preceding or following the current row to the current row.
    **Note:**

    -   x and y must be integer constants greater than or equal to 0. Their values range from 0 to 10000. 0 indicates the current row. Before you use `ROWS`, you must specify `ORDER BY` to specify a window range.
    -   Only the following window functions can use `ROWS` to specify the partitioning method: AVG, COUNT, MAX, MIN, STDDEV, and SUM.

## COUNT

This section describes the syntax, description, and example of the COUNT function.

-   Syntax

    ```
    Bigint count([distinct] expr) over(partition by [col1, col2…]
    [order by [col1[asc|desc], col2[asc|desc]…]] [windowing_clause])
    ```

-   Description

    Calculates the number of rows that meet expr.

-   Parameters

    -   `expr`: a value of any data type. If the value for a row is NULL, the row is not included in the calculation. If the `distinct` keyword is specified, this parameter indicates that only distinct values are counted.
    -   `partition by [col1, col2…]`: partitioning columns.
    -   `order by col1 [asc|desc], col2[asc|desc]`: If `ORDER BY` is not specified, the number of rows that meet `expr` in the current window is returned. If `ORDER BY` is specified, the returned results are sorted in the specified order and the number of rows from the starting row to the current row in the current window is returned.
    **Note:** `ORDER BY` cannot be used if the `distinct` keyword is specified.

-   Return value

    Returns a value of the BIGINT type.

-   Example

    The `test_src` table contains the `user_id` column of the BIGINT type.

    ```
    select user_id,count(user_id) over (partition by user_id) as count
    from test_src;
    +---------+------------+
    | user_id |  count     |
    +---------+------------+
    | 1       | 3          |
    | 1       | 3          |
    | 1       | 3          |
    | 2       | 1          |
    | 3       | 1          |
    +---------+------------+    
    -- If ORDER BY is not specified, the number of rows of the user_id column from the current window is returned.
    select user_id,count(user_id) over (partition by user_id order by user_id) as count
    from test_src;
    +---------+------------+
    | user_id | count      |
    +---------+------------+
    | 1       | 1          |      -- This row is the starting row of this window.
    | 1       | 2          |      -- Two records exist from the starting row to the current row. The number 2 is returned.
    | 1       | 3          |
    | 2       | 1          |
    | 3       | 1          |
    +---------+------------+
    -- If ORDER BY is specified, the number of rows from the starting row to the current row in the current window is returned.
    ```


If duplicate values are specified for `ORDER BY`, the processing method is based on the compatibility between MaxCompute and Hive.

-   If MaxCompute is not compatible with Hive, the returned values are COUNT.

    ```
    set odps.sql.hive.compatible=false;
    select user_id, price, count(price) over
      (partition by user_id order by price) as count from test_src;
    
    +------------+------------+------------+
    | user_id    | price      | count      |
    +------------+------------+------------+
    | 1          | 4.5        | 1          |  -- This row is the starting row of this window.
    | 1          | 5.5        | 2          |  -- The value of COUNT for the second row is 2.
    | 1          | 5.5        | 3          |  -- The value of COUNT for the third row is 3.
    | 1          | 6.5        | 4          |
    | 2          | NULL       | 0          |
    | 2          | 3.0        | 1          |
    | 3          | NULL       | 0          |
    | 3          | 4.0        | 1          |
    +------------+------------+------------+
    ```

-   If MaxCompute is compatible with Hive, the return value is the value of COUNT for the last row of the rows with same values.

    ```
    set odps.sql.hive.compatible=true;
    select user_id, price, count(price) over
      (partition by user_id order by price) as count from test_src;
    
    +------------+------------+------------+
    | user_id    | price      | count      |
    +------------+------------+------------+
    | 1          | 4.5        | 1          | -- This row is the starting row of this window.
    | 1          | 5.5        | 3          | -- The value of COUNT for the second row is the value of COUNT for the third row because the prices of the two rows are the same.
    | 1          | 5.5        | 3          | -- The value of COUNT for the third row is 3.
    | 1          | 6.5        | 4          |
    | 2          | NULL       | 0          |
    | 2          | 3.0        | 1          |
    | 3          | NULL       | 0          |
    | 3          | 4.0        | 1          |
    +------------+------------+------------+
    ```


## AVG

This section describes the syntax, description, and example of the AVG function.

-   Syntax

    ```
    avg([distinct] expr) over(partition by [col1, col2…]
    [order by [col1[asc|desc], col2[asc|desc]…]] [windowing_clause])
    ```

-   Description

    Calculates the average value of input values.

-   Parameters

    -   `distinct`: If the `distinct` keyword is specified, this parameter indicates that the average value of distinct values is calculated.
    -   `expr`: a value of the DOUBLE or DECIMAL type.
        -   If the input value is of the STRING or BIGINT type, it is implicitly converted into the DOUBLE type before calculation. If it is of another data type, an error is returned.
        -   If the value for a row is NULL, the row is not included in the calculation.
        -   The values of the BOOLEAN type are not included in the calculation.
    -   `partition by [col1, col2...]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: If `ORDER BY` is not specified, the average value of all values in the current window is returned. If `ORDER BY` is specified, the returned results are sorted in the specified order and the average value from the starting row to the current row in the current window is returned.
    **Note:** `ORDER BY` cannot be used if the `distinct` keyword is specified.

-   Return value

    Returns a value of the DOUBLE type.


If duplicate values are specified for `ORDER BY`, the processing method is based on the compatibility between MaxCompute and Hive.

-   If MaxCompute is not compatible with Hive, the return value is the average value for each row.

    ```
    set odps.sql.hive.compatible=false;
    select user_id, price, avg(price) over
      (partition by user_id order by price) from test_src;
    
    +------------+------------+-------------------+
    | user_id    | price      | _c2               |
    +------------+------------+-------------------+
    | 1          | 4.5        | 4.5               |    -- This row is the starting row of this window.
    | 1          | 5.5        | 5.0               |    -- The return value is the average value of the values for the first and second rows.
    | 1          | 5.5        | 5.166666666666667 |    -- The return value is the average value of the values from the first row to the third row.
    | 1          | 6.5        | 5.5               |
    | 2          | NULL       | NULL              |
    | 2          | 3.0        | 3.0               |
    | 3          | NULL       | NULL              |
    | 3          | 4.0        | 4.0               |
    +------------+------------+-------------------+
    ```

-   If MaxCompute is compatible with Hive, the return value is the average value of values for the last row of the rows with same values.

    ```
    set odps.sql.hive.compatible=true;
    select user_id, price, avg(price) over
      (partition by user_id order by price) from test_src;
    
    +------------+------------+-------------------+
    | user_id    | price      | _c2               |
    +------------+------------+-------------------+
    | 1          | 4.5        | 4.5               |  -- This row is the starting row of this window.
    | 1          | 5.5        | 5.166666666666667 |  -- The return value is the average value of values from the first row to the third row.
    | 1          | 5.5        | 5.166666666666667 |  -- The return value is the average value of values from the first row to the third row.
    | 1          | 6.5        | 5.5               |
    | 2          | NULL       | NULL              |
    | 2          | 3.0        | 3.0               |
    | 3          | NULL       | NULL              |
    | 3          | 4.0        | 4.0               |
    +------------+------------+-------------------+
    ```


## MAX

-   Syntax

    ```
    max([distinct] expr) over(partition by [col1, col2…]
    [order by [col1[asc|desc], col2[asc|desc]…]] [windowing_clause])
    ```

-   Description

    Calculates the maximum value of input values.

-   Parameters

    -   `expr`: a value of any data type except BOOLEAN. If the value for a row is NULL, the row is not included in the calculation. If the `distinct` keyword is specified, this parameter indicates that the maximum value of distinct values is calculated. The calculation result is not affected regardless of whether the parameter is specified.
    -   `partition by [col1, col2…]`: partitioning columns.
    -   `order by [col1[asc|desc], col2[asc|desc`: If `ORDER BY` is not specified, the maximum value of all values in the current window is returned. If `ORDER BY` is specified, the returned results are sorted in the specified order and the maximum value of values from the starting row to the current row in the current window is returned.
    **Note:** `ORDER BY` cannot be used if the `distinct` keyword is specified.

-   Return value

    Returns values of the same type as `expr`.


## MIN

-   Syntax

    ```
    min([distinct] expr) over(partition by [col1, col2…]
    [order by [col1[asc|desc], col2[asc|desc]…]] [windowing_clause])
    ```

-   Description

    Calculates the minimum value of input values.

-   Parameters

    -   `expr`: a value of any data type except BOOLEAN. If the value for a row is NULL, the row is not included in the calculation. If the `distinct` keyword is specified, this parameter indicates that the minimum value of distinct values is calculated. The calculation result is not affected regardless of whether the parameter is specified.
    -   `partition by [col1, col2…]`: partitioning columns.
    -   `order by [col1[asc|desc], col2[asc|desc`: If `ORDER BY` is not specified, the minimum value in the current window is returned. If `ORDER BY` is specified, the returned results are sorted in the specified order and the minimum value of values from the starting row to the current row in the current window is returned.
    **Note:** `ORDER BY` cannot be used if the `distinct` keyword is specified.

-   Return value

    Returns values of the same type as `expr`.


## MEDIAN

-   Syntax

    ```
    Double median(Double number1,number2...) over(partition by [col1, col2…])
    Decimal median(Decimal number1,number2...) over(partition by [col1,col2…])
    ```

-   Description

    Calculates the minimum value of median values.

-   Parameters
    -   number1,number1…: numbers of the DOUBLE or DECIMAL type. You can enter 1 to 255 numbers.
        -   If the input value is of the DOUBLE type, it is converted into an array of the DOUBLE type for calculation by default.
        -   If the input value is of the STRING or BIGINT type, it is implicitly converted into the DOUBLE type before calculation. If it is of another data type, an error is returned.
        -   If the input value is NULL, NULL is returned.
    -   `partition by [col1, col2…]`: partitioning columns.
-   Return value

    Returns a value of the DOUBLE or DECIMAL type.


## STDDEV

This section describes the syntax, description, and example of the STDDEV function.

-   Syntax

    ```
    Double stddev([distinct] expr) over(partition by [col1, col2…]
    [order by [col1[asc|desc], col2[asc|desc]…]] [windowing_clause])
    Decimal stddev([distinct] expr) over(partition by [col1, col2…] 
    [order by [col1[asc|desc], col2[asc|desc]…]] [windowing_clause])
    ```

-   Description

    Calculates the population standard deviation.

-   Parameters
    -   `expr`: a value of the DOUBLE or DECIMAL type.

        -   If the input value is of the STRING or BIGINT type, it is implicitly converted into the DOUBLE type before calculation. If it is of another data type, an error is returned.
        -   If the value for a row is NULL, the row is not included in the calculation.
        -   If the `distinct` keyword is specified, this parameter indicates that the population standard deviation of distinct values is calculated.
        **Note:**

        -   `ORDER BY` cannot be used if the `distinct` keyword is specified.
        -   `stddev` has an alias function `stddev_pop`, which is used in the same way as `stddev`.
    -   `partition by [col1, col2..]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: If `ORDER BY` is not specified, the population standard deviation of the current window is returned. If `ORDER BY` is specified, the returned results are sorted in the specified order and the population standard deviation from the starting row to the current row in the current window is returned.
-   Return value

    If the input value is of the DECIMAL type, a value of the DECIMAL type is returned. Otherwise, a value of the DOUBLE type is returned.

-   Example

    ```
    select window, seq, stddev_pop('1\01') over (partition by window order by seq);
    ```


If duplicate values are specified for `ORDER BY`, the processing method is based on the compatibility between MaxCompute and Hive.

-   If MaxCompute is not compatible with Hive, the return value is the value of STDDEV for each row.

    ```
    set odps.sql.hive.compatible=false;
    select user_id, price, stddev(price) over
      (partition by user_id order by price) from test_src;
    
    +------------+------------+--------------------+
    | user_id    | price      | _c2                |
    +------------+------------+--------------------+
    | 1          | 4.5        | 0.0                |  -- This row is the starting row of this window.
    | 1          | 5.5        | 0.5                |  -- The return value is the value of STDDEV for the first and second rows.
    | 1          | 5.5        | 0.4714045207910316 |  -- The return value is the value of STDDEV from the first row to the third row.
    | 1          | 6.5        | 0.7071067811865475 |
    | 2          | NULL       | NULL               |
    | 2          | 3.0        | 0.0                |
    | 3          | NULL       | NULL               |
    | 3          | 4.0        | 0.0                |
    +------------+------------+--------------------+
    ```

-   If MaxCompute is compatible with Hive, the return value is the value of STDDEV for the last row of the rows with same values.

    ```
    set odps.sql.hive.compatible=true;
    select user_id, price, stddev(price) over
      (partition by user_id order by price) from test_src;
    
    +------------+------------+--------------------+
    | user_id    | price      | _c2                |
    +------------+------------+--------------------+
    | 1          | 4.5        | 0.0                | -- This row is the starting row of this window.
    | 1          | 5.5        | 0.4714045207910316 | -- The return value is the value of STDDEV from the first row to the third row.
    | 1          | 5.5        | 0.4714045207910316 | -- The return value is the value of STDDEV from the first row to the third row.
    | 1          | 6.5        | 0.7071067811865475 |
    | 2          | NULL       | NULL               |
    | 2          | 3.0        | 0.0                |
    | 3          | NULL       | NULL               |
    | 3          | 4.0        | 0.0                |
    +------------+------------+--------------------+
    ```


## STDDEV\_SAMP

-   Syntax

    ```
    Double stddev_samp([distinct] expr) over(partition by [col1, col2…]
    [order by [col1[asc|desc], col2[asc|desc]…]] [windowing_clause])
    Decimal stddev_samp([distinct] expr) over((partition by [col1,col2…] 
    [order by [col1[asc|desc], col2[asc|desc]…]] [windowing_clause])
    ```

-   Description

    Calculates the sample standard deviation.

-   Parameters
    -   `expr`: a value of the DOUBLE or DECIMAL type.

        -   If the input value is of the STRING or BIGINT type, it is implicitly converted into the DOUBLE type before calculation. If it is of another data type, an error is returned.
        -   If the value for a row is NULL, the row is not included in the calculation.
        -   If the `distinct` keyword is specified, this parameter indicates that the sample standard deviation of distinct values is calculated.
        **Note:** `ORDER BY` cannot be used if the `distinct` keyword is specified.

    -   `partition by [col1, col2..]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: If `ORDER BY` is not specified, the sample standard deviation in the current window is returned. If `ORDER BY` is specified, the returned results are sorted in the specified order and the sample standard deviation from the starting row to the current row in the current window is returned.
-   Return value

    If the input value is of the DECIMAL type, a value of the DECIMAL type is returned. Otherwise, a value of the DOUBLE type is returned.


## SUM

This section describes the syntax, description, and example of the SUM function.

-   Syntax

    ```
    sum([distinct] expr) over(partition by [col1, col2…]
    [order by [col1[asc|desc], col2[asc|desc]…]] [windowing_clause])
    ```

-   Description

    Calculates the sum of input values.

-   Parameters

    -   `expr`: a value of the DOUBLE, DECIMAL, or BIGINT type.
        -   If the input value is of the STRING type, it is implicitly converted into the DOUBLE type before calculation. If it is of another data type, an error is returned.
        -   If the value for a row is NULL, the row is not included in the calculation.
        -   If the `distinct` keyword is specified, this parameter indicates that the sum of distinct values is calculated.
    -   `partition by [col1, col2..]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: If `ORDER BY` is not specified, the sum of the `expr` value in the current window is returned. If `ORDER BY` is specified, the returned results are sorted in the specified order and the sum of values from the first row to the current row in the current window is returned.
    **Note:** `ORDER BY` cannot be used if the `distinct` keyword is specified.

-   Return value
    -   If the input value is of the BIGINT type, a value of the BIGINT type is returned.
    -   If the input value is of the DECIMAL type, a value of the DECIMAL type is returned.
    -   If the input value is of the DOUBLE or STRING type, a value of the DOUBLE type is returned.

If duplicate values are specified for `ORDER BY`, the processing method is based on the compatibility between MaxCompute and Hive.

-   If MaxCompute is not compatible with Hive, the return values are different. The return value is the sum of values for each row.

    ```
    set odps.sql.hive.compatible=false;
    select user_id, price, sum(price) over
      (partition by user_id order by price) from test_src;
    
    +------------+------------+------------+
    | user_id    | price      | _c2        |
    +------------+------------+------------+
    | 1          | 4.5        | 4.5        |  -- This row is the starting row of this window.
    | 1          | 5.5        | 10.0       |  -- The return value is the sum of values for the first and second rows.
    | 1          | 5.5        | 15.5       |  -- The return value is the sum of values from the first row to the third row.
    | 1          | 6.5        | 22.0       |
    | 2          | NULL       | NULL       |
    | 2          | 3.0        | 3.0        |
    | 3          | NULL       | NULL       |
    | 3          | 4.0        | 4.0        |
    +------------+------------+------------+
    ```

-   If MaxCompute is compatible with Hive, the return values are the same. The return value is the sum of values for the last row of the rows with same values.

    ```
    set odps.sql.hive.compatible=true;
    select user_id, price, sum(price) over
      (partition by user_id order by price) from test_src;
    +------------+------------+------------+
    | user_id    | price      | _c2        |
    +------------+------------+------------+
    | 1          | 4.5        | 4.5        |  -- This row is the starting row of this window.
    | 1          | 5.5        | 15.5       |  -- The return value is the sum of values from the first row to the third row.
    | 1          | 5.5        | 15.5       |  -- The return value is the sum of values from the first row to the third row.
    | 1          | 6.5        | 22.0       |
    | 2          | NULL       | NULL       |
    | 2          | 3.0        | 3.0        |
    | 3          | NULL       | NULL       |
    | 3          | 4.0        | 4.0        |
    +------------+------------+------------+
    ```


## DENSE\_RANK

-   Syntax

    ```
    Bigint dense_rank() over(partition by [col1, col2…]
    order by [col1[asc|desc], col2[asc|desc]…])
    ```

-   Description

    Calculates continuous rankings. The rankings for the data in rows with the same `col2` are the same.

-   Parameters
    -   `partition by [col1, col2..]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: the field based on which data is ranked.
-   Return value

    Returns a value of the BIGINT type.

-   Example

    Table `emp` has the following data:

    ```
    | empno | ename | job | mgr | hiredate| sal| comm | deptno |
    7369,SMITH,CLERK,7902,1980-12-17 00:00:00,800,,20
    7499,ALLEN,SALESMAN,7698,1981-02-20 00:00:00,1600,300,30
    7521,WARD,SALESMAN,7698,1981-02-22 00:00:00,1250,500,30
    7566,JONES,MANAGER,7839,1981-04-02 00:00:00,2975,,20
    7654,MARTIN,SALESMAN,7698,1981-09-28 00:00:00,1250,1400,30
    7698,BLAKE,MANAGER,7839,1981-05-01 00:00:00,2850,,30
    7782,CLARK,MANAGER,7839,1981-06-09 00:00:00,2450,,10
    7788,SCOTT,ANALYST,7566,1987-04-19 00:00:00,3000,,20
    7839,KING,PRESIDENT,,1981-11-17 00:00:00,5000,,10
    7844,TURNER,SALESMAN,7698,1981-09-08 00:00:00,1500,0,30
    7876,ADAMS,CLERK,7788,1987-05-23 00:00:00,1100,,20
    7900,JAMES,CLERK,7698,1981-12-03 00:00:00,950,,30
    7902,FORD,ANALYST,7566,1981-12-03 00:00:00,3000,,20
    7934,MILLER,CLERK,7782,1982-01-23 00:00:00,1300,,10
    7948,JACCKA,CLERK,7782,1981-04-12 00:00:00,5000,,10
    7956,WELAN,CLERK,7649,1982-07-20 00:00:00,2450,,10
    7956,TEBAGE,CLERK,7748,1982-12-30 00:00:00,1300,,10
    ```

    Group employees by department, sort the employees in each group in descending order of `sal`, and then obtain the sequence numbers of employees in each group.

    ```
    -- deptno that specifies a department is used as a partitioning column. sal that specifies the salary is used as the value to be sorted in the returned result.
    SELECT deptno, ename, sal, DENSE_RANK() OVER (PARTITION BY deptno ORDER BY sal DESC) AS nums
        FROM emp;
    -- The following result is returned:
    +------------+-------+------------+------------+
    | deptno     | ename | sal        | nums       |
    +------------+-------+------------+------------+
    | 10         | JACCKA | 5000.0     | 1          |
    | 10         | KING  | 5000.0     | 1          |
    | 10         | CLARK | 2450.0     | 2          |
    | 10         | WELAN | 2450.0     | 2          |
    | 10         | TEBAGE | 1300.0     | 3          |
    | 10         | MILLER | 1300.0     | 3          |
    | 20         | SCOTT | 3000.0     | 1          |
    | 20         | FORD  | 3000.0     | 1          |
    | 20         | JONES | 2975.0     | 2          |
    | 20         | ADAMS | 1100.0     | 3          |
    | 20         | SMITH | 800.0      | 4          |
    | 30         | BLAKE | 2850.0     | 1          |
    | 30         | ALLEN | 1600.0     | 2          |
    | 30         | TURNER | 1500.0     | 3          |
    | 30         | MARTIN | 1250.0     | 4          |
    | 30         | WARD  | 1250.0     | 4          |
    | 30         | JAMES | 950.0      | 5          |
    +------------+-------+------------+------------+
    ```


## RANK

-   Syntax

    ```
    Bigint rank() over(partition by [col1, col2…]
    order by [col1[asc|desc], col2[asc|desc]…])
    ```

-   Description

    Returns the rankings of data in a window. If two rows have the same col2, the ranking of the second row drops.

-   Parameters
    -   `partition by [col1, col2..]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: the field based on which data is ranked.
-   Return value

    Returns a value of the BIGINT type.

-   Example

    Table `emp` has the following data:

    ```
    | empno | ename | job | mgr | hiredate| sal| comm | deptno |
    7369,SMITH,CLERK,7902,1980-12-17 00:00:00,800,,20
    7499,ALLEN,SALESMAN,7698,1981-02-20 00:00:00,1600,300,30
    7521,WARD,SALESMAN,7698,1981-02-22 00:00:00,1250,500,30
    7566,JONES,MANAGER,7839,1981-04-02 00:00:00,2975,,20
    7654,MARTIN,SALESMAN,7698,1981-09-28 00:00:00,1250,1400,30
    7698,BLAKE,MANAGER,7839,1981-05-01 00:00:00,2850,,30
    7782,CLARK,MANAGER,7839,1981-06-09 00:00:00,2450,,10
    7788,SCOTT,ANALYST,7566,1987-04-19 00:00:00,3000,,20
    7839,KING,PRESIDENT,,1981-11-17 00:00:00,5000,,10
    7844,TURNER,SALESMAN,7698,1981-09-08 00:00:00,1500,0,30
    7876,ADAMS,CLERK,7788,1987-05-23 00:00:00,1100,,20
    7900,JAMES,CLERK,7698,1981-12-03 00:00:00,950,,30
    7902,FORD,ANALYST,7566,1981-12-03 00:00:00,3000,,20
    7934,MILLER,CLERK,7782,1982-01-23 00:00:00,1300,,10
    7948,JACCKA,CLERK,7782,1981-04-12 00:00:00,5000,,10
    7956,WELAN,CLERK,7649,1982-07-20 00:00:00,2450,,10
    7956,TEBAGE,CLERK,7748,1982-12-30 00:00:00,1300,,10
    ```

    Group employees by department, sort the employees in each group in descending order of `sal`, and then obtain the sequence numbers of employees in each group.

    ```
    -- deptno that specifies a department is used as a partitioning column. sal that specifies the salary is used as the value to be sorted in the returned result.
    SELECT deptno,ename,sal, RANK() OVER (PARTITION BY deptno ORDER BY sal DESC) AS nums
    FROM emp;
    -- The following result is returned:
    +------------+-------+------------+------------+
    | deptno     | ename | sal        | nums       |
    +------------+-------+------------+------------+
    | 10         | JACCKA | 5000.0     | 1          |
    | 10         | KING  | 5000.0     | 1          |
    | 10         | CLARK | 2450.0     | 3          |
    | 10         | WELAN | 2450.0     | 3          |
    | 10         | TEBAGE | 1300.0     | 5          |
    | 10         | MILLER | 1300.0     | 5          |
    | 20         | SCOTT | 3000.0     | 1          |
    | 20         | FORD  | 3000.0     | 1          |
    | 20         | JONES | 2975.0     | 3          |
    | 20         | ADAMS | 1100.0     | 4          |
    | 20         | SMITH | 800.0      | 5          |
    | 30         | BLAKE | 2850.0     | 1          |
    | 30         | ALLEN | 1600.0     | 2          |
    | 30         | TURNER | 1500.0     | 3          |
    | 30         | MARTIN | 1250.0     | 4          |
    | 30         | WARD  | 1250.0     | 4          |
    | 30         | JAMES | 950.0      | 6          |
    +------------+-------+------------+------------+
    ```


## LAG

-   Syntax

    ```
    lag(expr, Bigint offset, default) over(partition by [col1, col2…]
    [order by [col1[asc|desc], col2[asc|desc]…]])
    ```

-   Description

    Returns the value for a row at a given offset preceding the current row. If the current row number is `rn`, the value of the row with the number of `rn-offset` is retrieved.

-   Parameters
    -   `expr`: a value of any data type.
    -   offset: a constant of the BIGINT type. The value of the offset must be greater than 0. If the input value is of the STRING or DOUBLE type, it is implicitly converted into the BIGINT type before calculation.
    -   default: the default value that is used if offset is out of the valid range. It is a constant value and its default value is NULL.
    -   `partition by [col1, col2..]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: the field based on which data is ranked.
-   Return value

    Returns a value of the same data type as `expr`.

-   Example

    ```
    select seq, lag(seq+100, 1) over (partition by window order by seq) as r from sliding_window;
    -- The following result is returned:
    +------------+------------+
    | seq        | r          |
    +------------+------------+
    | 0          | NULL       |
    | 1          | 100        |
    | 2          | 101        |
    | 3          | 102        |
    | 4          | 103        |
    | 5          | 104        |
    | 6          | 105        |
    | 7          | 106        |
    | 8          | 107        |
    | 9          | 108        |
    +------------+------------+
    ```


## LEAD

-   Syntax

    ```
    lead(expr, Bigint offset, default) over(partition by [col1, col2…]
    [order by [col1[asc|desc], col2[asc|desc]…]])
    ```

-   Description

    Retrieves the value for a row at a given offset following the current row. If the current row number is `rn`, the value of the row with the number of `rn+offset` is retrieved.

-   Parameters
    -   `expr`: a value of any data type.
    -   offset: a constant of the BIGINT type. The value of the offset must be greater than 0. If the input value is of the STRING or DOUBLE type, it is implicitly converted into the BIGINT type before calculation.
    -   default: the default value that is used if offset is out of the valid range. It is a constant value and its default value is NULL.
    -   `partition by [col1, col2..]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: the field based on which data is ranked.
-   Return value

    Returns a value of the same data type as `expr`.

-   Example

    ```
    select c_Double_a,c_String_b,c_int_a,lead(c_int_a,1) over(partition by c_Double_a order by c_String_b);
    select c_String_a,c_time_b,c_Double_a,lead(c_Double_a,1) over(partition by c_String_a order by c_time_b);
    select c_String_in_fact_num,c_String_a,c_int_a,lead(c_int_a) over(partition by c_String_in_fact_num order by c_String_a);
    ```


## PERCENT\_RANK

-   Syntax

    ```
    percent_rank() over(partition by [col1, col2…]
    order by [col1[asc|desc], col2[asc|desc]…])
    ```

-   Description

    Calculates the relative ranking of a row in a group of data.

-   Parameters
    -   `partition by [col1, col2..]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: the field based on which data is ranked.
-   Return value

    Returns a value of the DOUBLE type. The value can be 0 or 1. The relative ranking is calculated by using the following formula: `(Rank - 1)/(Number of rows - 1)`


## ROW\_NUMBER

-   Syntax

    ```
    row_number() over(partition by [col1, col2…]
    order by [col1[asc|desc], col2[asc|desc]…])
    ```

-   Description

    Calculates the row number, starting from 1.

-   Parameters
    -   `partition by [col1, col2..]`: partitioning columns.
    -   `order by col1[asc|desc], col2[asc|desc]`: the field based on which data is ranked.
-   Return value

    Returns a value of the BIGINT type.

-   Example

    Table `emp` has the following data:

    ```
    | empno | ename | job | mgr | hiredate| sal| comm | deptno |
    7369,SMITH,CLERK,7902,1980-12-17 00:00:00,800,,20
    7499,ALLEN,SALESMAN,7698,1981-02-20 00:00:00,1600,300,30
    7521,WARD,SALESMAN,7698,1981-02-22 00:00:00,1250,500,30
    7566,JONES,MANAGER,7839,1981-04-02 00:00:00,2975,,20
    7654,MARTIN,SALESMAN,7698,1981-09-28 00:00:00,1250,1400,30
    7698,BLAKE,MANAGER,7839,1981-05-01 00:00:00,2850,,30
    7782,CLARK,MANAGER,7839,1981-06-09 00:00:00,2450,,10
    7788,SCOTT,ANALYST,7566,1987-04-19 00:00:00,3000,,20
    7839,KING,PRESIDENT,,1981-11-17 00:00:00,5000,,10
    7844,TURNER,SALESMAN,7698,1981-09-08 00:00:00,1500,0,30
    7876,ADAMS,CLERK,7788,1987-05-23 00:00:00,1100,,20
    7900,JAMES,CLERK,7698,1981-12-03 00:00:00,950,,30
    7902,FORD,ANALYST,7566,1981-12-03 00:00:00,3000,,20
    7934,MILLER,CLERK,7782,1982-01-23 00:00:00,1300,,10
    7948,JACCKA,CLERK,7782,1981-04-12 00:00:00,5000,,10
    7956,WELAN,CLERK,7649,1982-07-20 00:00:00,2450,,10
    7956,TEBAGE,CLERK,7748,1982-12-30 00:00:00,1300,,10
    ```

    Group employees by department, sort the employees in each group in descending order of `sal`, and then obtain the sequence numbers of employees in each group.

    ```
    SELECT deptno,ename,sal,ROW_NUMBER() OVER (PARTITION BY deptno ORDER BY sal DESC) AS nums -- deptno that specifies a department is used as a partitioning column. sal that specifies the salary is used as the value to be sorted in the returned result.
        FROM emp;
    -- The following result is returned:
    +------------+-------+------------+------------+
    | deptno     | ename | sal        | nums       |
    +------------+-------+------------+------------+
    | 10         | JACCKA | 5000.0     | 1          |
    | 10         | KING  | 5000.0     | 2          |
    | 10         | CLARK | 2450.0     | 3          |
    | 10         | WELAN | 2450.0     | 4          |
    | 10         | TEBAGE | 1300.0     | 5          |
    | 10         | MILLER | 1300.0     | 6          |
    | 20         | SCOTT | 3000.0     | 1          |
    | 20         | FORD  | 3000.0     | 2          |
    | 20         | JONES | 2975.0     | 3          |
    | 20         | ADAMS | 1100.0     | 4          |
    | 20         | SMITH | 800.0      | 5          |
    | 30         | BLAKE | 2850.0     | 1          |
    | 30         | ALLEN | 1600.0     | 2          |
    | 30         | TURNER | 1500.0     | 3          |
    | 30         | MARTIN | 1250.0     | 4          |
    | 30         | WARD  | 1250.0     | 5          |
    | 30         | JAMES | 950.0      | 6          |
    +------------+-------+------------+------------+
    ```


## CLUSTER\_SAMPLE

-   Syntax

    ```
    boolean cluster_sample([Bigint x, Bigint y])
    over(partition by [col1, col2..])
    ```

-   Description

    Performs group sampling on data in a window.

-   Parameters
    -   x: a constant of the BIGINT type. The value of `x` must be greater than or equal to 1. If y is specified, x indicates that a window is divided into x portions. Otherwise, x indicates that the records of x rows in a window are extracted. In this case, True is returned for the x rows. If x is NULL, NULL is returned.
    -   y: a constant of the BIGINT type. The value of `y` must be greater than or equal to 1 and must be less than or equal to `x`. y indicates that y records of the x portions in a window are extracted. In this case, True is returned for the y records. If y is set to NULL, NULL is returned.
    -   `partition by [col1, col2]`: partitioning columns.
-   Return value

    Returns a value of the BOOLEAN type.

-   Example

    Table `test_tbl` has the `key` and `value` columns, `key` is the grouping field and the value options include `groupa` and `groupb`. value is the key value, as shown in the following code:

    ```
    +------------+--------------------+
    | key        | value              |
    +------------+--------------------+
    | groupa     | -1.34764165478145  |
    | groupa     | 0.740212609046718  |
    | groupa     | 0.167537127858695  |
    | groupa     | 0.630314566185241  |
    | groupa     | 0.0112401388646925 |
    | groupa     | 0.199165745875297  |
    | groupa     | -0.320543343353587 |
    | groupa     | -0.273930924365012 |
    | groupa     | 0.386177958942063  |
    | groupa     | -1.09209976687047  |
    | groupb     | -1.10847690938643  |
    | groupb     | -0.725703978381499 |
    | groupb     | 1.05064697475759   |
    | groupb     | 0.135751224393789  |
    | groupb     | 2.13313102040396   |
    | groupb     | -1.11828960785008  |
    | groupb     | -0.849235511508911 |
    | groupb     | 1.27913806620453   |
    | groupb     | -0.330817716670401 |
    | groupb     | -0.300156896191195 |
    | groupb     | 2.4704244205196    |
    | groupb     | -1.28051882084434  |
    +------------+--------------------+
    ```

    If you want to extract a sample of 10% of the values in each group, execute the following MaxCompute SQL statement:

    ```
    select key, value
        from (
            select key, value, cluster_sample(10, 1) over(partition by key) as flag
            from tbl
            ) sub
        where flag = true;
    -- The following result is returned:
    +-----+------------+
    | key | value      |
    +-----+------------+
    | groupa | 0.167537127858695 |
    | groupb | 0.135751224393789 |
    +-----+------------+
    ```


## CUME\_DIST

-   Syntax

    ```
    cume_dist() over(partition by col1[, col2…] order by col1 [asc|desc][, col2[asc|desc]…]])
    ```

-   Description

    Calculates the cumulative distribution. The cumulative distribution is the ratio of rows whose values are greater than or equal to the current value to all rows in a group.

-   Parameter

    `ORDER BY`: the value used for comparison.

-   Return value

    Returns the ratio of rows whose values are greater than or equal to the current value to all rows in a group.

-   Example

    Table `emp` has the following data:

    ```
    | empno | ename | job | mgr | hiredate| sal| comm | deptno |
    7369,SMITH,CLERK,7902,1980-12-17 00:00:00,800,,20
    7499,ALLEN,SALESMAN,7698,1981-02-20 00:00:00,1600,300,30
    7521,WARD,SALESMAN,7698,1981-02-22 00:00:00,1250,500,30
    7566,JONES,MANAGER,7839,1981-04-02 00:00:00,2975,,20
    7654,MARTIN,SALESMAN,7698,1981-09-28 00:00:00,1250,1400,30
    7698,BLAKE,MANAGER,7839,1981-05-01 00:00:00,2850,,30
    7782,CLARK,MANAGER,7839,1981-06-09 00:00:00,2450,,10
    7788,SCOTT,ANALYST,7566,1987-04-19 00:00:00,3000,,20
    7839,KING,PRESIDENT,,1981-11-17 00:00:00,5000,,10
    7844,TURNER,SALESMAN,7698,1981-09-08 00:00:00,1500,0,30
    7876,ADAMS,CLERK,7788,1987-05-23 00:00:00,1100,,20
    7900,JAMES,CLERK,7698,1981-12-03 00:00:00,950,,30
    7902,FORD,ANALYST,7566,1981-12-03 00:00:00,3000,,20
    7934,MILLER,CLERK,7782,1982-01-23 00:00:00,1300,,10
    7948,JACCKA,CLERK,7782,1981-04-12 00:00:00,5000,,10
    7956,WELAN,CLERK,7649,1982-07-20 00:00:00,2450,,10
    7956,TEBAGE,CLERK,7748,1982-12-30 00:00:00,1300,,10
    ```

    Group all employees by department and obtain the cumulative distribution of `sal` for each group.

    ```
    SELECT deptno
    , ename
    , sal
    , concat(round(cume_dist() OVER(PARTITION BY deptno ORDER BY sal desc)*100,2),'%') as cume_dist
    FROM emp;
    -- The following result is returned:
    +------------+-------+------------+-----------+
    | deptno     | ename | sal        | cume_dist |
    +------------+-------+------------+-----------+
    | 10         | JACCKA | 5000.0     | 33.33%    |
    | 10         | KING  | 5000.0     | 33.33%    |
    | 10         | CLARK | 2450.0     | 66.67%    |
    | 10         | WELAN | 2450.0     | 66.67%    |
    | 10         | TEBAGE | 1300.0     | 100.0%    |
    | 10         | MILLER | 1300.0     | 100.0%    |
    | 20         | SCOTT | 3000.0     | 40.0%     |
    | 20         | FORD  | 3000.0     | 40.0%     |
    | 20         | JONES | 2975.0     | 60.0%     |
    | 20         | ADAMS | 1100.0     | 80.0%     |
    | 20         | SMITH | 800.0      | 100.0%    |
    | 30         | BLAKE | 2850.0     | 16.67%    |
    | 30         | ALLEN | 1600.0     | 33.33%    |
    | 30         | TURNER | 1500.0     | 50.0%     |
    | 30         | MARTIN | 1250.0     | 83.33%    |
    | 30         | WARD  | 1250.0     | 83.33%    |
    | 30         | JAMES | 950.0      | 100.0%    |
    +------------+-------+------------+-----------+
    ```


