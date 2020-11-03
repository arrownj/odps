---
keyword: MaxCompute SQL
---

# Write MaxCompute SQL statements

This topic describes the common scenarios of using MaxCompute SQL statements and how to write them.

## Prepare a dataset

The emp and dept tables are used as the dataset in this example. You can create a table on a MaxCompute project and upload data to the table. For more information about how to import data, see [Overview](/intl.en-US/Development/Data upload and download/Overview.md).

Download [data files of the emp table](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/51009/cn_zh/1489374509403/emp%E6%95%B0%E6%8D%AE.txt) and [data files of the dept table](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/51009/cn_zh/1488265664915/dept%E8%A1%A8%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6.txt).

Execute the following statement to create the emp table:

```
CREATE TABLE IF NOT EXISTS emp (
  EMPNO STRING,
  ENAME STRING,
  JOB STRING,
  MGR BIGINT,
  HIREDATE DATETIME,
  SAL DOUBLE,
  COMM DOUBLE,
  DEPTNO BIGINT);
```

Execute the following statement to create the dept table:

```
CREATE TABLE IF NOT EXISTS dept (
  DEPTNO BIGINT,
  DNAME STRING,
  LOC STRING);
```

## Examples

-   Example 1: Query all departments that have at least one employee.

    We recommend that you use the JOIN clause to avoid large amounts of data in the query. Execute the following SQL statement:

    ```
    SELECT d. *
    FROM dept d
    JOIN (
        SELECT DISTINCT deptno AS no
        FROM emp
    ) e
    ON d.deptno = e.no;
    ```

-   Example 2: Query all employees who have higher salaries than Smith.

    The following code shows how to use MAPJOIN in the SQL statement for this scenario:

    ```
    SELECT /*+ MapJoin(a) */ e.empno
        , e.ename
        , e.sal
    FROM emp e
    JOIN (
        SELECT MAX(sal) AS sal
        FROM `emp`
        WHERE `ENAME` = 'SMITH'
    ) a
    ON e.sal > a.sal;
    ```

-   Example 3: Query the names of all employees and the names of their immediate superiors.

    The following code shows how to use EQUI JOIN in the SQL statement for this scenario:

    ```
    SELECT a.ename
        , b.ename
    FROM emp a
    LEFT OUTER JOIN emp b
    ON b.empno = a.mgr;
    ```

-   Example 4: Query all jobs that have basic salaries higher than USD 1,500.

    The following code shows how to use the HAVING clause in the SQL statement for this scenario:

    ```
    SELECT emp. `JOB`
        , MIN(emp.sal) AS sal
    FROM `emp`
    GROUP BY emp. `JOB`
    HAVING MIN(emp.sal) > 1500;
    ```

-   Example 5: Query the number of employees in each department, the average salary, and the average length of service.

    The following code shows how to use built-in functions in the SQL statement for this scenario:

    ```
    SELECT COUNT(empno) AS cnt_emp
        , ROUND(AVG(sal), 2) AS avg_sal
        , ROUND(AVG(datediff(getdate(), hiredate, 'dd')), 2) AS avg_hire
    FROM `emp`
    GROUP BY `DEPTNO`;
    ```

-   Example 6: Query the names and the sorting order of the first three employees who have the highest salaries.

    The following code shows how to use the TOP N clause in the SQL statement for this scenario:

    ```
    SELECT *
    FROM (
      SELECT deptno
        , ename
        , sal
        , ROW_NUMBER() OVER (PARTITION BY deptno ORDER BY sal DESC) AS nums
      FROM emp
    ) emp1
    WHERE emp1.nums < 4;
    ```

-   Example 7: Query the number of employees in each department and the proportion of clerks in these departments.

    ```
    SELECT deptno
        , COUNT(empno) AS cnt
        , ROUND(SUM(CASE 
          WHEN job = 'CLERK' THEN 1
          ELSE 0
        END) / COUNT(empno), 2) AS rate
    FROM `EMP`
    GROUP BY deptno;
    ```


## Notes

-   When you use the GROUP BY clause, the SELECT list can only consist of aggregate functions or columns that are part of the GROUP BY clause.
-   ORDER BY must be followed by LIMIT N.
-   The SELECT expression does not support subqueries. To use subqueries, you can rewrite the code to include a JOIN clause.
-   The JOIN clause does not support Cartesian projects. You can replace the JOIN clause with MAPJOIN.
-   UNION ALL must be replaced with subqueries.
-   The subquery that is specified in the IN or NOT IN clause must contain only one column and return a maximum of 1,000 rows. Otherwise, use the JOIN clause.

