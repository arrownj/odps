---
keyword: [relational operators, arithmetic operators, bitwise operators, logical operators]
---

# Operators

Operators are used to perform program code operations. This topic introduces four types of operators: relational operators, arithmetic operators, bitwise operators, and logical operators.

## Relational operators

|Operator|Description|
|:-------|:----------|
|A=B|-   If A or B is NULL, NULL is returned.
-   If A is equal to B, TRUE is returned. Otherwise, FALSE is returned. |
|A<\>B|-   If A or B is NULL, NULL is returned.
-   If A is not equal to B, TRUE is returned. Otherwise, FALSE is returned. |
|A<B|-   If A or B is NULL, NULL is returned.
-   If A is less than B, TRUE is returned. Otherwise, FALSE is returned. |
|A<=B|-   If A or B is NULL, NULL is returned.
-   If A is less than or equal to B, TRUE is returned. Otherwise, FALSE is returned. |
|A\>B|-   If A or B is NULL, NULL is returned.
-   If A is greater than B, TRUE is returned. Otherwise, FALSE is returned. |
|A\>=B|-   If A or B is NULL, NULL is returned.
-   If A is greater than or equal to B, TRUE is returned. Otherwise, FALSE is returned. |
|A IS NULL|If A is NULL, TRUE is returned. Otherwise, FALSE is returned.|
|A IS NOT NULL|If A is not NULL, TRUE is returned. Otherwise, FALSE is returned.|
|A LIKE B|If A or B is NULL, NULL is returned. If string A matches pattern B, TRUE is returned. Otherwise, FALSE is returned.-   The percent sign \(%\) matches an arbitrary number of characters.
-   The underscore \(\_\) matches a single character.
-   To match percent signs \(%\) or underscores \(\_\), escape them with single quotation marks \('\). Specifically, `‘%’` or `‘_’` is used for the match.

```
'aaa' like 'a__'= TRUE 
'aaa' like 'a%' = TRUE
'aaa' like 'aab'= FALSE 
'a%b' like 'a\\%b'= TRUE 
'axb' like 'a\\%b'= FALSE 
``` |
|A RLIKE B|If string A matches string constant or regular expression B, TRUE is returned. Otherwise, FALSE is returned. If B is empty, an error is returned. If A or B is NULL, NULL is returned.|
|A IN B|-   If A is included in set B, TRUE is returned. Otherwise, FALSE is returned.
-   If A is NULL, NULL is returned.
-   If B contains only one NULL element, that is, A IN\(NULL\), NULL is returned.
-   If constant set B contains one or more items, the data types of all items must be the same.

**Note:** If set B contains the NULL element and other elements, the data type of NULL is considered the same as the data types of other elements in set B. |
|BETWEEN AND|The expression is `A [NOT] BETWEEN B AND C`.-   If A, B, or C is NULL, this expression is empty.
-   If A is greater than or equal to B and less than or equal to C, TRUE is returned. Otherwise, FALSE is returned. |
|IS \[NOT\] DISTINCT FROM|The expression is `A IS [NOT] DISTINCT FROM B`. For more information, see [IS DISTINCT FROM](/intl.en-US/Exclusive Mode/Flink SQL/Built-in functions/Logical functions/IS DISTINCT FROM.md) and [IS NOT DISTINCT FROM](/intl.en-US/Exclusive Mode/Flink SQL/Built-in functions/Logical functions/IS NOT DISTINCT FROM.md).|

Common use of relational operators in statements:

```
SELECT * FROM user WHERE user_id = '0001'; 
SELECT * FROM user WHERE user_name <> 'maggie'; 
SELECT * FROM user WHERE age > ‘50’; 
SELECT * FROM user WHERE birth_day >= '1980-01-01 00:00:00'; 
SELECT * FROM user WHERE is_female is null; 
SELECT * FROM user WHERE is_female is not null; 
SELECT * FROM user WHERE user_id in (0001,0010); 
SELECT * FROM user WHERE user_name like 'M%';
```

Before you perform some relational operations, you must convert the data type. Otherwise, NULL may be returned. For more information about the conversion, see [Data type conversion](/intl.en-US/Development/SQL/Data type conversion.md). For example, `'2019-02-16 00:00:01'` is of the DATETIME type and `'2019-02-16'` is of the STRING type. You must explicitly convert them to the same type before you perform relational operations for comparison.

```
SELECT CAST('2019-02-16 00:00:01' AS STRING) > '2019-02-16';
SELECT CAST('2019-02-16 00:00:02' AS DATETIME) > '2019-02-16 00:00:01';
```

Values of the DOUBLE type in MaxCompute are different in precision. For this reason, we recommend that you do not use the equal sign \(=\) for comparison between two values of the DOUBLE type. You can subtract two values of the DOUBLE type, and then obtain the absolute value for comparison. If the absolute value is negligible, the two values of the DOUBLE type are considered equal. Example:

```
ABS(0.9999999999 - 1.0000000000) < 0.000000001
 -- 0.9999999999 and 1.0000000000 have a precision of 10 decimal digits, whereas 0.000000001 has the precision of 9 decimal digits.
 -- 0.9999999999 is considered equal to 1.0000000000.
```

**Note:**

-   ABS is a built-in function provided by MaxCompute. This function is used to obtain the absolute value of its input. For more information, see [ABS](/intl.en-US/Development/SQL/Builtin functions/Mathematical functions.md).
-   In most cases, a value of the DOUBLE type in MaxCompute can provide a precision of 14 decimal digits.
-   If you compare a value of the STRING type with that of the BIGINT type, the values are automatically converted to those of the DOUBLE type. This may cause a loss of precision during the comparison. To address this issue, you can execute CAST STRING AS BIGINT to convert the data type STRING to BIGINT before the comparison.

## Arithmetic operators

|Operator|Description|
|:-------|:----------|
|A+B|If A or B is NULL, NULL is returned. Otherwise, the result of A+B is returned.|
|A-B|If A or B is NULL, NULL is returned. Otherwise, the result of A-B is returned.|
|A\*B|If A or B is NULL, NULL is returned. Otherwise, the result of A\*B is returned.|
|A/B|If A or B is NULL, NULL is returned. Otherwise, the result of A/B is returned.**Note:** If A and B are of the BIGINT type, the return value is of the DOUBLE type. |
|A%B|If A or B is NULL, NULL is returned. Otherwise, the result of A%B is returned. A%B is the remainder result of A/B.|
|+A|A is returned.|
|-A|If A is NULL, NULL is returned. Otherwise, -A is returned.|

Common use of arithmetic operators in statements:

```
SELECT age+10, age-10, age%10, -age, age*age, age/10 FROM user;
```

**Note:**

-   You can use only the values of the STRING, BIGINT, or DOUBLE type to perform arithmetic operations. You cannot use the values of the DATATIME or BOOLEAN type to perform arithmetic operations.
-   Values of the STRING type are implicitly converted to those of the DOUBLE type before arithmetic operations.
-   If you use the values of the BIGINT and DOUBLE types to perform arithmetic operations, the value of the BIGINT type is implicitly converted to that of the DOUBLE type before the operations. The return value is of the DOUBLE type.
-   If both A and B are of the BIGINT type, the return value is of the DOUBLE type after you perform the A/B operation. For other arithmetic operations, the return value is of the BIGINT type.

## Bitwise operators

|Operator|Description|
|:-------|:----------|
|A&B|The bitwise AND result of A and B is returned. For example, the result of 1&2 is 0, the result of 1&3 is 1, and the bitwise AND result of NULL and any value is NULL. Both A and B must be of the BIGINT type.|
|A\|B|The bitwise OR result of A and B is returned. For example, the result of 1\|2 is 3, the result of 1\|3 is 3, and the bitwise OR result of NULL and any value is NULL. Both A and B must be of the BIGINT type.|
|A\|\|B|This operator is used to join strings. For example, `a||b||c` is equivalent to `CONCAT(a, b, c)`.|

**Note:** Bitwise operators do not support implicit type conversion. You can use only the values of the BIGINT type in bitwise operations.

## Logical operators

|Operator|Description|
|--------|-----------|
|A and B|TRUE and TRUE=TRUE|
|TRUE and FALSE=FALSE|
|FALSE and TRUE=FALSE|
|FALSE and NULL=FALSE|
|NULL and FALSE=FALSE|
|TRUE and NULL=NULL|
|NULL and TRUE=NULL|
|NULL and NULL=NULL|
|A or B|TRUE or TRUE=TRUE|
|TRUE or FALSE=TRUE|
|FALSE or TRUE=TRUE|
|FALSE or NULL=NULL|
|NULL or FALSE=NULL|
|TRUE or NULL=TRUE|
|NULL or TRUE=TRUE|
|NULL or NULL=NULL|
|NOT A|If A is NULL, NULL is returned.|
|If A is TRUE, FALSE is returned.|
|If A is FALSE, TRUE is returned.|

**Note:** Logical operators do not support implicit type conversion. You can use only the values of the BOOLEAN type in logical operations.

