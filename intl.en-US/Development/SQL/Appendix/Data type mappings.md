---
keyword: data type mappings
---

# Data type mappings

This topic describes the data type mappings between MaxCompute and the following components: Hive, Oracle, and MySQL.

The following table lists the data type mappings.

|MaxCompute data type|Hive data type|Oracle data type|MySQL data type|
|--------------------|:-------------|----------------|---------------|
|BOOLEAN|BOOLEAN|None**Note:** CHAR\(1\), INTEGER, or NUMBER\(1\) is used instead. The value 1 indicates true, and the value 0 indicates false.

|None**Note:** TINYINT\(1\) is used instead. |
|TINYINT|TINYINT|NUMBER\(3,0\)|TINYINT|
|SMALLINT|SMALLINT|NUMBER\(5,0\)|SMALLINT|
|INT|INT|NUMBER\(7,0\)|MEDIUMINT|
|INT|INT|NUMBER\(10,0\)|INT|
|BIGINT|BIGINT|NUMBER\(20,0\)|BIGINT|
|FLOAT|FLOAT|BINARY\_FLOAT**Note:** This data type is supported in Oracle Database 10g and later.

|FLOAT|
|DOUBLE|DOUBLE|BINARY\_DOUBLE**Note:** This data type is supported in Oracle Database 10g and later.

|DOUBLE|
|DECIMAL|DECIMAL|NUMBER\(P,S\)|-   DECIMAL
-   NUMERIC |
|STRING|STRING|-   VARCHAR
-   VARCHAR2
-   CHAR
-   NCHAR
-   NVARCHAR3

|-   VARCHAR
-   CHAR |
|VARCHAR|VARCHAR|-   VARCHAR
-   VARCHAR2
-   CHAR
-   NCHAR
-   NVARCHAR3

|VARCHAR|
|STRING|CHAR|CHAR|CHAR|
|BINARY|BINARY|RAW|-   BINARY
-   VARBINARY |
|TIMESTAMP|TIMESTAMP|TIMESTAMP\(N\)|TIMESTAMP|
|DATETIME|DATE|DATE|DATETIME|
|ARRAY|ARRAY|Not supported|Not supported|
|MAP|`MAP<key,value>`|Not supported|Not supported|
|STRUCT|STRUCT|Not supported|Not supported|
|Not supported|UNION|Not supported|Not supported|

