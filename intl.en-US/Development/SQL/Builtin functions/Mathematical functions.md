---
keyword: mathematical function
---

# Mathematical functions

MaxCompute SQL provides common mathematical functions that you can use for development. You can select mathematical functions as needed for data computing and data type conversions. This topic describes the syntax, parameters, and examples of mathematical functions supported by MaxCompute SQL.

The following table lists mathematical functions supported by MaxCompute SQL.

|Function|Description|
|--------|-----------|
|[ABS](#section_i1v_5lm_vdb)|Calculates the absolute value.|
|[ACOS](#section_cfp_qmm_vdb)|Calculates the arccosine.|
|[ASIN](#section_fau_d9e_7p5)|Calculates the arcsine.|
|[ATAN](#section_odw_jnm_vdb)|Calculates the arctangent.|
|[CEIL](#section_ugm_k4m_vdb)|Rounds up a number and returns the nearest integer.|
|[CONV](#section_tkx_q4m_vdb)|Converts a number from one number system to another.|
|[COS](#section_tpy_z4m_vdb)|Calculates the cosine.|
|[COSH](#section_tnp_gpm_vdb)|Calculates the hyperbolic cosine.|
|[COT](#section_hhz_lpm_vdb)|Calculates the cotangent.|
|[EXP](#section_q1n_rpm_vdb)|Calculates the exponential value.|
|[FLOOR](#section_yrw_wpm_vdb)|Rounds down a number and returns the nearest integer.|
|[LN](#section_pdm_fqm_vdb)|Calculates the natural logarithm.|
|[LOG](#section_iwc_4qm_vdb)|Calculates the logarithm.|
|[POW](#section_gmv_wqm_vdb)|Calculates the nth power of a value.|
|[RAND](#section_qlv_2rm_vdb)|Returns a random number.|
|[ROUND](#section_ocf_jrm_vdb)|Returns a value rounded to the specified decimal place.|
|[SIN](#section_tky_gvm_vdb)|Calculates the sine.|
|[SINH](#section_ccf_gym_vdb)|Calculates the hyperbolic sine.|
|[SQRT](#section_nns_lym_vdb)|Calculates the square root.|
|[TAN](#section_ibd_rym_vdb)|Calculates the tangent.|
|[TANH](#section_pfh_wym_vdb)|Calculates the hyperbolic tangent.|
|[TRUNC](#section_yly_1zm_vdb)|Truncates the input value of a number and right-fills it with zeros from the specified position.|
|[LOG2](#section_dh3_tzm_vdb)|Calculates the logarithm of a number with the base number of 2.|
|[LOG10](#section_bjc_zzm_vdb)|Calculates the logarithm of a number with the base number of 10.|
|[BIN](#section_ucn_21n_vdb)|Calculates the binary code.|
|[HEX](#section_nxv_j1n_vdb)|Converts an integer or a string into a hexadecimal number.|
|[UNHEX](#section_k5x_51n_vdb)|Converts a hexadecimal string into a string.|
|[RADIANS](#section_dwg_1bn_vdb)|Converts a degree to a radian value.|
|[DEGREES](#section_adl_hbn_vdb)|Converts a radian value to a degree.|
|[SIGN](#section_gq5_lbn_vdb)|Returns the sign of the input value.|
|[E](#section_yfc_zbn_vdb)|Calculates the value of e.|
|[PI](#section_hhc_dcn_vdb)|Calculates the value of π.|
|[FACTORIAL](#section_umk_gcn_vdb)|Calculates the factorial.|
|[CBRT](#section_frl_lcn_vdb)|Calculates the cube root.|
|[SHIFTLEFT](#section_k4z_pcn_vdb)|Shifts a value left by a specific number of places.|
|[SHIFTRIGHT](#section_iyl_vcn_vdb)|Shifts a value right by a specific number of places.|
|[SHIFTRIGHTUNSIGNED](#section_h2f_1dn_vdb)|Shifts an unsigned value right by a specific number of places.|
|[FORMAT\_NUMBER](#section_tpw_bk9_iwh)|Converts a number to a string in the specified format.|
|[WIDTH\_BUCKET](#section_pn8_ilh_3ll)|Returns the ID of the bucket into which the value of a specific expression falls.|

## ABS

-   Syntax

    ```
    DOUBLE ABS(DOUBLE number)
    BIGINT ABS(BIGINT number)
    DECIMAL ABS(DECIMAL number)
    ```

-   Description

    This function calculates the absolute value of number.

-   Parameters

    number: If number is of the DOUBLE, BIGINT, or DECIMAL type, a value of the same type is returned.

    -   If number is of the BIGINT type, the return value must be of the BIGINT type.
    -   If number is of the DOUBLE type, the return value must be of the DOUBLE type.
    -   If number is of the DECIMAL type, the return value must be of the DECIMAL type.
    -   If number is of the STRING type, it is implicitly converted to a value of the DOUBLE type before calculation.
    -   If number is of a data type other than the preceding four types, an error is returned.
    **Note:** If number is of the BIGINT type and is greater than the maximum value of the BIGINT type, a value of the DOUBLE type is returned. However, the precision may be lost.

-   Return value

    The type of the return value depends on that of the input value, which can be DOUBLE, BIGINT, or DECIMAL. If the input value is NULL, NULL is returned.

-   Examples

    ```
    ABS(null)=null
    ABS(-1)=1
    ABS(-1.2)=1.2
    ABS("-2")=2.0
    ABS(122320837456298376592387456923748)=1.2232083745629837e32
    ```

    The following example shows the usage of an ABS function in SQL statements. Other built-in functions, except window functions and aggregate functions, are used in a similar way.

    ```
    -- Calculate the absolute value of the id field in tbl1.
    SELECT ABS(id) FROM tbl1;
    ```


## ACOS

-   Syntax

    ```
    DOUBLE ACOS(DOUBLE number)
    DECIMAL ACOS(DECIMAL number)
    ```

-   Description

    This function calculates the arccosine of number.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. The value ranges from -1 to 1. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. The value ranges from 0 to π. If the input value is NULL, NULL is returned.

-   Examples

    ```
    ACOS("0.87")=0.5155940062460905
    ACOS(0)=1.5707963267948966
    ```


## ASIN

-   Syntax

    ```
    DOUBLE ASIN(DOUBLE number)
    DECIMAL ASIN(DECIMAL number)
    ```

-   Description

    This function calculates the arcsine of number.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. The value ranges from -1 to 1. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. The value ranges from -π/2 to π/2. If the input value is NULL, NULL is returned.

-   Examples

    ```
    ASIN(1)=1.5707963267948966
    ASIN(-1)=-1.5707963267948966
    ```


## ATAN

-   Syntax

    ```
    DOUBLE ATAN(DOUBLE number)
    ```

-   Description

    This function calculates the arctangent of number.

-   Parameters

    number: a value of the DOUBLE type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE type is returned. The value ranges from -π/2 to π/2. If the input value is NULL, NULL is returned.

-   Examples

    ```
    ATAN(1)=0.7853981633974483
    ATAN(-1)=-0.7853981633974483
    ```


## CEIL

-   Syntax

    ```
    BIGINT CEIL(DOUBLE value)
    BIGINT CEIL(DECIMAL value)
    ```

-   Description

    This function rounds up value and returns the nearest integer.

-   Parameters

    value: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the BIGINT type is returned. If the input value is NULL, NULL is returned.

-   Examples

    ```
    CEIL(1.1)=2
    CEIL(-1.1)=-1
    ```


## CONV

-   Syntax

    ```
    STRING CONV(STRING input, BIGINT from_base, BIGINT to_base)
    ```

-   Description

    This function converts a number from one number system to another.

-   Parameters
    -   input: the integer you want to convert, which is of the STRING type. If the input value is of the BIGINT or DOUBLE type, it is implicitly converted to a value of the STRING type before calculation.
    -   from\_base and to\_base: decimal numbers. The values can be 2, 8, 10, or 16. If the input value is of the STRING or DOUBLE type, it is implicitly converted to a value of the BIGINT type before calculation.
-   Return value

    A value of the STRING type is returned. If any input value is NULL, NULL is returned. The conversion process runs at 64-bit precision. If an overflow occurs, an error is returned. If the input value is a negative value that begins with an en dash \(-\), an error is returned. If the input value is a decimal, it is converted to an integer before the conversion of number systems. The decimal part is left out.

-   Examples

    ```
    CONV('1100', 2, 10)='12'
    CONV('1100', 2, 16)='c'
    CONV('ab', 16, 10)='171'
    CONV('ab', 16, 16)='ab'
    ```


## COS

-   Syntax

    ```
    DOUBLE COS(DOUBLE number)
    DECIMAL COS(DECIMAL number)
    ```

-   Description

    This function calculates the cosine of number, which is a radian value.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If the input value is NULL, NULL is returned.

-   Examples

    ```
    COS(3.1415926/2)=2.6794896585028633e-8
    COS(3.1415926)=-0.9999999999999986
    ```


## COSH

-   Syntax

    ```
    DOUBLE COSH(DOUBLE number)
    DECIMAL COSH(DECIMAL number)
    ```

-   Description

    This function calculates the hyperbolic cosine of number.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If the input value is NULL, NULL is returned.


## COT

-   Syntax

    ```
    DOUBLE COT(DOUBLE number)
    DECIMAL COT(DECIMAL number)
    ```

-   Description

    This function calculates the cotangent of number, which is a radian value.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If the input value is NULL, NULL is returned.


## EXP

-   Syntax

    ```
    DOUBLE EXP(DOUBLE number)
    DECIMAL EXP(DECIMAL number)
    ```

-   Description

    This function calculates the exponential value of number.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    The exponential value of number is returned. The value is of the DOUBLE or DECIMAL type. If the input value is NULL, NULL is returned.


## FLOOR

-   Syntax

    ```
    BIGINT FLOOR(DOUBLE number)
    BIGINT FLOOR(DECIMAL number)
    ```

-   Description

    This function rounds down number and returns the nearest integer that is no greater than the value of number.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the BIGINT type is returned. If the input value is NULL, NULL is returned.

-   Examples

    ```
    FLOOR(1.2)=1
    FLOOR(1.9)=1
    FLOOR(0.1)=0
    FLOOR(-1.2)=-2
    FLOOR(-0.1)=-1
    FLOOR(0.0)=0
    FLOOR(-0.0)=0
    ```


## LN

-   Syntax

    ```
    DOUBLE LN(DOUBLE number)
    DECIMAL LN(DECIMAL number)
    ```

-   Description

    This function calculates the natural logarithm of number.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type.

    -   If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.
    -   If the input value is NULL, NULL is returned. If the input value is a negative value or 0, an error is returned.
-   Return value

    A value of the DOUBLE or DECIMAL type is returned.


## LOG

-   Syntax

    ```
    DOUBLE LOG(DOUBLE base, DOUBLE x)
    DECIMAL LOG(DECIMAL base, DECIMAL x)
    ```

-   Description

    This function calculates the logarithm of x whose base number is base.

-   Parameters
    -   base: the base value, which is of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.
    -   x: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.
-   Return value

    The logarithm value of the DOUBLE or DECIMAL type is returned.

    -   If any input value is NULL, NULL is returned.
    -   If any input value is a negative value or 0, an error is returned.
    -   If the value of base is 1, an error is returned. The value 1 causes division by 0.

## POW

-   Syntax

    ```
    DOUBLE POW(DOUBLE x, DOUBLE y)
    DECIMAL POW(DECIMAL x, DECIMAL y)
    ```

-   Description

    This function calculates the yth power of x, namely, `x^y`.

-   Parameters
    -   x: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.
    -   y: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.
-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If any input value is NULL, NULL is returned.


## RAND

-   Syntax

    ```
    DOUBLE RAND(BIGINT seed)
    ```

-   Description

    This function returns a random number of the DOUBLE type. The value ranges from 0 to 1.

-   Parameters

    seed: the random seed of the BIGINT type. This parameter specifies the starting point in generating random numbers. The parameter is optional.

-   Return value

    A value of the DOUBLE type is returned.

-   Examples

    ```
    SELECT RAND() FROM;
    SELECT RAND(1) FROM;
    ```


## ROUND

-   Syntax

    ```
    DOUBLE ROUND(DOUBLE number, [BIGINT DECIMAL_places])
    DECIMAL ROUND(DECIMAL number, [BIGINT DECIMAL_places])
    ```

-   Description

    This function returns a number rounded to the specified decimal place.

-   Parameters
    -   number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.
    -   DECIMAL\_places: the decimal place, which is a constant of the BIGINT type. The value is rounded to the decimal point. If it is of another data type, an error is returned. If this parameter is not specified, the number is rounded to the ones place. The default value is 0.

        **Note:** The value of DECIMAL\_places can be negative. A negative value is counted from the decimal point to the left and the decimal part is excluded. If DECIMAL\_places exceeds the length of the integer part, 0 is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If any input value is NULL, NULL is returned.

-   Examples

    ```
    ROUND(125.315)=125.0
    ROUND(125.315, 0)=125.0
    ROUND(125.315, 1)=125.3
    ROUND(125.315, 2)=125.32
    ROUND(125.315, 3)=125.315
    ROUND(-125.315, 2)=-125.32
    ROUND(123.345, -2)=100.0
    ROUND(null)=null
    ROUND(123.345, 4)=123.345
    ROUND(123.345, -4)=0.0
    ```


## SIN

-   Syntax

    ```
    DOUBLE SIN(DOUBLE number)
    DECIMAL SIN(DECIMAL number)
    ```

-   Description

    This function calculates the sine of number, which is a radian value.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If the input value is NULL, NULL is returned.


## SINH

-   Syntax

    ```
    DOUBLE SINH(DOUBLE number)
    DECIMAL SINH(DECIMAL number)
    ```

-   Description

    This function calculates the hyperbolic sine of number.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If the input value is NULL, NULL is returned.


## SQRT

-   Syntax

    ```
    DOUBLE SQRT(DOUBLE number)
    DECIMAL SQRT(DECIMAL number)
    ```

-   Description

    This function calculates the square root of number.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. It must be greater than 0. If it is less than 0, an error is returned. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If the input value is NULL, NULL is returned.


## TAN

-   Syntax

    ```
    DOUBLE TAN(DOUBLE number)
    DECIMAL TAN(DECIMAL number)
    ```

-   Description

    This function calculates the tangent of number, which is a radian value.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If the input value is NULL, NULL is returned.


## TANH

-   Syntax

    ```
    DOUBLE TANH(DOUBLE number)
    DECIMAL TANH(DECIMAL number)
    ```

-   Description

    This function calculates the hyperbolic tangent of number.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If the input value is NULL, NULL is returned.


## TRUNC

-   Syntax

    ```
    DOUBLE TRUNC(DOUBLE number[, BIGINT DECIMAL_places])
    DECIMAL TRUNC(DECIMAL number[, BIGINT DECIMAL_places])
    ```

-   Description

    This function truncates the input value of number to a specified number of decimal places.

-   Parameters
    -   number: a value of the DOUBLE or DECIMAL type. If the input value is of the STRING or BIGINT type, it is implicitly converted to a value of the DOUBLE type before calculation. If the input value is of another data type, an error is returned.
    -   DECIMAL\_places: the decimal place, which is a constant of the BIGINT type. This parameter indicates the position where the number is truncated. If it is of another data type, it is converted to the BIGINT type. If this parameter is not specified, the number is truncated to the ones place.

        **Note:** DECIMAL\_places can be a negative value, which indicates that the number is truncated from the decimal point to the left and the decimal part is left out. If DECIMAL\_places exceeds the length of the integer part, 0 is returned.

-   Return value

    A value of the DOUBLE or DECIMAL type is returned. If any input value is NULL, NULL is returned.

    **Note:**

    -   If a value of the DOUBLE type is returned, the return value may not be properly displayed. This issue exists in all systems. For more information, see `TRUNC(125.815,1)` in this example.
    -   The number is filled with zeros from the specified position.
-   Examples

    ```
    TRUNC(125.815)=125.0
    TRUNC(125.815,0)=125.0
    TRUNC(125.815,1)=125.80000000000001
    TRUNC(125.815,2)=125.81
    TRUNC(125.815,3)=125.815
    TRUNC(-125.815,2)=-125.81
    TRUNC(125.815,-1)=120.0
    TRUNC(125.815,-2)=100.0
    TRUNC(125.815,-3)=0.0
    TRUNC(123.345,4)=123.345
    TRUNC(123.345,-4)=0.0
    ```


## Additional functions of MaxCompute V2.0

If a new data type, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, or BINARY, needs to be used in a MaxCompute V2.0 SQL statement, you must insert a SET command before the SQL statement to enable the new data type.

-   Session level: To use a new data type, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement, and commit and execute them together.
-   Project level: The project owner can set the project as needed. It takes 10 to 15 minutes for the settings to take effect. Run the following command:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `setproject`, see [Project operations](/intl.en-US/Development/Common commands/Project operations.md). For the precautions you must take when you enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## LOG2

-   Syntax

    ```
    DOUBLE LOG2(DOUBLE number)
    DOUBLE LOG2(DECIMAL number)
    ```

-   Description

    This function calculates the logarithm of number with the base number of 2.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type.

-   Return value

    A value of the DOUBLE type is returned. If the input value is 0 or NULL, NULL is returned.

-   Examples

    ```
    LOG2(null)=null
    LOG2(0)=null
    LOG2(8)=3.0
    ```


## LOG10

-   Syntax

    ```
    DOUBLE LOG10(DOUBLE number)
    DOUBLE LOG10(DECIMAL number)
    ```

-   Description

    This function calculates the logarithm of number with the base number of 10.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type.

-   Return value

    A value of the DOUBLE type is returned. If the input value is 0 or NULL, NULL is returned.

-   Examples

    ```
    LOG10(null)=null
    LOG10(0)=null
    LOG10(8)=0.9030899869919435
    ```


## BIN

-   Syntax

    ```
    STRING BIN(BIGINT number)
    ```

-   Description

    This function calculates the binary code of number.

-   Parameters

    number: a value of the BIGINT type.

-   Return value

    A value of the STRING type is returned. If the input value is 0, 0 is returned. If the input value is NULL, NULL is returned.

-   Examples

    ```
    BIN(0)='0'
    BIN(null)='null'
    BIN(12)='1100'
    ```


## HEX

-   Syntax

    ```
    STRING HEX(BIGINT number) 
    STRING HEX(STRING number)
    STRING HEX(BINARY number)
    ```

-   Description

    This function converts an integer or a string into a hexadecimal number.

-   Parameters

    number: If number is of the BIGINT type, a hexadecimal number is returned. If number is of the STRING type, a string in hexadecimal format is returned.

-   Return value
    -   If the input value is not 0 or NULL, a value of the STRING type is returned.
    -   If the input value is 0, 0 is returned.
    -   If the input value is NULL, an error is returned.
-   Examples

    ```
    HEX(0)=0
    HEX('abc')='616263'
    HEX(17)='11'
    HEX('17')='3137'
    HEX(null): indicates that an exception occurs and the execution failed.
    ```


## UNHEX

-   Syntax

    ```
    BINARY UNHEX(STRING number)
    ```

-   Description

    This function converts a hexadecimal string into a string.

-   Parameters

    number: the hexadecimal string.

-   Return value

    A value of the BINARY type is returned. If the input value is 0, an error is returned. If the input value is NULL, NULL is returned.

-   Examples

    ```
    UNHEX('616263')='abc'
    UNHEX(616263)='abc'
    ```


## RADIANS

-   Syntax

    ```
    DOUBLE RADIANS(DOUBLE number)
    ```

-   Description

    This function converts a degree to a radian value.

-   Parameters

    number: a value of the DOUBLE type.

-   Return value

    A value of the DOUBLE type is returned. If the input value is NULL, NULL is returned.

-   Examples

    ```
    RADIANS(90)=1.5707963267948966
    RADIANS(0)=0.0
    RADIANS(null)=null
    ```


## DEGREES

-   Syntax

    ```
    DOUBLE DEGREES(DOUBLE number) 
    DOUBLE DEGREES(DECIMAL number)
    ```

-   Description

    This function converts a radian value to a degree.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type.

-   Return value

    A value of the DOUBLE type is returned. If the input value is NULL, NULL is returned.

-   Examples

    ```
    DEGREES(1.5707963267948966)=90.0
    DEGREES(0)=0.0
    DEGREES(null)=null
    ```


## SIGN

-   Syntax

    ```
    DOUBLE SIGN(DOUBLE number)
    DOUBLE SIGN(DECIMAL number)
    ```

-   Description

    This function returns the sign of an input parameter.

-   Parameters

    number: a value of the DOUBLE or DECIMAL type.

-   Return value

    A value of the DOUBLE type is returned.

    -   If the input value is a positive value, 1.0 is returned.
    -   If the input value is a negative value, -1.0 is returned.
    -   If the input value is 0, 0.0 is returned.
    -   If the input value is NULL, NULL is returned.
-   Examples

    ```
    SIGN(-2.5)=-1.0
    SIGN(2.5)=1.0
    SIGN(0)=0.0
    SIGN(null)=null
    ```


## E

-   Syntax

    ```
    DOUBLE E()
    ```

-   Description

    This function calculates the value of `e`.

-   Return value

    A value of the DOUBLE type is returned.

-   Examples

    ```
    E()=2.718281828459045
    ```


## PI

-   Syntax

    ```
    DOUBLE PI()
    ```

-   Description

    This function calculates the value of π.

-   Return value

    A value of the DOUBLE type is returned.

-   Examples

    ```
    PI()=3.141592653589793
    ```


## FACTORIAL

-   Syntax

    ```
    BIGINT FACTORIAL(Int number)
    ```

-   Description

    This function calculates the factorial of number.

-   Parameters

    number: a value of the INT type. The value ranges from 0 to 20.

-   Return value

    A value of the BIGINT type is returned. If the input value is 0, 1 is returned. If the input value is NULL or a value that does not fall into the range from 0 to 20, NULL is returned.

-   Examples

    ```
    FACTORIAL(5)=120 --5! =5*4*3*2*1=120
    ```


## CBRT

-   Syntax

    ```
    DOUBLE CBRT(DOUBLE number)
    ```

-   Description

    This function calculates the cube root of number.

-   Parameters

    number: a value of the DOUBLE type.

-   Return value

    A value of the DOUBLE type is returned. If the input value is NULL, NULL is returned.

-   Examples

    ```
    CBRT(8)=2
    CBRT(null)=null
    ```


## SHIFTLEFT

-   Syntax

    ```
    INT SHIFTLEFT(TINYINT|SMALLINT|INT number1, INT number2)
    BIGINT SHIFTLEFT(BIGINT number1, INT number2)
    ```

-   Description

    This function shifts a value left by a specific number of places \(<<\).

-   Parameters
    -   number1: an integer of the TINYINT, SMALLINT, INT, or BIGINT type.
    -   number2: an integer of the INT type.
-   Return value

    A value of the INT or BIGINT type is returned.

-   Examples

    ```
    SHIFTLEFT(1,2)=4  // Shift the binary value of 1 two places to the left (1<<2,0001 shifted to be 0100).
    SHIFTLEFT(4,3)=32  // Shift the binary value of 4 three places to the left (4<<3,0100 shifted to be 100000).
    ```


## SHIFTRIGHT

-   Syntax

    ```
    INT SHIFTRIGHT(TINYINT|SMALLINT|INT number1, INT number2)
    BIGINT SHIFTRIGHT(BIGINT number1, INT number2)
    ```

-   Description

    This function shifts a value right by a specific number of places \(\>\>\).

-   Parameters
    -   number1: an integer of the TINYINT, SMALLINT, INT, or BIGINT type.
    -   number2: an integer of the INT type.
-   Return value

    A value of the INT or BIGINT type is returned.

-   Examples

    ```
    SHIFTRIGHT(4,2)=1 // Shift the binary value of 4 two places to the right (4>>2,0100 shifted to be 0001).
    SHIFTRIGHT(32,3)=4 // Shift the binary value of 32 three places to the right (32>>3,100000 shifted to be 0100).
    ```


## SHIFTRIGHTUNSIGNED

-   Syntax

    ```
    INT SHIFTRIGHTUNSIGNED(TINYINT|SMALLINT|INT number1, INT number2)
    BIGINT SHIFTRIGHTUNSIGNED(BIGINT number1, INT number2)
    ```

-   Description

    This function shifts an unsigned value right by a specific number of places \(\>\>\>\).

-   Parameters
    -   number1: an integer of the TINYINT, SMALLINT, INT, or BIGINT type.
    -   number2: an integer of the INT type.
-   Return value

    A value of the INT or BIGINT type is returned.

-   Examples

    ```
    SHIFTRIGHTUNSIGNED(8,2)=2  // Shift the binary unsigned value of 8 two places to the right (8>>>2,1000 shifted to be 0010).
    SHIFTRIGHTUNSIGNED(-14,2)=1073741820  // Shift the binary value of -14 two places to the right (-14>>>2, 11111111 11111111 11111111 11110010 shifted to be 00111111 11111111 11111111 11111100).
    ```


## FORMAT\_NUMBER

-   Syntax

    ```
    STRING FORMAT_NUMBER(FLOAT|DOUBLE|DECIMAL expr1, expr2)
    ```

-   Description

    This function converts a number to a string in the specified format.

-   Parameters
    -   expr1: a numeric expression that you want to format.
    -   expr2: the number of decimal places. It can be of the INT type. It can also be expressed in the format of `#,###,###. ##`.
        -   If expr2 is greater than 0, the value is rounded to the specified place after the decimal point.
        -   If expr2 is equal to 0, the value has no decimal point or fractional part.
        -   If expr2 is less than 0 or greater than 340, an error is returned.
-   Return value

    A value of the STRING type is returned.

-   Examples

    ```
    SELECT FORMAT_NUMBER(5.230134523424545456,3);// The return value is 5.230.
    SELECT FORMAT_NUMBER(12332.123456, '#,###,###,###. ###');// The return value is 12,332.123.
    ```


## WIDTH\_BUCKET

-   Syntax

    ```
    WIDTH_BUCKET(NUMERIC expr, NUMERIC min_value, NUMERIC max_value, INT num_buckets)
    ```

-   Description

    This function specifies the number of buckets and the minimum and maximum values of the acceptable range for a bucket. It allows you to construct equi-width buckets, in which the bucket range is divided into intervals that have an identical size. It returns the ID of the bucket into which the value of a specific expression falls. This function supports the following data types: DECIMAL\(precision,scale\) in the MaxCompute V2.0 data type edition, BIGINT, INT, FLOAT, DOUBLE, and DECIMAL. For more information, see [MaxCompute V2.0 data type edition](/intl.en-US/Development/Data types/MaxCompute V2.0 data type edition.md).

-   Parameters
    -   expr: the expression whose matching bucket ID you want to identify.
    -   min\_value: the minimum value of the acceptable range for the bucket.
    -   max\_value: the maximum value of the acceptable range for the bucket. The value must be greater than min\_value.
    -   num\_buckets: the number of buckets. The value must be greater than 0.
-   Return value

    A value of the BIGINT type is returned. The value ranges from 0 to num\_buckets plus 1. If the value of expr is less than min\_value, 0 is returned. If the value of expr is greater than max\_value, the value of num\_buckets plus 1 is returned. If the value of expr is NULL, NULL is returned. In other cases, the ID of the bucket into which the value falls is returned. The bucket ID is named based on the following formula: `Bucket ID = FLOOR[num_buckets × (expr - min_value)/(max_value - min_value) + 1]`.

-   Examples

    ```
    SELECT key,value,WIDTH_BUCKET(value,100,500,5) as value_group
    FROM VALUES
        (1,99),
        (2,100),
        (3,199),
        (4,200),
        (5,499),
        (6,500),
        (7,501),
        (8,NULL)
    AS t(key,value);
    -- The following result is returned:
    +-------+--------+-------------+
    | key   | value  | value_group |
    +-------+--------+-------------+
    | 1     | 99     | 0           |
    | 2     | 100    | 1           |
    | 3     | 199    | 2           |
    | 4     | 200    | 2           |
    | 5     | 499    | 5           |
    | 6     | 500    | 6           |
    | 7     | 501    | 6           |
    | 8     | \N     | \N          |
    +-------+--------+-------------+
    ```


