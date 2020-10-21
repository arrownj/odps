---
keyword: [other functions, DECODE, CAST, DECODE]
---

# Other functions

This topic describes how to use functions such as CAST, DECODE, LEAST, ARRAY, SPLIT, and MAP.

## CAST

Function declaration:

```
cast(expr as <type>)
```

Description: converts the result of an expression to another data type. For example, `cast('1' as bigint)` converts `'1'` of the STRING type to `1` of the INTEGER type. If the conversion fails, an error is returned.

Usage:

-   `cast(double as bigint)`: converts a value of the DOUBLE type to the BIGINT type.
-   `cast(string as bigint)`: converts a value of the STRING type to the BIGINT type. If the string is comprised of numerals expressed in the INTEGER form, it is converted to the BIGINT type. If the string is comprised of numerals expressed in the FLOAT or EXPONENTIAL form, it is converted to the DOUBLE type and then to the BIGINT type.
-   The date format for `cast(string as datetime)` and `cast(datetime as string)` is yyyy-mm-dd hh:mi:ss.

## COALESCE

Function declaration:

```
coalesce(expr1, expr2, ...)
```

Description: returns the first non-NULL value in the list. If all values in the list are NULL, NULL is returned.

Parameters:

-   `expr`: the value you want to test. All the values must be of the same data type or be NULL. Otherwise, an error is returned.
-   One or more parameters are required. Otherwise, an error is returned.

Return value: The type of the return value must be the same as that of the input parameter.

## DECODE

Function declaration:

```
decode(expression, search, result[, search, result]...[, default])
```

Description: implements the `IF-THEN-ELSE` conditional branching feature.

Parameters:

-   `expression`: the expression that you want to compare.
-   `search`: the search item that you want to compare with `expression`.
-   `result`: the value returned if the values of `search` and `expression` match.
-   `default`: the value returned if no search item matches the expression. If `default` is not specified, NULL is returned. This parameter is optional.

Return value:

-   The matched `search` value is returned.
-   If no matched item exists, `default` is returned.
-   If `default` is not specified, NULL is returned.

    **Note:**

    -   Three or more parameters are specified.
    -   The values of the `result` parameter must be of the same type or be NULL. Different data types cause an error. The values of `search` and `expression` must be of the same type. Otherwise, an error is returned.
    -   If `search` in the `DECODE` function has duplicated values that can match the expression, the first mapping result is returned.

Example

```
select
decode(customer_id,
1, 'Taobao',
2, 'Alipay',
3, 'Aliyun',
Null, 'N/A',
'Others') as result
from sale_detail;
```

In this example, the `DECODE` function implements the `IF-THEN-ELSE` statement.

```
if customer_id = 1 then
result := 'Taobao';
elsif customer_id = 2 then
result := 'Alipay';
elsif customer_id = 3 then
result := 'Aliyun';
...
else
result := 'Others';
end if;
```

**Note:**

-   In most cases, if "NULL = NULL" appears during the computing, the result returned by the MaxCompute SQL engine is NULL, whereas the `DECODE` function considers that the two NULL values are the same.
-   In this example, if the value of `customer_id` is NULL, the `DECODE` function returns N/A.

## GET\_IDCARD\_AGE

Function declaration:

```
get_idcard_age(idcardno)
```

Description: returns an age based on the ID card number. The return value is the difference between the current year and the birth year identified on the ID card.

Parameters

`idcardno`: the ID card number of the STRING type. The value is a 15-digit or 18-digit number. During calculation, the validity of the ID card is checked based on the province code and the last digit of the ID card. NULL is returned if the check fails.

Return value: It is of the BIGINT type. If the input value is NULL, NULL is returned.

## GET\_IDCARD\_BIRTHDAY

Function declaration:

```
get_idcard_birthday(idcardno)
```

Description: returns the date of birth based on the ID card number.

Parameters:

`idcardno`: the ID card number of the STRING type. The value is a 15-digit or 18-digit number. During calculation, the validity of the ID card is checked based on the province code and the last digit of the ID card. NULL is returned if the check fails.

