# Demo: Modify incompatible SQL

The MaxCompute development team has completed the grayscale upgrade of MaxCompute 2.0 recently. The new version fully embraces open source ecosystems, supports more languages and functions, and enables faster operation. It also implements more rigorous syntax inspection, so that errors may be returned for some less rigorous syntax cases that can run normally in the earlier editor.

To enable smooth grayscale upgrade of MaxCompute 2.0, the MaxCompute framework supports rollback. If a task of MaxCompute 2.0 fails, it will be executed in MaxCompute 1.0. The rollback increases the E2E latency of the task. We recommend that you set `set odps.sql.planner.mode=lot;` to manually disable the rollback function before submitting jobs, to avoid the impact resulting from the changes made to the MaxCompute rollback policy.

The MaxCompute team notifies the owners of problematic tasks by email or DingTalk based on the online rollback condition. Modify your SQL tasks immediately; otherwise, the tasks may fail. Check the following errors against your tasks to avoid task failure in case of missed notifications.

The following lists the syntaxes for which MaxCompute 2.0 may return errors.

## group.by.with.star

**SELECT \* … GROUP BY… syntax is problematic.**

In the earlier version of MaxCompute, `select * from group by key` is supported even when the columns that match \* are not included in the GROUP BY key. Compatible with Hive, MaxCompute 2.0 prohibits this syntax unless the GROUP BY list is a column in all source tables. Examples:

Scenario 1: **The GROUP BY key does not include all columns**.

Incorrect syntax:

```
SELECT * FROM t GROUP BY key;
```

Error message:

```
FAILED: ODPS-0130071:[1,8] Semantic analysis exception - column reference t.value should appear in GROUP BY key
```

Correct syntax:

```
SELECT DISTINCT key FROM t;
```

Scenario 2: **The GROUP BY key includes all columns**.

Not recommended syntax:

```
SELECT * FROM t GROUP BY key, value; -- t has columns key and value
```

Though MaxCompute 2.0 does not return an error, we recommend that you modify the syntax as follows:

```
SELECT DISTINCT key, value FROM t;
```

## bad.escape

**The escape sequence is incorrect.**

According to the MaxCompute documentation, in string literal, every ASCII character ranging from 0 to 127 must be written in the format of a backslash followed by three octal numbers. For example, 0 is written as \\001, and 1 is written as \\002. Currently, \\01 and \\0001 are considered as \\001.

It may bring confusions to new users. For example, “\\000” + “1” cannot written as “\\0001”. For users who migrate data from other systems to MaxCompute, incorrect data may be generated.

**Note:** An error may be returned if numbers are appended to `\0 00`, for example, `\0001 - \0009 or \00001`.

MaxCompute 2.0 solves this problem. The script owner must correct the sequence, for example:

Incorrect syntax:

```
SELECT split(key, "\01"), value like "\0001" FROM t;
```

Error message:

```

FAILED: ODPS-0130161:[1,19] Parse exception - unexpected escape sequence: 01
ODPS-0130161:[1,38] Parse exception - unexpected escape sequence: 0001-
```

Correct syntax:

```
SELECT split(key, "\001"), value like "\001" FROM t;
```

## column.repeated.in.creation

**Duplicate column names are detected during execution of the CREATE TABLE statement.**

MaxCompute 2.0 returns an error when duplicate column names are detected when the CREATE TABLE statement is executed. For example:

Incorrect syntax:

```
CREATE TABLE t (a BIGINT, b BIGINT, a BIGINT);
```

Error message:

```
FAILED: ODPS-0130071:[1,37] Semantic analysis exception - column repeated in creation: a
```

Correct syntax:

```
CREATE TABLE t (a BIGINT, b BIGINT);
```

## string.join.double

**The data on both sides of the equal sign of a Join condition belongs to the String and Double types.**

In the old version of MaxCompute, the String and Double type data is converted to the Bigint type at the cost of precision. 1.1 = “1” in a Join condition is considered as equal But compatible with Hive, MaxCompute 2.0 converts the String and Double type data to the Double type.

