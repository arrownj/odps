---
keyword: date functions
---

# Date functions

MaxCompute SQL provides common date functions. You can select an appropriate date function based on your needs to complete date calculation and conversion. This topic describes the syntax, parameters, and examples of date functions supported by MaxCompute SQL. It guides you through data development by using date functions.

|Function|Description|
|--------|-----------|
|[DATEADD](#section_qjz_lrl_vdb)|Modifies a date value based on datepart and delta.|
|[DATE\_ADD](#section_aza_roh_gfl)|Determines the value of startdate based on delta.|
|[DATEDIFF](#section_xl2_nsl_vdb)|Calculates the difference between two date values based on the time unit specified by datepart.|
|[DATEPART](#section_am4_xtl_vdb)|Returns a specified part of a date value.|
|[DATETRUNC](#section_zbr_d5l_vdb)|Truncates a date value based on the time unit specified by datepart.|
|[GETDATE](#section_o4p_45l_vdb)|Returns the current system time as a date value.|
|[ISDATE](#section_rzl_s5l_vdb)|Determines whether a date string can be converted into a date value in a specified format.|
|[LASTDAY](#section_vhk_w2m_vdb)|Returns the last day of the month in which the specified date value falls.|
|[TO\_DATE](#section_b3z_1fm_vdb)|Converts a string to a date value in a specified format.|
|[TO\_CHAR](#section_a2d_rfm_vdb)|Converts a date value to a string in a specified format.|
|[UNIX\_TIMESTAMP](#section_k4r_zfm_vdb)|Converts a date value to a UNIX timestamp that is an integer.|
|[FROM\_UNIXTIME](#section_yzv_j5l_vdb)|Converts a UNIX timestamp of the BIGINT type to a date value of the DATETIME type.|
|[WEEKDAY](#section_g41_2gm_vdb)|Returns a number that represents the day of the week in which the provided date falls.|
|[WEEKOFYEAR](#section_rjv_hgm_vdb)|Returns a number that represents the week of the year in which the provided date falls.|
|[YEAR](#section_gb4_g3m_vdb)|Returns the year in which the specified date value falls.|
|[QUARTER](#section_ckz_m3m_vdb)|Returns the quarter in which the specified date value falls.|
|[MONTH](#section_atw_s3m_vdb)|Returns the month in which the specified date value falls.|
|[DAY](#section_lt5_w3m_vdb)|Returns the day in which the specified date value falls.|
|[DAYOFMONTH](#section_k4p_fjm_vdb)|Returns the day part of a date value.|
|[HOUR](#section_lrl_jjm_vdb)|Returns the hour part of a date value.|
|[MINUTE](#section_o15_mjm_vdb)|Returns the minute part of a date value.|
|[SECOND](#section_ylr_pjm_vdb)|Returns the second part of a date value.|
|[CURRENT\_TIMESTAMP](#section_r1r_sjm_vdb)|Returns the current timestamp.|
|[FROM\_UTC\_TIMESTAMP](#section_unk_x6h_uor)|Converts a UTC timestamp to a timestamp for a specified time zone.|
|[ADD\_MONTHS](#section_psn_vjm_vdb)|Returns a date value that is obtained after a number of months are added to a specified date.|
|[LAST\_DAY](#section_idn_1km_vdb)|Returns the last day of the month in which the specified date value falls.|
|[NEXT\_DAY](#section_eqf_2km_vdb)|Returns the date of the first weekday that is later than a specified data value.|
|[MONTHS\_BETWEEN](#section_cbq_4km_vdb)|Returns the number of months between date1 and date2.|
|[EXTRACT](#section_d4o_h86_fj6)|Returns a specified part of a timestamp.|

## DATEADD

-   Syntax

    ```
    DATETIME DATEADD(DATETIME date, BIGINT delta, STRING datepart)
    ```

-   Description

    Modifies a `date` value based on `datepart` and `delta` that you specified.

-   Parameters
    -   date: a date value of the DATETIME type.

        If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.

    -   delta: a value of the BIGINT type, which indicates the interval to add to or subtract from the specified part of the date value. If the value of `delta` is greater than 0, the date value is incremented. Otherwise, the date value is decremented.

        If the input value is of the STRING or DOUBLE type, it is implicitly converted into a value of the BIGINT type before calculation. If the input value is of another data type, an error is returned.

        **Note:**

        -   If you add or subtract the interval specified by `delta` at a date part, a carry or return at more significant date parts may occur. The year, month, hour, minute, and second parts are computed by using different numeral systems. The year part uses the base-10 numeral system. The month part uses the base-12 numeral system. The hour part uses the base-24 numeral system. The minute and second parts use the base-60 numeral system.
        -   If the DATEADD function adds an interval specified by `delta` to the month part of a date value and this operation does not cause an overflow of `day`, keep `day` unchanged. Otherwise, set `day` to the last day of the specified month.
    -   datepart: the part you want to modify in the date value. The value is a constant of the STRING type. For example, `dd` indicates a day. An error is returned if the value is in an invalid format or is not a constant of the STRING type.

        The value of this parameter is specified in compliance with the rules of conversions between the STRING and DATETIME types. That is, the value yyyy indicates that the DATEADD function adds an interval to the year part of the date value, whereas the value mm indicates that the DATEADD function adds an interval to the month part of the date value. For more information about the rules of data type conversion, see [Type conversion](/intl.en-US/Development/SQL/Data type conversion.md). Extended date formats are also supported, such as `-year`, `-month`, `-mon`, `-day`, and `-hour`.

-   Return value

    A value of the DATETIME type. If any input parameter is set to NULL, NULL is returned.

-   Examples
    -   Example 1: Common use of DATEADD

        ```
        SELECT DATEADD(DATETIME '2005-02-28 00:00:00', 1, 'dd') ;
        -- The return value is 2005-03-01 00:00:00. After one day is added, the result is beyond the last day of February. The actual date value is the first day of March.
        SELECT DATEADD(DATETIME '2005-02-28 00:00:00', -1, 'dd');
        -- The return value is 2005-02-27 00:00:00. One day is subtracted.
        SELECT DATEADD(DATETIME '2005-02-28 00:00:00', 20, 'mm');
        -- The return value is 2006-10-28 00:00:00. After 20 months are added, the month overflows, and the year increases by 1.
        SELECT DATEADD(DATETIME '2005-02-28 00:00:00', 1, 'mm');
        -- The return value is 2005-03-28 00:00:00.
        SELECT DATEADD(DATETIME '2005-01-29 00:00:00', 1, 'mm');
        -- The return value is 2005-02-28 00:00:00. February in 2005 has only 28 days. Therefore, the last day of February is returned.
        SELECT DATEADD(DATETIME '2005-03-30 00:00:00', -1, 'mm');
        -- The return value is 2005-02-28 00:00:00.
        ```

    -   Example 2: Use of DATEADD in which a value of the DATETIME type is expressed as a constant.

        In MaxCompute SQL statements, a value of the DATETIME type cannot be directly expressed as a constant. The following statement uses an invalid expression of a value of the DATETIME type:

        ```
        SELECT DATEADD(2005-03-30 00:00:00, -1, 'mm') FROM tbl1;
        ```

        The following statement demonstrates how to correctly express a constant as a value of the DATETIME type.

        ```
        -- Explicitly convert a constant of the STRING type to the DATETIME type.
        SELECT DATEADD(CAST("2005-03-30 00:00:00" AS DATETIME), -1, 'mm') FROM tbl1;
        ```


## DATE\_ADD

-   Syntax

    ```
    DATE DATE_ADD(DATE/TIMESTAMP/STRING startdate, BIGINT delta)
    ```

-   Description

    Determines the value of startdate based on delta.

-   Parameters
    -   startdate: the start date. The value of the DATE, DATETIME, or STRING type is supported. An error is returned if the value is of another data type.

        If the input value is of the STRING type, it is implicitly converted into a value of the DATE type before calculation. The value of the STRING type is in the format of `'yyyy-mm-dd'`, for example, `'2019-12-27'`.

    -   delta: a value of the BIGINT type, which indicates the interval to add to or subtract from the specified part of the date value. An error is returned if it is of another type. If the value of delta is greater than 0, the date value is incremented. Otherwise, the date value is decremented.
-   Return value

    A value of the DATE type.

-   Examples

    ```
    SET odps.sql.type.system.odps2=true;
    -- Enable the new data types introduced in MaxCompute V2.0. Commit this statement along with SQL statements.
    SELECT DATE_ADD(DATETIME '2005-02-28 00:00:00', 1);
    -- The return value is 2005-03-01. After one day is added, the result is beyond the last day of February. The actual value is the first day of March.
    SELECT DATE_ADD(DATE '2005-02-28', -1);
    -- The return value is 2005-02-27. One day is subtracted.
    SELECT DATE_ADD( '2005-02-28 00:00:00', 20);
    -- The return value is 2005-03-20.
    ```


## DATEDIFF

-   Syntax

    ```
    BIGINT DATEDIFF(DATETIME date1, DATETIME date2, STRING datepart)
    ```

-   Description

    Calculates the difference between date1 and date2. The difference is measured in the time unit specified by datepart.

-   Parameters
    -   datet1 and date2: values of the DATETIME type, which indicate the minuend and subtrahend. If the input values are of the STRING type, they are implicitly converted into the values of the DATETIME type before calculation. If the input values are of another data type, an error is returned.
    -   datepart: the time unit, which is a constant of the STRING type. It supports extended date formats. If datepart is not in the specified format or is of another data type, an error is returned.

        If you enable [new data types](/intl.en-US/Development/Data types/Data type editions.md) of MaxCompute, datepart can be left unspecified. By default, day is used.

        **Note:** This function omits the lower unit based on the time unit specified by datepart and then calculates the result.

-   Return value

    A value of the BIGINT type. If any input parameter is set to NULL, NULL is returned. A negative value is returned if date1 is earlier than date2.

-   Examples

    ```
    --The start time is 2005-12-31 23:59:59 and the end time is 2006-01-01 00:00:00.
        SELECT DATEDIFF(end, start, 'dd') = 1
        SELECT DATEDIFF(end, start, 'mm') = 1
        SELECT DATEDIFF(end, start, 'yyyy') = 1
        SELECT DATEDIFF(end, start, 'hh') = 1
        SELECT DATEDIFF(end, start, 'mi') = 1
        SELECT DATEDIFF(end, start, 'ss') = 1
        SELECT DATEDIFF(DATETIME'2013-05-31 13:00:00', '2013-05-31 12:30:00', 'ss') = 1800
        SELECT DATEDIFF(DATETIME'2013-05-31 13:00:00', '2013-05-31 12:30:00', 'mi') = 30
    --The start time is 2018-06-04 19:33:23.234 and the end time is 2018-06-04 19:33:23.250. Date values with milliseconds do not adopt the standard DATETIME type and therefore cannot be implicitly converted into the DATETIME type. In this case, explicit conversion is required.
        SELECT DATEDIFF(TO_DATE('2018-06-04 19:33:23.250', 'yyyy-MM-dd hh:mi:ss.ff3'),TO_DATE('2018-06-04 19:33:23.234', 'yyyy-MM-dd hh:mi:ss.ff3') , 'ff3') = 16
    ```


## DATEPART

-   Syntax

    ```
    BIGINT DATEPART(DATETIME date, STRING datepart)
    ```

-   Description

    Returns a specified part of a date value.

-   Parameters
    -   date: a value of the DATETIME type. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.
    -   datepart: a constant of the STRING type. It supports extended date formats. If datepart is not in the specified format or is of another data type, an error is returned.
-   Return value

    A value of the BIGINT type. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    SELECT DATEPART(DATETIME'2013-06-08 01:10:00', 'yyyy')  =  2013
    SELECT DATEPART(DATETIME'2013-06-08 01:10:00', 'mm')  =  6
    ```


## DATETRUNC

-   Syntax

    ```
    DATETIME DATETRUNC (DATETIME date, STRING datepart)
    ```

-   Description

    Truncates a date value based on the time unit specified by datepart.

-   Parameters
    -   date: a value of the DATETIME type. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.
    -   datepart: a constant of the STRING type. It supports extended date formats. If `datepart` is not in the specified format or is of another data type, an error is returned.
-   Return value

    A value of the DATETIME type. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    SELECT DATETRUNC(DATETIME'2011-12-07 16:28:46', 'yyyy') = 2011-01-01 00:00:00
    SELECT DATETRUNC(DATETIME'2011-12-07 16:28:46', 'month') = 2011-12-01 00:00:00
    SELECT DATETRUNC(DATETIME'2011-12-07 16:28:46', 'DD') = 2011-12-07 00:00:00
    ```


## GETDATE

-   Syntax

    ```
    DATETIME GETDATE()
    ```

-   Description

    Returns the current system time as a date value. MaxCompute uses UTC+8 as the standard time zone.

-   Return value

    The current date and time of the DATETIME type.

    **Note:** In a MaxCompute SQL task that is executed in a distributed manner, `GETDATE` always returns a fixed value. The return value is an arbitrary time during the execution of the MaxCompute SQL task. The time is accurate to seconds. In MaxCompute V2.0 that supports more data types, the time is accurate to milliseconds.


## ISDATE

-   Syntax

    ```
    BOOLEAN ISDATE(STRING date, STRING format)
    ```

-   Description

    Determines whether a date string can be converted into a date value in a specified format. If the date string can be converted into a date value in the specified format, TRUE is returned. Otherwise, FALSE is returned.

-   Parameters
    -   date: a value of the STRING type. If the input value is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type, it is implicitly converted into a value of the STRING type before calculation. If the input value is of another data type, an error is returned.
    -   format: a constant of the STRING type. It does not support extended date formats. If the input value is of another data type, an error is returned. If redundant format strings exist in format, this function converts the date string that corresponds to the first format string into a date value. The rest strings are treated as delimiters. For example, `ISDATE("1234-yyyy", "yyyy-yyyy")` returns TRUE.
-   Return value

    A value of the BOOLEAN type. If any input parameter is set to NULL, NULL is returned.


## LASTDAY

-   Syntax

    ```
    DATETIME LASTDAY(DATETIME date)
    ```

-   Description

    Returns the last day of the month in which the specified date value falls. The value is accurate to days. The hour, minute, and second parts are expressed as `00:00:00`.

-   Parameter

    date: a value of the DATETIME type. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    A value of the DATETIME type. If any input parameter is set to NULL, NULL is returned.


## TO\_DATE

-   Syntax

    ```
    DATETIME TO_DATE(STRING date, STRING format)
    ```

-   Description

    Converts a string to a date value in a specified format.

-   Parameters
    -   date: a date value of the STRING type, which indicates the date string you want to convert. If the input value is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type, it is implicitly converted into a value of the STRING type before calculation. If the input value is of another data type or an empty string, an error is returned.
    -   format: a constant of the STRING type, which indicates a date format. If the input value is of another data type, an error is returned. format does not support the extended date formats. Other characters are omitted as invalid characters during parsing.

        The value of format must contain `yyyy`. Otherwise, an error is returned. If redundant format strings exist in format, this function converts the date string that corresponds to the first format string into a date value. The rest strings are treated as delimiters. For example, `TO_DATE("1234-2234", "yyyy-yyyy")` returns `1234-01-01 00:00:00`.

        In the format, yyyy indicates a 4-digit year, mm indicates a 2-digit month, dd indicates a 2-digit day, hh indicates an hour based on the base-24 numeral system, mi indicates a 2-digit minute, ss indicates a 2-digit second, and ff3 indicates a 3-digit millisecond.

-   Return value

    A value of the DATETIME type and in the format of `yyyy-mm-dd hh:mi:ss`. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    SELECT TO_DATE('Alibaba 2010-12*03', 'Alibaba yyyy-mm*dd') = 2010-12-03 00:00:00
    SELECT TO_DATE('20080718', 'yyyymmdd') = 2008-07-18 00:00:00
    SELECT TO_DATE('200807182030','yyyymmddhhmi') = 2008-07-18 20:30:00
    SELECT TO_DATE('2008718', 'yyyymmdd') = null --'2008718' cannot be converted to a standard date value, and an error is returned. It must be written as '20080718'.
    SELECT TO_DATE('Alibaba 2010-12*3', 'Alibaba yyyy-mm*dd') = null --'Alibaba 2010-12*3' cannot be converted to a standard date value, and an error is returned. It must be written as 'Alibaba 2010-12*03'.
    SELECT TO_DATE('2010-24-01', 'yyyy') = null --'2010-24-01' cannot be converted to a standard date value, and an error is returned. It must be written as '2010-01-24'.
    SELECT TO_DATE('20181030 15-13-12.345','yyyymmdd hh-mi-ss.ff3')=2018-10-30 15:13:12
    ```


## TO\_CHAR

-   Syntax

    ```
    STRING TO_CHAR(DATETIME date, STRING format)
    ```

-   Description

    Converts a date value of the DATETIME type into a string in a specified format.

-   Parameters
    -   date: a date value of the DATETIME type, which indicates the date value you want to convert. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.
    -   format: a constant of the STRING type. An error is returned if the value is not a constant or not of the STRING type. In the format parameter, the date format part is replaced by the related data and other characters remain unchanged in the output.
-   Return value

    A value of the STRING type. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    SELECT TO_CHAR(DATETIME'2010-12-03 00:00:00', 'Alibaba Cloud Financial Services yyyy-mm*dd') = 'Alibaba Cloud Financial Services 2010-12*03'
    SELECT TO_CHAR(DATETIME'2008-07-18 00:00:00', 'yyyymmdd') = '20080718' 
    SELECT TO_CHAR(DATETIME'Alibaba 2010-12*3', 'Alibaba yyyy-mm*dd') --'Alibaba 2010-12*3' cannot be converted to a standard date value, and an error is returned. It must be written as 'Alibaba 2010-12*03'.
    SELECT TO_CHAR(DATETIME'2010-24-01', 'yyyy') --'2010-24-01' is not a standard date value and an error is returned. It must be written as '2010-01-24'.
    SELECT TO_CHAR(DATETIME'2008718', 'yyyymmdd') --'2008718' is not a standard date value and an error is returned. It must be written as '20080718'.
    ```


## UNIX\_TIMESTAMP

-   Syntax

    ```
    BIGINT UNIX_TIMESTAMP(DATETIME date)
    ```

-   Description

    Converts a date value to a UNIX timestamp that is an integer.

-   Parameter

    date: a date value of the DATETIME type. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned. If new data types are enabled, the implicit conversion fails. In this case, you must use the CAST function for conversion. For example, you can use `UNIX_TIMESTAMP(CAST(... AS DATETIME))`.

-   Return value

    A UNIX timestamp of the BIGINT type. If any input parameter is set to NULL, NULL is returned.

-   Example

    ```
    SELECT UNIX_TIMESTAMP(DATETIME'2009-03-20 11:11:00'); -- The return value is 1237518660.
    ```


## FROM\_UNIXTIME

-   Syntax

    ```
    DATETIME FROM_UNIXTIME(BIGINT unixtime)
    ```

-   Description

    Converts unixtime of the BIGINT type to a date value of the DATETIME type.

-   Parameter

    unixtime: a date value of the BIGINT type in the UNIX format. Its value is accurate to seconds. If the input value is of the STRING, DOUBLE, or DECIMAL type, it is implicitly converted into a value of the BIGINT type before calculation.

-   Return value

    A value of the DATETIME type. If any input parameter is set to NULL, NULL is returned.

    **Note:** In HIVE-compatible mode where `set odps.sql.hive.compatible=true;` is executed, if the input value is of the STRING type, a date value of the STRING type is returned.

-   Example

    ```
    SELECT FROM_UNIXTIME(123456789) = 1973-11-30 05:33:09
    ```


## WEEKDAY

-   Syntax

    ```
    BIGINT WEEKDAY (DATETIME date)
    ```

-   Description

    Returns a number that represents the day of the week in which the provided date falls.

-   Parameter

    date: a date value of the DATETIME type.

-   Return value

    A value of the BIGINT type. If any input parameter is set to NULL, NULL is returned. Monday is treated as the first day of a week and its return value is 0. Days are numbered in ascending order starting from 0. The return value of Sunday is 6.


## WEEKOFYEAR

-   Syntax

    ```
    BIGINT WEEKOFYEAR (DATETIME date)
    ```

-   Description

    Returns a number that represents the week of the year in which the provided date falls. Monday is treated as the first day of a week.

    **Note:** To determine whether a week belongs to a year or its next year, find the year in which more than four days of the week fall. If the week belongs to the year, it is treated as the last week of the year. If the week belongs to the next year, it is treated as the first week of the next year.

-   Parameter

    date: a date value of the DATETIME type.

-   Return value

    A value of the BIGINT type. If any input parameter is set to NULL, NULL is returned.

-   Example

    ```
    SELECT WEEKOFYEAR(TO_DATE("20141229", "yyyymmdd")) FROM dual;  
    -- Return result:
    +------------+
    | _c0        |
    +------------+
    | 1          |
    +------------+
    -- 20141229 is in year 2014, but most days of the week fall in year 2015. Therefore, the return value 1 indicates the first week of year 2015.    
    SELECT WEEKOFYEAR(TO_DATE("20141231", "yyyymmdd")) FROM dual; -- The return value is 1.  
    SELECT WEEKOFYEAR(TO_DATE("20151229", "yyyymmdd")) FROM dual; -- The return value is 53.
    ```


## Additional functions provided by MaxCompute V2.0.

MaxCompute V2.0 provides additional date functions. If the functions you are using involve new data types, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, or BINARY, run the following SET statement to enable these data types:

-   Session level: To use a new data type, you must insert `set odps.sql.type.system.odps2=true;` before the SQL statement, and commit and execute them together.
-   Project level: The project owner can set the project as needed. It takes 10 to 15 minutes for the settings to take effect. Run the following command:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `setproject`, see [Project operations](/intl.en-US/Development/Common commands/Project operations.md). For the precautions you must take when you enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## YEAR

-   Syntax

    ```
    INT YEAR(DATETIME/STRING date)
    ```

-   Description

    Returns the year in which the specified date value falls. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the DATETIME or STRING type. The format of the date value must include `yyyy-mm-dd` and exclude redundant strings. Otherwise, NULL is returned.

-   Return value

    A value of the INT type.

-   Examples

    ```
    SELECT YEAR('1970-01-01 12:30:00') = 1970
    SELECT YEAR('1970-01-01') = 1970
    SELECT YEAR('70-01-01') = 70
    SELECT YEAR('1970-01-01') = 1970
    SELECT YEAR('1970/03/09') = null
    SELECT YEAR(null) -- An error is returned.
    ```


## QUARTER

-   Syntax

    ```
    INT QUARTER (DATETIME/TIMESTAMP/STRING date)
    ```

-   Description

    Returns the quarter in which the specified date value falls. Valid values: 1 to 4. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the DATETIME, TIMESTAMP, or STRING type. The format of the date value must include `yyyy-mm-dd`. If the date value is of another data type, NULL is returned.

-   Return value

    A value of the INT type. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    SELECT QUARTER('1970-11-12 10:00:00') = 4
    SELECT QUARTER('1970-11-12') = 4
    ```


## MONTH

-   Syntax

    ```
    INT MONTH(DATETIME/STRING date)
    ```

-   Description

    Returns the month in which the specified date value falls. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the DATETIME or STRING type. If it is of another data type, an error is returned.

-   Return value

    A value of the INT type.

-   Examples

    ```
    SELECT MONTH('2014-09-01') = 9
    SELECT MONTH('20140901') = null
    ```


## DAY

-   Syntax

    ```
    INT DAY(DATETIME/STRING date)
    ```

-   Description

    Returns the day in which the specified date value falls. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the DATETIME or STRING type. The format of the date value can be `yyyy-mm-dd` or `yyyy-mm-dd hh:mi:ss`. If the date value is of another data type, an error is returned.

-   Return value

    A value of the INT type.

-   Examples

    ```
    SELECT DAY('2014-09-01') = 1
    SELECT DAY('20140901') = null
    ```


## DAYOFMONTH

-   Syntax

    ```
    INT DAYOFMONTH(date)
    ```

-   Description

    Returns the day part of a date value. For example, if the date value is October 13 in 2017, 13 is returned after you run the command `INT DAYOFMONTH(2017-10-13)`. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the STRING type. An error is returned if it is of another data type.

-   Return value

    A value of the INT type.

-   Examples

    ```
    SELECT DAYOFMONTH('2014-09-01') = 1
    SELECT DAYOFMONTH('20140901') = null
    ```


## HOUR

-   Syntax

    ```
    INT HOUR(DATETIME/STRING date)
    ```

-   Description

    Returns the hour part of a date value.

-   Parameter

    date: a date value of the DATETIME or STRING type. If the date value is of another data type, an error is returned. This function is an additional function of MaxCompute V2.0.

-   Return value

    A value of the INT type.

-   Examples

    ```
    SELECT HOUR('2014-09-01 12:00:00') = 12
    SELECT HOUR('12:00:00') = 12
    SELECT HOUR('20140901120000') = null
    ```


## MINUTE

-   Syntax

    ```
    INT MINUTE(DATETIME/STRING date)
    ```

-   Description

    Returns the minute part of a date value. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the DATETIME or STRING type. If the date value is of another data type, an error is returned.

-   Return value

    A value of the INT type.

-   Examples

    ```
    SELECT MINUTE('2014-09-01 12:30:00') = 30
    SELECT MINUTE('12:30:00') = 30
    SELECT MINUTE('20140901120000') = null
    ```


## SECOND

-   Syntax

    ```
    INT SECOND(DATETIME/STRING date)
    ```

-   Description

    Returns the second part of a date value.

-   Parameter

    date: a date value of the DATETIME or STRING type. If the date value is of another data type, an error is returned. This function is an additional function of MaxCompute V2.0.

-   Return value

    A value of the INT type.

-   Examples

    ```
    SELECT SECOND('2014-09-01 12:30:45') = 45
    SELECT SECOND('12:30:45') = 45
    SELECT SECOND('20140901123045') = null
    ```


## CURRENT\_TIMESTAMP

-   Syntax

    ```
    TIMESTAMP CURRENT_TIMESTAMP()
    ```

-   Description

    Returns the current timestamp. The return value is not fixed. This function is an additional function of MaxCompute V2.0.

-   Return value

    A value of the TIMESTAMP type.

-   Example

    ```
    SELECT CURRENT_TIMESTAMP() FROM dual; -- The return value is '2017-08-03 11:50:30.661'.
    ```


## FROM\_UTC\_TIMESTAMP

-   Syntax

    ```
    TIMESTAMP FROM_UTC_TIMESTAMP({any primitive type}*, STRING timezone)
    ```

-   Description

    Converts a UTC timestamp to a timestamp for a specified time zone. This function is an additional function of MaxCompute V2.0.

-   Parameters
    -   \{any primitive type\}\*: a value of the TIMESTAMP, DATETIME, TINYINT, SMALLINT, INT, or BIGINT type. If the value is of the TINYINT, SMALLINT, INT, or BIGINT type, the unit is milliseconds.
    -   timezone: the destination time zone to convert the timestamp, such as PST.

        This function supports only the Asia/Shanghai time zone and does not support the GMT+9 time zone.

-   Return value

    A value of the TIMESTAMP type.

-   Examples

    ```
    SELECT FROM_UTC_TIMESTAMP(1501557840000, 'PST'); -- The unit of the input parameter is milliseconds (ms), and the return value is 2017-08-01 04:24:00.
    SELECT FROM_UTC_TIMESTAMP('1970-01-30 16:00:00','PST'); -- The return value is 1970-01-30 08:00:00.0.
    SELECT FROM_UTC_TIMESTAMP('1970-01-30','PST'); -- The return value is 1970-01-29 16:00:00.0.
    ```


## ADD\_MONTHS

-   Syntax

    ```
    STRING ADD_MONTHS(STRING startdate, INT nummonths)
    ```

-   Description

    Returns a date value that is obtained after num\_months are added to startdate. This function is an additional function of MaxCompute V2.0.

-   Parameters
    -   startdate: a value of the STRING type. Its format must include `yyyy-mm-dd`. Otherwise, NULL is returned.
    -   num\_months: a value of the INT type.
-   Return value

    A value of the STRING type and in the format of `yyyy-mm-dd`.

-   Examples

    ```
    SELECT ADD_MONTHS('2017-02-14',3) = '2017-05-14'
    SELECT ADD_MONTHS('17-2-14',3) = '0017-05-14'
    SELECT ADD_MONTHS('2017-02-14 21:30:00',3) = '2017-05-14'
    SELECT ADD_MONTHS('20170214',3) = null
    ```


## LAST\_DAY

-   Syntax

    ```
    STRING LAST_DAY(STRING date)
    ```

-   Description

    Returns the last day of the month in which the specified date value falls.

-   Parameter

    date: a date value of the STRING type and in the format of `yyyy-mm-dd hh:mi:ss` or `yyyy-mm-dd`.

-   Return value

    A value of the STRING type and in the format of `yyyy-mm-dd`.

-   Examples

    ```
    SELECT LAST_DAY('2017-03-04') = '2017-03-31'
    SELECT LAST_DAY('2017-07-04 11:40:00') = '2017-07-31'
    SELECT LAST_DAY('20170304') = null
    ```


## NEXT\_DAY

-   Syntax

    ```
    STRING NEXT_DAY(STRING startdate, STRING week)
    ```

-   Description

    Returns the date of the first day that is later than startdate and matches the week value. That is, the date of the specified day in the next week.

-   Parameters
    -   startdate: a date value of the STRING type and in the format of `yyyy-mm-dd hh:mi:ss` or `yyyy-mm-dd`.
    -   week: a value of the STRING type, which is the first two or three letters in the week name or the full week name, for example, MO, TUE, or FRIDAY.
-   Return value

    A value of the STRING type and in the format of `yyyy-mm-dd`.

-   Examples

    ```
    SELECT NEXT_DAY('2017-08-01','TU') = '2017-08-08'
    SELECT NEXT_DAY('2017-08-01 23:34:00','TU') = '2017-08-08'
    SELECT NEXT_DAY('20170801','TU') = null
    ```


## MONTHS\_BETWEEN

-   Syntax

    ```
    DOUBLE MONTHS_BETWEEN(DATETIME/TIMESTAMP/STRING date1, DATETIME/TIMESTAMP/STRING date2)
    ```

-   Description

    Returns the number of months between date1 and date2. This function is an additional function of MaxCompute V2.0.

-   Parameters
    -   date1: a date value of the DATETIME, TIMESTAMP, or STRING type and in the format of `yyyy-mm-dd hh:mi:ss` or `yyyy-mm-dd`.
    -   date2: a date value of the DATETIME, TIMESTAMP, or STRING type and in the format of `yyyy-mm-dd hh:mi:ss` or `yyyy-mm-dd`.
-   Return value

    A value of the DOUBLE type.

    -   A positive value is returned if date1 is later than date2. A negative value is returned if date2 is later than date1.
    -   If both date1 and date2 correspond to the last days of two months, the return value is an integer that represents the number of months. Otherwise, the return value is computed by using the following formula: \(date1 - date2\)/31
-   Examples

    ```
    SELECT MONTHS_BETWEEN('1997-02-28 10:30:00', '1996-10-30') = 3.9495967741935485
    SELECT MONTHS_BETWEEN('1996-10-30','1997-02-28 10:30:00' ) = -3.9495967741935485
    SELECT MONTHS_BETWEEN('1996-09-30','1996-12-31') = -3.0
    ```


## EXTRACT

-   Syntax

    ```
    INT EXTRACT(<datepart> FROM <timestamp>)
    ```

-   Description

    Extracts the date part specified by datepart from the date timestamp. This function is an additional function of MaxCompute V2.0.

-   Parameters
    -   datepart: a value that can be set to YEAR, MONTH, DAY, HOUR, or MINUTE.
    -   timestamp: a date value of the TIMESTAMP type.
-   Return value

    A value of the INT type.

-   Example

    ```
    SET odps.sql.type.system.odps2=true;
    SELECT  EXTRACT(year FROM '2019-05-01 11:21:00') year
             ,EXTRACT(month FROM '2019-05-01 11:21:00') month
             ,EXTRACT(day FROM '2019-05-01 11:21:00') day
             ,EXTRACT(hour FROM '2019-05-01 11:21:00') hour
             ,EXTRACT(minute FROM '2019-05-01 11:21:00') minute;
    
    -- Return result:
    +------+-------+------+------+--------+
    | year | month | day  | hour | minute |
    +------+-------+------+------+--------+
    | 2019 | 5     | 1    | 11   | 21     |
    +------+-------+------+------+--------+
    ```