Return value: It is of the DATETIME type. If the input value is NULL, NULL is returned.

## GET\_IDCARD\_SEX

Function declaration:

```
get_idcard_sex(idcardno)
```

Description: returns the gender based on the ID card number. The return value can be `M` \(male\) or `F` \(female\).

Parameters:

`idcardno`: the ID card number of the STRING type. The value is a 15-digit or 18-digit number. During calculation, the validity of the ID card is checked based on the province code and the last digit of the ID card. NULL is returned if the check fails.

Return value: It is of the STRING type. If the input value is NULL, NULL is returned.

## GREATEST

Function declaration:

```
greatest(var1, var2, ...)
```

Description: returns the maximum value of the input parameters.

Parameters: `var1` and `var2` are input values. They can be of the BIGINT, DOUBLE, DECIMAL, DATETIME, or STRING type. If the values of all input parameters are NULL, NULL is returned.

Return value:

-   The maximum value of the input parameters is returned. If no implicit conversion is required, the return value is of the same type as the input parameter.
-   NULL is interpreted as the minimum value.
-   If the input parameters are of different types, those of the DOUBLE, BIGINT, DECIMAL, and STRING types need to be converted to the DOUBLE type for comparison, and those of the STRING and DATETIME types need to be converted to the DATETIME type for comparison. Implicit conversions of other types are not allowed.
-   If `set odps.sql.hive.compatible` is set to true and the value of any input parameter is NULL, NULL is returned.

## ORDINAL

Function declaration:

```
ordinal(BIGINT nth, var1, var2, ...)
```

Description: sorts the input variables in ascending order, and returns the value in the `nth` bit.

Parameters:

-   `nth`: the position of the value to be returned. The value is of the BIGINT type. If it is set to NULL, NULL is returned.
-   `var1` and `var2`: the input variables. They can be of the BIGINT, DOUBLE, DATETIME, or STRING type.

Return value:

-   The value in the `nth` bit is returned. If no implicit conversion is required, the return value is of the same type as the input parameter.
-   If a data type conversion is performed among the DOUBLE, BIGINT, and STRING types, a value of the DOUBLE type is returned. If a data type conversion is performed between the STRING and DATETIME types, a value of the DATETIME type is returned. Implicit conversions of other types are not allowed.
-   NULL is interpreted as the minimum value.

Example:

```
ordinal(3, 1, 3, 2, 5, 2, 4, 6) = 2
```

## PARTITION\_EXISTS

Function declaration:

```
boolean partition_exists(string table_name, string... partitions)
```

Description: queries whether a specific partition exists.

Parameters:

-   table\_name: the name of the table that you want to query. It is of the STRING type. You can specify a project name in the table name, for example, `my_proj.my_table`. If you do not specify a project name, the current project name is used by default.
-   partitions: the name of the partition that you want to query. It is of the STRING type. The value of this parameter is a list of partition keys based on the sequence of partition key columns in the table. The numbers of partition keys and partition key columns must be the same.

Return value: It is of the BOOLEAN type. If the specified partition exists, True is returned. Otherwise, False is returned.

Examples:

```
-- Create partitioned table foo.
CREATE TABLE foo (id BIGINT) PARTITIONED BY (ds STRING, hr STRING);
-- Add a partition to foo.
ALTER TABLE foo ADD PARTITION (ds='20190101', hr='1');
-- Check whether partitions ds='20190101' and hr='1' exist.
SELECT partition_exists('foo', '20190101', '1');
```

## LEAST

Function declaration:

```
least(var1, var2, ...)
```

Description: returns the minimum value of the input parameters.

Parameters: `var1` and `var2` are input values. They can be of the BIGINT, DOUBLE, DECIMAL, DATETIME, or STRING type. If the values of all input parameters are NULL, NULL is returned.

Return value:

-   The minimum value of the input parameters is returned. If no implicit conversion is required, the return value is of the same type as the input parameter.
-   If a data type conversion is performed among the DOUBLE, BIGINT, and STRING types, a value of the DOUBLE type is returned. If a data type conversion is performed between the STRING and DATETIME types, a value of the DATETIME type is returned. If a data type conversion is performed among the DECIMAL, DOUBLE, BIGINT, and STRING types, a value of the DECIMAL type is returned. Implicit conversions of other types are not allowed.
-   NULL is interpreted as the minimum value.