Syntax that is not recommended:

```
SELECT * FROM t1 JOIN t2 ON t1.double_value = t2.string_value;
```

Warning:

```
WARNING:[1,48]  implicit conversion from STRING to DOUBLE, potential data loss, use CAST function to suppress
```

Recommended correction:

```
select * from t1 join t2 on t.double_value = cast(t2.string_value as double);
```

You can also convert the data as needed.

## window.ref.prev.window.alias

**Window functions reference other aliases in the select list of the same level.**

Examples:

With rn absent from t1, the incorrect syntax is as follows:

```

SELECT row_number() OVER (PARTITION BY c1 ORDER BY c1) rn,
row_number() OVER (PARTITION by c1 ORDER BY rn) rn2
FROM t1;
```

Error message:

```
FAILED: ODPS-0130071:[2,45] Semantic analysis exception - column rn cannot be resolved
```

Correct syntax:

```

SELECT row_number() OVER (PARTITION BY c1 ORDER BY rn) rn2
FROM
(
SELECT c1, row_number() OVER (PARTITION BY c1 ORDER BY c1) rn
FROM t1
) tmp;
```

## select.invalid.token.after.star

**Select \* is followed by an alias.**

The select list allows the use of \* to select all the columns of a table, but \* cannot be followed by any alias even if the expanded \* has only one column. The new editor returns errors for similar syntaxes. For example:

Incorrect syntax:

```
select * as alias from dual;
```

Error message:

```
FAILED: ODPS-0130161:[1,10] Parse exception - invalid token 'as'
```

Correct syntax:

```
select * from dual;
```

## agg.having.ref.prev.agg.alias

**The select list is preceded by an aggregate function alias when HAVING exists.** Examples:

Incorrect syntax:

```

SELECT count(c1) cnt,
sum(c1) / cnt avg
FROM t1
GROUP BY c2
HAVING cnt > 1;
```

Error message:

```

FAILED: ODPS-0130071:[2,11] Semantic analysis exception - column cnt cannot be resolved
ODPS-0130071:[2,11] Semantic analysis exception - column reference cnt should appear in GROUP BY key
```

s and cnt do not exist in source table t1, but MaxCompute of the old version does not return an error because HAVING exists. In MaxCompute 2.0, the prompt “column cannot be resolved” appears and an error is returned.

Correct syntax:

```

SELECT cnt, s, s/cnt avg
FROM
(
SELECT count(c1) cnt,
sum(c1) s
FROM t1
GROUP BY c2
HAVING count(c1) > 1
) tmp;
```

## order.by.no.limit

**ORDER BY is not followed by the LIMIT statement.**

By default, MaxCompute requires that ORDER BY be followed by the LIMIT statement to limit the number of data records. Because ORDER BY is used for full data sorting, the execution performance is low without the LIMIT statement. Examples:

Incorrect syntax:

```

select * from (select *
from (select cast(login_user_cnt as int) as uv, '3' as shuzi
from test_login_cnt where type = 'device' and type_name = 'mobile') v
order by v.uv desc) v
order by v.shuzi limit 20;
```

Error message:

```
FAILED: ODPS-0130071:[4,1] Semantic analysis exception - ORDER BY must be used with a LIMIT clause
```

Correct syntax:

Add the LIMIT statement to the subquery “order by v.uv desc”.

In MaxCompute 1.0, the view inspection is not strict. For example, a view is created in a project which does not require LIMIT statement check \(odps.sql.validate.orderby.limit=false\).

```
CREATE VIEW dual_view AS SELECT id FROM dual ORDER BY id;
```

When you run the following statement to access the view:

```
SELECT * FROM dual_view;
```

MaxCompute 1.0 does not return an error, whereas MaxCompute 2.0 returns the following error:

```
FAILED: ODPS-0130071:[1,15] Semantic analysis exception - while resolving view xdj.xdj_view_limit - ORDER BY must be used with a LIMIT clause
```

## generated.column.name.multi.window

