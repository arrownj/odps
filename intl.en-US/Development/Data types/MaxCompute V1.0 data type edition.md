---
keyword: MaxCompute V1.0 data type edition
---

# MaxCompute V1.0 data type edition

The MaxCompute V1.0 data type edition is one of the three data type editions of MaxCompute. The MaxCompute V1.0 data type edition supports only MaxCompute V1.0 data types. This topic introduces the MaxCompute V1.0 data type edition and provides the set up methods, supported data types, and differences with other data type editions.

## Definition

If you use the MaxCompute V1.0 data type edition in your project, specify the following information to define the data types:

```
setproject odps.sql.type.system.odps2=false; -- Disables the MaxCompute V2.0 data types.
setproject odps.sql.decimal.odps2=false; -- Disables the DECIMAL type in MaxCompute V2.0.
setproject odps.sql.hive.compatible=false; -- Disables Hive-compatible data types.
```

## Scenarios

The MaxCompute V1.0 data type edition is suitable for projects in early versions of MaxCompute. These projects contain dependent components that do not support the MaxCompute V2.0 data type edition.

## Data types

|Data type|Constant|Description|
|---------|--------|-----------|
|BIGINT|100000000000 L and -1 L|The 64-bit signed integer type.Valid values: -263 + 1 to 263 - 1. |
|DOUBLE|3.14159261E+7|The 64-bit binary floating-point type.|
|DECIMAL|3.5BD and 99999999999.9999999BD|The precise numeric type based on the decimal system.Valid values of the integer part: -1036 + 1 to 1036 - 1. The decimal part is accurate to 10-18. The value is fixed at 54 digits. The integer part consists of 36 digits. The decimal part consists of 18 digits. |
|STRING|"abc", 'bcd', "alibaba", and 'inc'|The string type.The maximum length is 8 MB. |
|DATETIME|DATETIME '2017-11-11 00:00:00'|The date and time type.Valid values: 0000-01-01 to 9999-12-31. |
|BOOLEAN|True and False|The BOOLEAN type.Valid values: True and False. |

The following notes describe the data types:

-   All data types support NULL values.
-   By default, an integer constant is processed as the BIGINT type. If a constant exceeds the value range of the BIGINT type, for example, 1,000,000,000,000,000,000,000,000, the constant is processed as the DOUBLE type. For example, the integer constant 1 in `SELECT 1 + a;` is processed as the BIGINT type.
-   If the MaxCompute V1.0 data type edition is used, built-in functions that contain parameters of the MaxCompute V2.0 data types cannot be used.
-   Partition key columns of partitioned tables must be of the STRING type.
-   Constants of the STRING type can be concatenated. For example, abc and xyz can be concatenated as abcxyz.
-   If a constant is inserted into a field of the DECIMAL type, the expression of the constant must conform to the format in the constant definition. For example, `3.5BD` in the following sample code:

    ```
    INSERT INTO test_tb(a) VALUES (3.5BD);
    ```

-   Time values of the DATETIME type do not include the millisecond component. You can add `-dfp` in Tunnel commands to specify the date format to display milliseconds in time values. For example, `tunnel upload -dfp 'yyyy-MM-dd HH:mm:ss.SSS'`. For more information about Tunnel commands, see [Tunnel commands](/intl.en-US/Development/Data upload and download/Run Tunnel commands to upload and download data/Tunnel commands.md).

## Differences between the MaxCompute V1.0 data type edition and other data type editions

-   The LIMIT, ORDER BY, DISTRIBUTE BY, SORT BY, and CLUSTER BY operations are different between the MaxCompute V1.0 data type edition and the MaxCompute V2.0 data type edition.

    In this example, the `SELECT * FROM t1 UNION ALL SELECT * FROM t2 LIMIT10;` statement is used.

    -   If the MaxCompute V1.0 data type edition is used, the statement is expressed as `SELECT * FROM t1 UNION ALL SELECT * FROM (SELECT * FROM t2 LIMIT 10) t2;`.
    -   If the MaxCompute V2.0 data type edition is used, the statement is expressed as `SELECT * FROM (SELECT * FROM t1 UNION ALL SELECT * FROM t2) t LIMIT 10;`.
-   The IN expressions are different between the MaxCompute V1.0 data type edition and the MaxCompute V2.0 data type edition.

    In this example, the `a IN (1, 2, 3)` expression is used.

    -   If the MaxCompute V1.0 data type edition is used, all values within the parentheses \(\) must be of the same type.
    -   If the MaxCompute V2.0 data type edition is used, all values within the parentheses \(\) can be implicitly converted to the same type.

## Complex data types

|Data type|Definition|Constructor|
|:--------|:---------|:----------|
|ARRAY|-   `ARRAY<BIGINT>`
-   `ARRAY<STRUCT<a:BIGINT, b:STRING>>`

|-   `ARRAY(1, 2, 3)`
-   `ARRAY(ARRAY(1, 2), ARRAY(3, 4))` |
|MAP|-   `MAP<STRING, STRING>`
-   `MAP<BIGINT, ARRAY<STRING>>`

|-   `MAP("k1", "v1", "k2", "v2")`
-   `MAP(1S, ARRAY('a', 'b'), 2S, ARRAY('x', 'y'))` |
|STRUCT|-   `STRUCT<'x', BIGINT, 'y', BIGINT>`
-   `STRUCT<'field1', BIGINT, 'field2', ARRAY<BIGINT>, 'field3', MAP<BIGINT>>`

|-   `NAMED_STRUCT('x', 1, 'y', 2)`
-   `NAMED_STRUCT('field1', 100L, 'field2', ARRAY(1, 2), 'field3', MAP(1, 100, 2, 200))` |

**Note:** In MaxCompute, complex data types can be nested. For more information about the related built-in functions, see [ARRAY](/intl.en-US/Development/SQL/Builtin functions/Other functions.md), [MAP](/intl.en-US/Development/SQL/Builtin functions/Other functions.md), or [STRUCT](/intl.en-US/Development/SQL/Builtin functions/Other functions.md).