## MAX\_PT

Function declaration:

```
max_pt(table_full_name)
```

Description: returns the maximum value of each first-level partition of a partitioned table in alphabetic order. The first-level partitions have data files.

Parameter:

`table_full_name`: the table name that has a project name, such as prj.src. It is of the STRING type. You must have the permissions to read the table.

Return value: The maximum value of each first-level partition is returned.

Example:

`tbl` is a partitioned table that has the following partitions, and all the partitions have data files.

```
pt ='20120901'
pt ='20120902'
```

If you execute the following statement, the return value of `max_pt` is `'20120902'`, and the MaxCompute SQL statement reads data from the `pt='20120902'` partition:

```
select * from tbl where pt=max_pt('myproject.tbl');
```

**Note:**

If a partition is added by using the `ALTER TABLE` statement and contains no data file, this partition is not returned.

## UUID

Function declaration:

```
string TRING uuid()
```

Description: returns a random ID, for example, `29347a88-1e57-41ae-bb68-a9edbdd94212`.

**Note:** The return value of this function is a random global ID that has a low probability of duplication.

## SAMPLE

Function declaration:

```
boolean sample(x, y, column_name1,column_name2,...)
```

Description: samples all values read from `column_name` based on `x` and `y`, and filter out the rows that do not meet the sampling condition.

Parameters:

-   `x` and `y`: integer constants that are greater than 0. Their values are of the BIGINT type. They indicate that the values fall into `x` portions by calling the hash function and the `yth` portion is used.

    If `y` is not specified, the first portion is used, and you must not `specify column_name`.````

    An error is returned if `x` and `y` are of another type, their values are less than or equal to 0, or y is greater than x.`` If the value of `x` or `y` is NULL, NULL is returned.

-   `column_name`: the column you want to sample. This parameter is optional. If it is not specified, random sampling is performed based on the values of `x` and `y`. It can be of any type, and the column value can be NULL. No implicit conversion is performed. If `column_name` is the constant NULL, an error is returned.

Return value: It is of the BOOLEAN type.

**Note:** To avoid data skew due to the NULL value, uniform hashing is performed on the NULL values in `column_name` in `x` portions. If `column_name` is not specified and the data volume is small, the output is not necessarily uniform. In this case, we recommend that you specify `column_name` to obtain a uniform output.

Example:

Assume that the `tbla` table exists and has the `cola` column.

```
select * from tbla where sample (4, 1 , cola) = true;
-- The values in the cola column fall into four portions by calling the hash function, and the first portion is used.
select * from tbla where sample (4, 2) = true;
-- The values in each row are randomly hashed to four portions, and the second portion is used.
```

## CASE WHEN expressions

MaxCompute provides the following `CASE WHEN` syntax:

-   ```
case value
when value1 then result1
when value2 then result2
...
else resultn
end
```

-   ```
case
when (_condition1) then result1
when (_condition2) then result2
when (_condition3) then result3
...
else resultn
end
```


The `CASE WHEN` expression can return different values based on the result of the `VALUE` expression. The following example shows that different regions are obtained based on the value of `shop_name`:

```
select
case
when shop_name is null then 'default_region'
when shop_name like 'hang%' then 'zj_region'
end as region
from sale_detail;
```

**Note:**

-   If the types of all `result` values are only BIGINT and DOUBLE, the types are converted to the DOUBLE type and then values are returned.
-   If `result` values of the STRING type exist, the types are converted to the STRING type and then values are returned. If the conversion fails, for example, the conversion from the BOOLEAN type to the STRING type, an error is returned.
-   Conversions between other types are not allowed.

## IF expressions

Syntax:

```
if(testCondition, valueTrue, valueFalseOrNull)
```

Description: determines whether `testCondition` is true. If it is true, `valueTrue` is returned. Otherwise, `valueFalse` or NULL is returned.

Parameters:

-   `testCondition`: the expression that you want to check, which is of the BOOLEAN type.
-   `valueTrue`: the value returned if `testCondition` is true.
-   `valueFalseOrNull`: the value returned if `testCondition` is false. It can be set to NULL.

Return value: The type of the return value is the same as that of `valueTrue` or `valueFalseOrNull`.

Example:

```
select if(1=2,100,200) from dual; 
-- The following result is returned:
+------------+
| _c0 |
+------------+
| 200 |
+------------+
```

## SPLIT

Function declaration:

```
split(str, pat)
```

Description: returns an array after `str` is split with `pat`.

Parameters:

-   `str`: the string that you want to split, which is of the STRING type.
-   `pat`: the delimiter of the STRING type, which supports regular expressions.

Return value: It is in the `array <STRING>` format, and contains `str` after being split with `pat`.

Example:

```
select split("a,b,c",",") from dual;
-- The following result is returned:
+------+
| _c0 |
+------+
| [a, b, c] |
+------+
```

## STR\_TO\_MAP

Function declaration:

```
str_to_map(text [, delimiter1 [, delimiter2]])
```

Description: splits `text` into key-value pairs by using `delimiter1` and then separate keys from values in the key-values pairs by using `delimiter2`.

Parameters:

-   `text`: the string that you want to split, which is of the STRING type.
-   `delimiter1`: the delimiter used to split a string, which is of the STRING type. If it is not specified, a comma \(`,`\) is used by default.
-   `delimiter2`: the delimiter used to separate keys from values, which is of the STRING type. If it is not specified, a period \(`.`\) is used by default.

Return value: It is in the MAP<STRING, STRING\> format. The return value indicates that text is split by using delimiter1, and then by using delimiter2.

Example:

```
select str_to_map('test1&1-test2&2','-','&');
-- The following result is returned:
+------------+
| _c0        |
+------------+
| {"test2":"2","test1":"1"} |
+------------+
```

## EXPLODE

Function declaration:

```
explode (var)
```

Description: transposes one row into multiple rows.

-   If the parameter is of the ARRAY <T\> type, the data of the ARRAY type stored in a column is converted to multiple-row data.
-   If the parameter is of the MAP<K, V\> type, each key-value pair of the MAP type stored in a column is transposed to a row with two columns, one for the key and the other for the value.

Parameters:

`var`: It can be of the ARRAY<T\> or MAP<K, V\> type.

Return value: Rows after the conversion are returned.

**Note:**

Limits:

-   A `SELECT` statement can only contain one EXPLODE function, and no other columns of a table are allowed.
-   This function cannot be used with the `GROUP BY`, `CLUSTER BY`, `DISTRIBUTE BY`, or `SORT BY` clause.

Example:

```
explode(array(null, 'a', 'b', 'c')) col
```

## MAP

Function declaration:

```
MAP map(K key1, V value1, K key2, V value2, ...)
```

Description: creates mappings by using the given key-value pairs.

Parameters:

-   All `key` types, such as those after implicit conversions, must be the same and be of basic data types.
-   All `value` types, such as those after implicit conversions, must be the same and can be of any data types.

Return value: It is of the MAP type.

Example:

When the field in the `t_table` table is `c1 BIGINT, c2 STRING, c3 STRING, c4 BIGINT, c5 BIGINT`, the following data is displayed:

```
+------------+----+----+------------+------------+
| c1         | c2 | c3 | c4         | c5         |
+------------+----+----+------------+------------+
| 1000       | k11 | k21 | 86         | 15         |
| 1001       | k12 | k22 | 97         | 2          |
| 1002       | k13 | k23 | 99         | 1          |
+------------+----+----+------------+------------+
```

Execute the following statement:

```
select  map(c2,c4,c3,c5) from t_table;
-- The following result is returned:
+------+
| _c0  |
+------+
| {k11:86, k21:15} |
| {k12:97, k22:2} |
| {k13:99, k23:1} |
+------+
```

**Note:** If a new data type, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, or BINARY, needs to be used in a MaxCompute SQL statement, you must insert a `SET` statement before the SQL statement to enable the new data type.