**Automatically generated aliases are used.**

In the earlier version of MaxCompute, an alias is auto generated for every expression of the SELECT statement. The alias is displayed on the console. However, the earlier version does not guarantee that the alias generation rule is correct or remains unchanged. Therefore, we do not recommend that you use auto generated aliases.

In MaxCompute 2.0, a warning is given for use of auto generated aliases, but due to breadth of involvement such use is not prohibited for the moment.

In some cases, known changes are made to the alias generation rules in the different versions of MaxCompute. Some online jobs depend on the auto generated aliases. Queries may fail when MaxCompute performs version upgrade or rollback. If you have such problems, modify queries and explicitly specify the alias of the column of interest. Examples:

Not recommended syntax:

```
SELECT _c0 FROM (SELECT count(*) FROM dual) t;
```

Recommended correction:

```
SELECT c FROM (SELECT count(*) c FROM dual) t;
```

## non.boolean.filter

**Non-Boolean filter conditions are used.**

MaxCompute prohibits the implicit conversion between the Boolean type and other data types. However, the earlier version of MaxCompute allows the use of Bigint type filter conditions in some cases. MaxCompute 2.0 prohibits the use of Bigint type filter conditions. If your scripts have Bigint type filter conditions, modify them promptly. Examples:

Incorrect syntax:

```
select id, count(*) from dual group by id having id;
```

Error message:

```
FAILED: ODPS-0130071:[1,50] Semantic analysis exception - expect a BOOLEAN expression
```

Correct syntax:

```
select id, count(*) from dual group by id having id <> 0;
```

## post.select.ambiguous

**The GROUP BY, CLUSTER BY, DISTRIBUTE BY, and SORT BY statements reference columns with conflicting names.**

In the old version of MaxCompute, by default, the system selects the last column of the select list as the operation object. MaxCompute 2.0 reports an error in this case. Make relevant modification timely. Examples:

Incorrect syntax:

```
select a, b as a from t order by a limit 10;
```

Error message:

```
FAILED: ODPS-0130071:[1,34] Semantic analysis exception - a is ambiguous, can be both t.a or null.a
```

Correct syntax:

```
select a as c, b as a from t order by a limit 10;
```

The pushed change covers the statements with conflicting column names but the same syntax. Although there is no ambiguity, an error is often returned for these statements and a warning is triggered. We recommend that you make relevant modification.

## duplicated.partition.column

**Partitions with the same name are specified in a query.**

In the earlier version of MaxCompute, no error is returned when two partition keys with the same name are specified. The latter partition key overwrites the former one. It causes confusion. MaxCompute 2.0 returns an error in this case.

Incorrect syntax:

```
insert overwrite table partition (ds = '1', ds = '2'）select ... ;
```

In fact, ds = ‘1’ is ignored during execution.

Correct syntax:

```
insert overwrite table partition (ds = '2'）select ... ;
```

Incorrect syntax:

```
create table t (a bigint, ds string) partitioned by (ds string);
```

Correct syntax:

```
create table t (a bigint) partitioned by (ds string);
```

## order.by.col.ambiguous

**The ORDER BY clause references the duplicate aliases in the select list.**

Incorrect syntax:

```

SELECT id, id
FROM dual 
ORDER BY id;
```

Correct syntax:

```

SELECT id, id id2
FROM dual 
ORDER BY id;
```

Remove the duplicate aliases before the ORDER BY clause can reference them.

## in.subquery.without.result

**colx does not exist in the source table if colx in subquery does not return any results.**

Incorrect syntax:

```

SELECT * FROM dual
WHERE not_exist_col IN (SELECT id FROM dual LIMIT 0);
```

Error message:

```
FAILED: ODPS-0130071:[2,7] Semantic analysis exception - column not_exist_col cannot be resolved
```

## ctas.if.not.exists

**The syntax of the target table is incorrect.**

If the target table exists, the earlier version of MaxCompute does not check the syntax, whereas MaxCompute 2.0 does. Many errors may return in this case, for example:

Incorrect syntax:

```

CREATE TABLE IF NOT EXISTS dual
AS
SELECT * FROM not_exist_table;
```

Error message:

```
FAILED: ODPS-0130131:[1,50] Table not found - table meta_dev.not_exist_table cannot be resolved
```

## worker.restart.instance.timeout

In the earlier version of MaxCompute, every time a UDF outputs a record, a write operation is triggered on the distributed file system, and a heartbeat packet is sent to Fuxi. If the UDF does not output any records for 10 minutes, the following error is reported:

```
FAILED: ODPS-0123144: Fuxi job failed - WorkerRestart errCode:252,errMsg:kInstanceMonitorTimeout, usually caused by bad udf performance.
```

The runtime framework of MaxCompute 2.0 supports vectoring. It processes multiple rows of a column at a time to improve the execution efficiency. Vectoring may cause the normal statements to time out in the case that a heartbeat packet is not sent to Fuxi within the specified time while multiple records are processed at a time. The interval between two output records does not exceed 10 minutes.

If a time-out error occurs, we recommend that you first check the UDF performance. It takes several seconds to process each record. If the UDF performance cannot be optimized, as a workaround, you can set the value of “batch row” manually. The default value is 1024.

```
set odps.sql.executionengine.batch.rowcount=16;
```

## divide.nan.or.overflow

**The old version of MaxCompute does not support division constant folding.**

In the old version of MaxCompute, the physical execution plan for a statement is as follows:

```

EXPLAIN
Select if (false, 0/0, 1.0)
FROM dual;
In Task M1_Stg1:
    Data source: meta_dev.dual
    TS: alias: dual
      SEL: If(False, Divide(UDFToDouble(0), UDFToDouble(0)), 1.0)
        FS: output: None
```

The IF and Divide functions are retained. During execution, the first parameter of IF is set to “false”, and the expression of the second parameter Divide is not evaluated. Division by zero is normal.

MaxCompute 2.0 supports division constant folding. An error is returned. As shown in the following:

Incorrect syntax:

```

SELECT IF(FALSE, 0/0, 1.0)
FROM dual;
```

Error message:

```
FAILED: ODPS-0130071:[1,19] Semantic analysis exception - encounter runtime exception while evaluating function /, detailed message: DIVIDE func result NaN, two params are 0.000000 and 0.000000
```

An overflow error may occur besides nan. For example:

Incorrect syntax:

```

SELECT IF(FALSE, 1/0, 1.0)
FROM dual;
```

Error message:

```
FAILED: ODPS-0130071:[1,19] Semantic analysis exception - encounter runtime exception while evaluating function /, detailed message: DIVIDE func result overflow, two params are 1.000000 and 0.000000
```

Correct syntax:

We recommend that you remove /0 and use valid constants.

A similar problem occurs in the constant folding of CASE WHEN, such as CASE WHEN TRUE THEN 0 ELSE 0/0. During constant folding in MaxCompute 2.0, all subexpressions are evaluated, causing incorrect division by zero.

CASE WHEN may involve more complex optimization scenarios, for example:

```

SELECT CASE WHEN key = 0 THEN 0 ELSE 1/key END
FROM (
SELECT 0 AS key FROM src
UNION ALL
SELECT key FROM src) r;
```

The optimizer pushes down the division operation to subqueries. A similar conversion is as follows:

```

M (
SELECT CASE WHEN 0 = 0 THEN 0 ELSE 1/0 END c1 FROM src
UNION ALL
SELECT CASE WHEN key = 0 THEN 0 ELSE 1/key END c1 FROM src) r;
```

Error message:

```
FAILED: ODPS-0130071:[0,0] Semantic analysis exception - physical plan generation failed: java.lang.ArithmeticException: DIVIDE func result overflow, two params are 1.000000 and 0.000000
```

An error is returned for the constant folding of the first clause of UNION ALL. We recommend that you transfer CASE WHEN in SQL to subqueries and eliminate useless CASE WHEN statements and /0.