-   Session level: To use a new data type, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement, and commit and execute them together.
-   Project-level: MaxCompute allows you to enable new data types for a project. The project owner can run the following command to configure a project:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `setproject`, see [Other operations](/intl.en-US/Development/Common commands/Other operations.md). For the precautions you must take when you enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## MAP\_KEYS

Function declaration:

```
ARRAY map_keys(map<K, V>)
```

Description: returns all keys in the map parameter as an array.

Parameter: The parameter value is of the MAP type.

Return value: It is of the ARRAY type. If the input value is NULL, NULL is returned.

Example:

When the field in the `t_table_map` table is `c1 BIGINT,t_map MAP<STRING,BIGINT>`, the following data is displayed:

```
+------------+-------+
| c1         | t_map |
+------------+-------+
| 1000       | {k11:86, k21:15} |
| 1001       | {k12:97, k22:2} |
| 1002       | {k13:99, k23:1} |
+------------+-------+
```

Execute the following statement:

```
select  c1,map_keys(t_map) from t_table_map;
-- The following result is returned:
+------------+------+
| c1         | _c1  |
+------------+------+
| 1000       | [k11, k21] |
| 1001       | [k12, k22] |
| 1002       | [k13, k23] |
+------------+------+
```

## MAP\_VALUES

Function declaration:

```
ARRAY map_values(map<K, V>)
```

Description: returns all values in the map parameter as an array.

Parameter: The parameter value is of the MAP type.

Return value: It is of the ARRAY type. If the input value is NULL, NULL is returned.

Example:

```
select map_values(map('a',123,'b',456));
-- The following result is returned:
[123, 456]
```

## ARRAY

Function declaration:

```
ARRAY array(value1,value2, ...)
```

Description: creates an array by using input values.

Parameters: The parameters can be of any type, but the types of all parameters must be the same.

Return value: It is of the ARRAY type.

Example:

When the field in the `t_table` table is `c1 BIGINT, c2 STRING, c3 STRING, c4 BIGINT, c5 BIGINT`, the following data is displayed:

```
+------------+----+----+------------+------------+
| c1         | c2 | c3 | c4         | c5         |
+------------+----+----+------------+------------+
| 1000       | k11 | k21 | 86         | 15         |
| 1001       | k12 | k22 | 97         | 2          |
| 1002       | k13 | k23 | 99         | 1          |
+------------+----+----+------------+------------+
```

Execute the following statement:

```
select array(c2,c4,c3,c5) from t_table;
-- The following result is returned:
+------+
| _c0  |
+------+
| [k11, 86, k21, 15] |
| [k12, 97, k22, 2] |
| [k13, 99, k23, 1] |
+------+
```

**Note:** If a new data type, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, or BINARY, needs to be used in a MaxCompute SQL statement, you must insert a `SET` statement before the SQL statement to enable the new data type.

-   Session level: To use a new data type, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement, and commit and execute them together.
-   Project-level: MaxCompute allows you to enable new data types for a project. The project owner can run the following command to configure a project:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `setproject`, see [Other operations](/intl.en-US/Development/Common commands/Other operations.md). For the precautions you must take when you enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## SIZE

Function declaration:

```
INT size(map)
INT size(array)
```

Description: `size(map)` is used to return the number of key-value pairs in the specified `map` parameter. `size(array)` is used to return the number of elements in the specified `array` parameter.

Parameters:

-   `map`: the data of the MAP type.
-   `array`: the data of the ARRAY type.

Return value: It is of the INT type.

Example:

```
select size(map('a',123,'b',456)) from dual; -- Value 2 is returned.
select size(map('a',123,'b',456,'c',789)) from dual; -- Value 3 is returned.
select size(array('a','b')) from dual; -- Value 2 is returned.
select size(array(123,456,789)) from dual; -- Value 3 is returned.
```

## ARRAY\_CONTAINS

Function declaration:

```
boolean array_contains(ARRAY<T> a,value v)
```

Description: checks whether array `a` contains value `v`.

Parameters:

-   `a`: the data of the ARRAY type.
-   `v`: The value must be of the same type as the data type in the array.