```

SELECT c1 END
FROM (
SELECT 0 c1 END FROM src
UNION ALL
SELECT CASE WHEN key = 0 THEN 0 ELSE 1/key END) r;
```

## small.table.exceeds.mem.limit

The earlier version of MaxCompute supports Multi-way Join optimization. Multiple JOIN statements with the same Join key are merged for execution in the same Fuxi task, such as J4\_1\_2\_3\_Stg1 in the following example.

```

EXPLAIN
SELECT t1. *
FROM t1 JOIN t2 ON t1.c1 = t2.c1
JOIN t3 ON t1.c1 = t3.c1;
```

The earlier version of MaxCompute has the following physical execution plan:

```

In Job job0:
root Tasks: M1_Stg1, M2_Stg1, M3_Stg1
J4_1_2_3_Stg1 depends on: M1_Stg1, M2_Stg1, M3_Stg1

In Task M1_Stg1:
    Data source: meta_dev.t1

In Task M2_Stg1:
    Data source: meta_dev.t2

In Task M3_Stg1:
    Data source: meta_dev.t3

In Task J4_1_2_3_Stg1:
    JOIN: t1 INNER JOIN unknown INNER JOIN unknown
        SEL: t1._col0, t1._col1, t1._col2
            FS: output: None
```

The earlier version of MaxCompute still keeps the physical execution plan when MapJoin Hints are added, and gives priority to applications in Multi-way Join optimization. It may ignore the user-specified MapJoin Hint.

```

EXPLAIN
SELECT /*+mapjoin(t1)*/ t1. *
FROM t1 JOIN t2 ON t1.c1 = t2.c1
JOIN t3 ON t1.c1 = t3.c1;
```

The physical execution plan of the earlier version of MaxCompute is the same as the preceding one.

The optimizer of MaxCompute 2.0 gives priority to the user-specified MapJoin Hint. In the preceding example, if the value of t1 is relatively higher, the following error is returned:

```
FAILED: ODPS-0010000:System internal **error** - SQL Runtime Internal Error: Hash Join Cursor HashJoin_REL… small **table** exceeds, **memory** limit(MB) 640, fixed **memory** used …, variable **memory** used …
```

We recommend that you remove the MapJoin Hint if MapJoin is not your expected behavior.

## sigkill.oom

Like small.table.exceeds.mem.limit, if you specify a MapJoin Hint and your small tables are of a relatively larger size, in the earlier version of MaxCompute, multiple JOIN statements may be optimized by Multi-way Join and can be successfully executed. However, in MaxCompute 2.0, some users may set odps.sql.mapjoin.memory.max to prevent small tables from exceeding the size limit. Each MaxCompute worker has a fixed memory limit. If small tables are of a large size, MaxCompute workers may be killed because the memory limit exceeds. The error is similar to the following:

```
Fuxi job failed - WorkerRestart errCode:9,errMsg:SigKill(OOM), usually caused by OOM(**out****of** memory).
```

We recommend that you remove MapJoin Hint and use Multi-way Join.

## wm\_concat.first.argument.const

According to the [Aggregate function](/intl.en-US/Development/SQL/Builtin functions/Aggregate functions.md) document, the first parameter of WM\_CONCAT must be a constant. The old version of MaxCompute does not have strict check standards. For example, when the source table has no data, no error is returned even if the first parameter of WM\_CONCAT is ColumnReference.

```

The function statement is as follows:
string wm_concat(string separator, string str)
Description of parameters:
Separator: String-type constant.  Constants of other types or non-constants can cause exceptions.
```

MaxCompute 2.0 checks the validity of parameters during the planning phase. An error is returned if the first parameter of WM\_CONCAT is not a constant. Examples:

Incorrect syntax:-

```
SELECT wm_concat(value, ',') FROM src GROUP BY value;
```

Error message:

```
FAILED: ODPS-0130071:[0,0] Semantic analysis exception - physical plan generation failed: com.aliyun.odps.lot.cbo.validator.AggregateCallValidator$AggregateCallValidationException: Invalid argument** type **- The first argument of WM_CONCAT must be constant string.
```

## pt.implicit.convertion.failed

srcpt is a partition table with two partitions:

```

CREATE TABLE srcpt(key STRING, value STRING) PARTITIONED BY (pt STRING);
ALTER TABLE srcpt ADD PARTITION (pt='pt1');
ALTER TABLE srcpt ADD PARTITION (pt='pt2');
```

For the preceding SQL statements, the constants of the IN INT type in the pt columns of the String type are converted to the Double type for comparison. Even if odps.sql.udf.strict.mode is set to “true” in the project, the old version of MaxCompute does not return an error and it filters out all pt columns, whereas in MaxCompute 2.0, an error is returned. Examples:

Incorrect syntax:

```
SELECT key FROM srcpt WHERE pt IN (1, 2);
```

Error message:

```
FAILED: ODPS-0130071:[0,0] Semantic analysis exception - physical plan generation failed: java.lang.NumberFormatException: ODPS-0123091:Illegal type cast - In function cast, value 'pt1' cannot be casted from String to Double.
```

We recommend that you avoid comparing String type partition columns and INT-type constants and convert INT-type constants to the String type.

## having.use.select.alias

SQL specifies that the GROUP BY+HAVING clause precedes the SELECT clause. Therefore, the column alias generated by the SELECT clause cannot be used in the HAVING clause. For example:

Incorrect syntax:

```
SELECT id id2 FROM DUAL GROUP BY id HAVING id2 > 0;
```

Error message:

```
FAILED: ODPS-0130071:[1,44] Semantic analysis exception - column id2 cannot be resolvedODPS-0130071:[1,44] Semantic analysis exception - column reference id2 should appear in GROUP BY key
```

id2 is the column alias generated by the SELECT clause and cannot be used in the HAVING clause.

## dynamic.pt.to.static

In MaxCompute 2.0, dynamic partitions may be converted to static ones by the optimizer. For example:

```
INSERT OVERWRITE TABLE srcpt PARTITION(pt) SELECT id, 'pt1' FROM dual;
```

is converted to:

```
INSERT OVERWRITE TABLE srcpt PARTITION(pt='pt1') SELECT id FROM dual;
```

If the specified partition value is invalid \(for example, ‘$\{bizdate\}’ is used\), MaxCompute 2.0 returns an error during syntax check. For more information, For more information, see [Partition](/intl.en-US/Product Introduction/Definitions/Partition.md).

Incorrect syntax:

```
INSERT OVERWRITE TABLE srcpt PARTITION(pt) SELECT id, '${bizdate}' FROM dual LIMIT 0;
```

Error message:

```
FAILED: ODPS-0130071:[1,24] Semantic analysis exception - wrong columns count 2 in data source, requires 3 columns (includes dynamic partitions if any)
```

In the earlier version of MaxCompute, with LIMIT 0, no results are returned by the SQL statements, and no dynamic partitions are created. In this case, no error is returned.

## lot.not.in.subquery

**Processing of the null value of In subquery.**

In the standard SQL IN operation, if the value list contains a null value, the returned value may be “null” or “true”, but cannot be “false”. For example, 1 in \(null, 1, 2, 3\) is “true”, whereas 1 in \(null, 2, 3\) is “null”, and null in \(null, 1, 2, 3\) is “null”. Likewise, for the NOT IN operation, if the value list contains a null value, the returned value may be “false” or “null”, but cannot be “true”.

MaxCompute 2.0 performs processing with a standard behavior. If you receive a notification on this problem, check your queries to determine whether the subqueries in the IN operation have a null value and whether the related behavior meets your expectation. If not, make relevant changes. Examples:

```
select * from t where c not in (select accepted from c_list);
```

Ignore this problem if “accepted” does not have null values. If a null value exists, c not in \(select accepted from c\_list\) returns the value “true” in the earlier version, but returns a null value in the new version.

Correct syntax:

```
select * from t where c not in (select accepted from c_list where accepted is not null)
```