Return value: It is of the BOOLEAN type.

Example:

When the field in the `t_table_array` table is `c1 BIGINT, t_array ARRAY<STRING>`, the following data is displayed:

```
+------------+---------+
| c1         | t_array |
+------------+---------+
| 1000       | [k11, 86, k21, 15] |
| 1001       | [k12, 97, k22, 2] |
| 1002       | [k13, 99, k23, 1] |
+------------+---------+
```

Execute the following statement:

```
select c1, array_contains(t_array,'1') from t_table_array;
-- The following result is returned:
+------------+------+
| c1         | _c1  |
+------------+------+
| 1000       | false |
| 1001       | false |
| 1002       | true |
+------------+------+
```

## SORT\_ARRAY

Function declaration:

```
ARRAY sort_array(ARRAY<T>)
```

Description: sorts a given array.

Parameter:

`ARRAY<T>`: the data of the ARRAY type. Data in the array can be of any type.

Return value: It is of the ARRAY type.

Example:

When the field in the `t_array` table is `c1 ARRAY<STRING>,c2 ARRAY<INT> ,c3 ARRAY<STRING>`, the following data is displayed:

```
+------------+---------+--------------+
| c1         | c2      | c3           |
+------------+---------+--------------+
| [a, c, f, b]  | [4, 5, 7, 2, 5, 8]  |  [You, Me, Him] |
+------------+---------+--------------+
```

Execute the following statement:

```
select sort_array(c1),sort_array(c2),sort_array(c3) from t_array;
-- The following result is returned:
[a, b, c, f] [2, 4, 5, 5, 7, 8] [Him, You, Me]
```

**Note:** If a new data type, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, or BINARY, needs to be used in a MaxCompute SQL statement, you must insert a `SET` statement before the SQL statement to enable the new data type.

-   Session level: To use a new data type, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement, and commit and execute them together.
-   Project level: You can enable a new data type at the project level. The project owner can run the following command to configure a project:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `setproject`, see [Other operations](/intl.en-US/Development/Common commands/Other operations.md). For the precautions you must take when you enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## POSEXPLODE

Function declaration:

```
posexplode(ARRAY<T>)
```

Description: converts the array into a table that has two columns. The first column lists subscripts of each value in the array, starting from 0, and the second column lists array elements.

Parameters

`ARRAY<T>`: the data of the ARRAY type. Data in the array can be of any type.

Return value: The generated table is returned.

Example:

```
select posexplode(array('a','c','f','b')) from dual;
-- The following result is returned:
+------------+-----+
| pos | val |
+------------+-----+
| 0 | a |
| 1 | c |
| 2 | f |
| 3 | b |
+------------+-----+
```

**Note:** If a new data type, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, or BINARY, needs to be used in a MaxCompute SQL statement, you must insert a `SET` statement before the SQL statement to enable the new data type.

-   Session level: To use a new data type, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement, and commit and execute them together.
-   Project-level: MaxCompute allows you to enable new data types for a project. The project owner can run the following command to configure a project:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `setproject`, see [Other operations](/intl.en-US/Development/Common commands/Other operations.md). For the precautions you must take when you enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## STRUCT

Function declaration:

```
STRUCT struct(value1,value2, ...)
```

Description: creates a struct based on the given `value` list.

Parameters

`value`: It can be of any type.``

Return value: It is of the STRUCT type. The fields are consecutively named, for example, `col1, col2, ...`.

Example:

```
select struct('a',123,'ture',56.90) from dual;
-- The following result is returned:
{col1:a, col2:123, col3:ture, col4:56.9}
```

**Note:** If a new data type, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, or BINARY, needs to be used in a MaxCompute SQL statement, you must insert a `SET` statement before the SQL statement to enable the new data type.

-   Session level: To use a new data type, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement, and commit and execute them together.
-   Project-level: MaxCompute allows you to enable new data types for a project. The project owner can run the following command to configure a project:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `setproject`, see [Other operations](/intl.en-US/Development/Common commands/Other operations.md). For the precautions you must take when you enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## NAMED\_STRUCT

Function declaration:

```
STRUCT named_struct(string name1, T1 value1, string name2, T2 value2, ...)
```

Description: creates a struct based on the given `name/value` list.

Parameters:

-   `value`: It can be of any type.``
-   `name`: the name of the field of the STRING type. This parameter is a constant.

Return value: It is of the STRUCT type. The fields are consecutively named, for example, `name1, name2, ...`.

Example:

```
select named_struct('user_id',10001,'user_name','LiLei','married','F','weight',63.50) from dual;
-- The following result is returned:
{user_id:10001, user_name:LiLei, married:F, weight:63.5}
```

**Note:** If a new data type, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, or BINARY, needs to be used in a MaxCompute SQL statement, you must insert a `SET` statement before the SQL statement to enable the new data type.

-   Session level: To use a new data type, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement, and commit and execute them together.
-   Project-level: MaxCompute allows you to enable new data types for a project. The project owner can run the following command to configure a project:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `setproject`, see [Other operations](/intl.en-US/Development/Common commands/Other operations.md). For the precautions you must take when you enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## INLINE

Function declaration:

```
inline(array<struct<f1:T1, f2:T2, ... >>)
```

Description: expands a given struct array with each array element that corresponds to a row and each struct element corresponds to a column in each row.

Parameters:

`struct`: The values in the array can be of any type.``

Return value: The generated table is returned.

Example:

When the field in the `t_table` table is `t_struct struct<user_id:bigint,user_name:string,married:string,weight:double>`, the following data is displayed:

```
+----------+
| t_struct |
+----------+
| {user_id:10001, user_name:LiLei, married:N, weight:63.5} |
| {user_id:10002, user_name:HanMeiMei, married:Y, weight:43.5} |
+----------+
```

Execute the following statement:

```
select inline(array(t_struct)) from t_table;
-- The following result is returned:
+------------+-----------+---------+------------+
| user_id    | user_name | married | weight     |
+------------+-----------+---------+------------+
| 10001      | LiLei     | N       | 63.5       |
| 10002      | HanMeiMei | Y       | 43.5       |
+------------+-----------+---------+------------+
```

**Note:** If a new data type, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, or BINARY, needs to be used in a MaxCompute SQL statement, you must insert a `SET` statement before the SQL statement to enable the new data type.

-   Session level: To use a new data type, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement, and commit and execute them together.
-   Project-level: MaxCompute allows you to enable new data types for a project. The project owner can run the following command to configure a project:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `setproject`, see [Other operations](/intl.en-US/Development/Common commands/Other operations.md). For the precautions you must take when you enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## TABLE\_EXISTS

Function declaration:

```
boolean table_exists(string table_name)
```

Description: queries whether a specified table exists.

Parameters:

table\_name: the name of the table that you want to query. It is of the STRING type. You can specify a project name in the table name, for example, `my_proj.my_table`. If you do not specify a project name, the current project name is used by default.

Return value: It is of the BOOLEAN type. If the specified table exists, True is returned. Otherwise, False is returned.

Examples

```
-- Used in the SELECT statement.
SELECT IF(table_exists('abd'), col1, col2) FROM src;
```

## TRANS\_ARRAY

Function declaration:

```
trans_array (num_keys, separator, key1,key2,...,col1, col2,col3) as (key1,key2,...,col1, col2)
```

Description: transposes one row into multiple rows, specifically, transposes an array separated by fixed delimiters in a column into multiple rows.

Parameters:

-   `num_keys`: The value is a constant of the BIGINT type, which must be greater than or equal to 0.`` This parameter specifies the number of columns that can be used as `key` when you transpose one row into multiple rows.
-   `key`: duplicate columns in multiple rows when you transpose one row to multiple rows.
-   `separator`: the separator used to separate a string into multiple elements. It is a constant of the STRING type. If it is left blank, an error is returned.
-   `keys`: the columns that are used as `key` during the transposition. The number of keys is specified by `num_keys`. If `num_keys` equals the total number of all columns \(which means that all columns are used as `key`\), only one row is returned.``
-   `cols`: the array that you want to be converted to rows. All columns following `keys` are considered as arrays to be transposed. The parameter value must be of the STRING type to store arrays in the STRING format, such as `Hangzhou;Beijing;shanghai`. The values in this array are separated with semicolons \(`;`\).

Return value: The rows after the transposition are returned. The names of new columns are specified by `as`. The data types of columns used as the `key` remain unchanged. All other columns are of the STRING type. The number of separated rows depends on the array with the maximum number. If the number of rows is insufficient, NULL is added.

**Note:**

UDTF limits:

-   All columns that are used as `key` must be placed before the columns are to be transposed.
-   Only one UDTF is allowed in a `SELECT` statement.
-   This function cannot be used with the `GROUP BY`, `CLUSTER BY`, `DISTRIBUTE BY`, or `SORT BY` clause.

Example:

-   The `t_table table` lists the following data:

```
+----------+----------+------------+
| login_id | login_ip | login_time |
+----------+----------+------------+
| wangwangA | 192.168.0.1,192.168.0.2 | 20120101010000,20120102010000 |
| wangwangB | 192.168.45.10,192.168.67.22,192,168.6.3 | 20120111010000,20120112010000,20120223080000 |
+----------+----------+------------+
```

    Execute the following statement:

    ```
    select trans_array(1, ",", login_id, login_ip, login_time) as (login_id,login_ip,login_time) from t_table;
    The following result is returned:
    +----------+----------+------------+
    | login_id | login_ip | login_time |
    +----------+----------+------------+
    | wangwangB | 192.168.45.10 | 20120111010000 |
    | wangwangB | 192.168.67.22 | 20120112010000 |
    | wangwangB | 192.168.6.3 | 20120223080000 |
    | wangwangA | 192.168.0.1 | 20120101010000 |
    | wangwangA | 192.168.0.2 | 20120102010000 |
    +----------+----------+------------+
    ```

-   The table lists the following data:

    ```
    Login_id LOGIN_IP LOGIN_TIME 
    wangwangA 192.168.0.1,192.168.0.2 20120101010000
    ```

    NULL is added to complete the array which has insufficient data.

    ```
    Login_id Login_ip Login_time 
    wangwangA 192.168.0.1 20120101010000
    wangwangA 192.168.0.2 NULL
    ```


## TRANS\_COLS

Function declaration:

```
trans_cols (num_keys, key1,key2,...,col1, col2,col3) as (idx, key1,key2,...,col1, col2)
```

Description: transposes one row into multiple rows, specifically, transposes different columns into different rows.

Parameters:

-   `num_keys`: The value is a constant of the BIGINT type, which must be greater than or equal to 0.`` This parameter specifies the number of columns that can be used as `key` when you transpose one row into multiple rows.
-   `key`: duplicate columns in multiple rows when you transpose one row to multiple rows.
-   `separator`: the separator used to separate a string into multiple elements. It is a constant of the STRING type. If it is left blank, an error is returned.
-   `keys`: the columns that are used as `key` during the transposition. The number of keys is specified by `num_keys`. If `num_keys` equals the total number of all columns \(which means that all columns are used as `key`\), only one row is returned.``
-   `cols`: the column you want to convert into a row.

Return value: The rows after the transposition are returned. The names of new columns are specified by `as`. The first output column is the subscripts of the transposition, which start with 1. The data types of the columns used as keys remain unchanged, whereas the data types of other columns are the same as the originals.

**Note:**

UDTF limits:

-   All columns that are used as `key` must be placed before the columns are to be transposed.
-   Only one UDTF is allowed in a `SELECT` statement.

Example:

The `t_table` table lists the following data:

```
+----------+----------+------------+
| Login_id | Login_ip1 | Login_ip2 |
+----------+----------+------------+
| wangwangA | 192.168.0.1 | 192.168.0.2 |
+----------+----------+------------+
```

Execute the following statement:

```
select trans_cols(1, login_id, login_ip1, login_ip2) as (idx, login_id, login_ip) from t_table;
```

The following result is returned:

```
idx    login_id    login_ip
1    wangwangA    192.168.0.1
2    wangwangA    192.168.0.2
```

