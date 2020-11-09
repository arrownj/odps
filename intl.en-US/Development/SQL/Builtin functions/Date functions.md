---
keyword: date functions
---

# Date functions

This topic describes the date functions provided by MaxCompute, which can be used in SQL statements.

## DATEADD

-   Syntax

    ```
    datetime dateadd(datetime date, bigint delta, string datepart)
    ```

-   Description

    Modifies the value of `date` based on a specified unit `datepart` and specified scope `delta`.

-   Parameters
    -   date: a value of the DATETIME type.

        If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.

    -   delta: a value of the BIGINT type, which indicates the interval to add to the specified part of the date value. If the value of `delta` is greater than 0, the date value is incremented. Otherwise, the date value is decremented.

        If the input value is of the STRING or DOUBLE type, it is implicitly converted into a value of the BIGINT type before calculation. If the input value is of another data type, an error is returned.

        **Note:**

        -   If you add or subtract an interval specified by `delta` at a date part, a carry or return at more significant date parts may occur. The year, month, hour, minute, and second parts are computed by using different numeral systems. The year part uses the base-10 numeral system. The month part uses the base-12 numeral system. The hour part uses the base-24 numeral system. The minute and second parts use the base-60 numeral system.
        -   If the DATEADD function adds an interval specified by `delta` to the month part of a date value and this operation does not cause an overflow of `day`, keep `day` unchanged. Otherwise, set `day` to the last day of the target month.
    -   datepart: the part you want to modify in the date value. The value is a constant of the STRING type. For example, `dd` indicates a day. An error is returned if the value is in an invalid format or is not a constant of the STRING type.

        The value of this parameter is specified in compliance with the rules of data type conversion between STRING and DATETIME values. That is, the value yyyy indicates that the DATEADD function adds an interval to the year part of the date value, whereas the value mm indicates that the DATEADD function adds an interval to the month part of the date value. For more information about the rules of data type conversion, see [Type conversion](/intl.en-US/Development/SQL/Data type conversion.md). Extended date formats are also supported, such as `-year`, `-month`, `-mon`, `-day`, and `-hour`.

-   Return value

    Returns a value of the DATETIME type. If any input parameter is set to NULL, NULL is returned.

-   Examples
    -   Example 1: Common use of DATEADD

        ```
        select dateadd(datetime '2005-02-28 00:00:00', 1, 'dd') ;
        -- The return value is 2005-03-01 00:00:00. After one day is added, the result is beyond the last day of February. The actual value is the first day of March.
        select dateadd(datetime '2005-02-28 00:00:00', -1, 'dd');
        -- The return value is 2005-02-27 00:00:00. 
        select dateadd(datetime '2005-02-28 00:00:00', 20, 'mm');
        -- The return value is 2006-10-28 00:00:00. After 20 months are added, the month overflows, and the year increases by 1.
        select dateadd(datetime '2005-02-28 00:00:00', 1, 'mm');
        -- The return value is 2005-03-28 00:00:00.
        select dateadd(datetime '2005-01-29 00:00:00', 1, 'mm');
        -- The return value is 2005-02-28 00:00:00. There are only 28 days in February, 2005. Therefore, the last day of February is returned.
        select dateadd(datetime '2005-03-30 00:00:00', -1, 'mm');
        -- The return value is 2005-02-28 00:00:00.
        ```

    -   Example 2: Use of DATEADD in which DATETIME is expressed as a constant

        In MaxCompute SQL statements, a value of the DATETIME type cannot be directly expressed as a constant. The following statement uses an incorrect expression of the DATETIME type:

        ```
        select dateadd(2005-03-30 00:00:00, -1, 'mm') from tbl1;
        ```

        To use a constant of the DATETIME type, refer to the following expression:

        ```
        -- Explicitly convert the constant of the STRING type to the DATETIME type.
        select dateadd(cast("2005-03-30 00:00:00" as datetime), -1, 'mm') from tbl1;
        ```


## DATE\_ADD

-   Syntax

    ```
    date date_add(date/timestamp/string startdate, bigint delta)
    ```

-   Description

    Determines the value of startdate based on the value of delta.

-   Parameters
    -   startdate: the start date. The value of the DATE, DATETIME, or STRING type is supported. An error is returned if the value is of another data type.

        If the input value is of the STRING type, it is implicitly converted into a value of the DATE type before calculation. The value of the STRING type is in the format of `'yyyy-mm-dd'`, for example, `'2019-12-27'`.

    -   delta: a value of the BIGINT type, which indicates the interval to add to the specified part of the date value. An error is returned if it is of another type. If the value of delta is greater than 0, the date value is incremented. Otherwise, the date value is decremented.
-   Return value

    Returns a value of the DATE type.

-   Examples

    ```
    set odps.sql.type.system.odps2=true;
    -- Enable the new data types introduced in MaxCompute V2.0. You need to commit this statement along with SQL statements.
    select date_add(datetime '2005-02-28 00:00:00', 1);
    -- The return value is 2005-03-01. After one day is added, the result is beyond the last day of February. The actual value is the first day of March.
    select date_add(date '2005-02-28', -1);
    -- The return value is 2005-02-27, which is the previous day of 2005-03-01.
    select date_add( '2005-02-28 00:00:00', 20);
    -- The return value is 2005-03-20.
    ```


## DATEDIFF

-   Syntax

    ```
    bigint datediff(datetime date1, datetime date2, string datepart)
    ```

-   Description

    Calculates the difference between date1 and date2. The difference is measured in the time unit specified by datepart.

-   Parameters
    -   datet1 and date2: values of the DATETIME type, which indicate the minuend and subtrahend. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.
    -   datepart: the time unit, which is a constant of the STRING type. It supports extended date formats. An error is returned if the value of datepart is in an invalid format or is not a constant of the STRING type.

        If you enable [new data types](/intl.en-US/Development/Data types/Data type editions.md) of MaxCompute, datepart can be left unspecified. By default, day is used.

        **Note:** This function omits the lower unit based on the unit specified by datepart and then calculates the result.

-   Return value

    Returns a value of the BIGINT type. If any input parameter is set to NULL, NULL is returned. A negative value is returned if date1 is earlier than date2.

-   Examples

    ```
    The start time is 2005-12-31 23:59:59 and the end time is 2006-01-01 00:00:00.
        datediff(end, start, 'dd') = 1
        datediff(end, start, 'mm') = 1
        datediff(end, start, 'yyyy') = 1
        datediff(end, start, 'hh') = 1
        datediff(end, start, 'mi') = 1
        datediff(end, start, 'ss') = 1
        datediff(datetime'2013-05-31 13:00:00', '2013-05-31 12:30:00', 'ss') = 1800
        datediff(datetime'2013-05-31 13:00:00', '2013-05-31 12:30:00', 'mi') = 30
    The start time is 2018-06-04 19:33:23.234 and the end time is 2018-06-04 19:33:23.250. Date values with milliseconds do not adopt the standard DATETIME type and therefore cannot be implicitly converted into the DATETIME type. In this case, explicit conversion is required.
        datediff(to_date('2018-06-04 19:33:23.250', 'yyyy-MM-dd hh:mi:ss.ff3'),to_date('2018-06-04 19:33:23.234', 'yyyy-MM-dd hh:mi:ss.ff3') , 'ff3') = 16
    ```


## DATEPART

-   Syntax

    ```
    bigint datepart(datetime date, string datepart)
    ```

-   Description

    Extracts the value of the specified datepart in date.

-   Parameters
    -   date: a value of the DATETIME type. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.
    -   datepart: a constant of the STRING type. It supports extended date formats. An error is returned if the value of datepart is in an invalid format or is not a constant of the STRING type.
-   Return value

    Returns a value of the BIGINT type. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    datepart(datetime'2013-06-08 01:10:00', 'yyyy')  =  2013
    datepart(datetime'2013-06-08 01:10:00', 'mm')  =  6
    ```


## DATETRUNC

-   Syntax

    ```
    datetime datetrunc (datetime date, string datepart)
    ```

-   Description

    Truncates a date value based on the specified datepart.

-   Parameters
    -   date: a value of the DATETIME type. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.
    -   datepart: a constant of the STRING type. It supports extended date formats. An error is returned if the value of `datepart` is in an invalid format or is not a constant of the STRING type.
-   Return value

    Returns a value of the DATETIME type. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    datetrunc(datetime'2011-12-07 16:28:46', 'yyyy') = 2011-01-01 00:00:00
    datetrunc(datetime'2011-12-07 16:28:46', 'month') = 2011-12-01 00:00:00
    datetrunc(datetime'2011-12-07 16:28:46', 'DD') = 2011-12-07 00:00:00
    ```


## GETDATE

-   Syntax

    ```
    datetime getdate()
    ```

-   Description

    Returns the current system time. MaxCompute uses UTC+8 as the standard time.

-   Return value

    Returns the current date and time of the DATETIME type.

    **Note:** In a MaxCompute SQL task that is executed in a distributed manner, `getdate` always returns a fixed value. The return value is an arbitrary time during the execution of the MaxCompute SQL task. The time is accurate to seconds. In MaxCompute V2.0, the time is accurate to milliseconds.


## ISDATE

-   Syntax

    ```
    boolean isdate(string date, string format)
    ```

-   Description

    Determines whether a date string can be converted into a date value in a specified format. If the date string can be converted into a date value in the specified format, TRUE is returned. Otherwise, FALSE is returned.

-   Parameters
    -   date: a value of the STRING type. If the input value is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type, it is implicitly converted into a value of the STRING type before calculation. If the input value is of another data type, an error is returned.
    -   format: a constant of the STRING type. It does not support extended date formats. If the input value is of another data type, an error is returned. If redundant format strings exist in format, the function determines whether the date string that corresponds to the first format string can be converted into a date value of the first format string. The rest strings are treated as delimiters. For example, `isdate("1234-yyyy", "yyyy-yyyy")` returns TRUE.
-   Return value

    Returns a value of the Boolean type. If any input parameter is set to NULL, NULL is returned.


## LASTDAY

-   Syntax

    ```
    datetime lastday(datetime date)
    ```

-   Description

    Returns the last day of the current month to which the date belongs. The value is accurate to days. The hour, minute, and second parts are expressed as `00:00:00`.

-   Parameter

    date: a value of the DATETIME type. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.

-   Return value

    Returns a value of the DATETIME type. If any input parameter is set to NULL, NULL is returned.


## TO\_DATE

-   Syntax

    ```
    datetime to_date(string date, string format)
    ```

-   Description

    Converts a date string in a specified format into a date value.

-   Parameters
    -   date: a date value of the STRING type, which indicates the date string you want to convert. If the input value is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type, it is implicitly converted into a value of the STRING type before calculation. If the input value is of another data type or an empty string, an error is returned.
    -   format: a constant of the STRING type, which indicates a date format. If the input value is of another data type, an error is returned. format does not support the extended date format. Other characters are omitted as invalid characters during parsing.

        The value of format must contain `yyyy`. Otherwise, an error is returned. If redundant format strings exist in format, the function converts the date string that corresponds to the first format string into a date value. The rest strings are treated as delimiters. For example, `to_date("1234-2234", "yyyy-yyyy")` returns `1234-01-01 00:00:00`.

        In the format, yyyy indicates a 4-digit year, mm indicates a 2-digit month, dd indicates a 2-digit day, hh indicates an hour in 24-hour display, mi indicates a 2-digit minute, ss indicates a 2-digit second, and ff3 indicates a 3-digit millisecond.

-   Return value

    Returns a value of the DATETIME type in the format of `yyyy-mm-dd hh:mi:ss`. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    to_date('Alibaba 2010-12*03', 'Alibaba yyyy-mm*dd') = 2010-12-03 00:00:00
    to_date('20080718', 'yyyymmdd') = 2008-07-18 00:00:00
    to_date('200807182030','yyyymmddhhmi') = 2008-07-18 20:30:00
    to_date('2008718', 'yyyymmdd') = null -- Invalid format, causing an exception.
    to_date('Alibaba 2010-12*3', 'Alibaba yyyy-mm*dd') = null -- Invalid format, causing an exception.
    to_date('2010-24-01', 'yyyy') = null -- Invalid format, causing an exception.
    to_date('20181030 15-13-12.345','yyyymmdd hh-mi-ss.ff3')=2018-10-30 15:13:12
    ```


## TO\_CHAR

-   Syntax

    ```
    string to_char(datetime date, string format)
    ```

-   Description

    Converts a date value of the DATETIME type into a string of the specified format.

-   Parameters
    -   date: a date value of the DATETIME type, which indicates the date value you want to convert. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned.
    -   format: a constant of the STRING type. An error is returned if the value is not a constant or not of the STRING type. In the format, the date format part is replaced with the data and other characters remain unchanged in the output.
-   Return value

    Returns a value of the STRING type. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    to_char(datetime'2010-12-03 00:00:00', 'Alibaba Finance yyyy-mm*dd') = 'Alibaba Finance 2010-12*03'
    to_char(datetime'2008-07-18 00:00:00', 'yyyymmdd') = '20080718' 
    to_char(datetime'Alibaba 2010-12*3', 'Alibaba yyyy-mm*dd') -- An error will be returned.
    to_char(datetime'2010-24-01', 'yyyy') -- An error will be returned.
    to_char(datetime'2008718', 'yyyymmdd') -- An error will be returned.
    ```


## UNIX\_TIMESTAMP

-   Syntax

    ```
    bigint unix_timestamp(datetime date)
    ```

-   Description

    Converts a date value of the DATETIME type into a value of the integer type in the UNIX format.

-   Parameter

    date: a date value of the DATETIME type. If the input value is of the STRING type, it is implicitly converted into a value of the DATETIME type before calculation. If the input value is of another data type, an error is returned. If you enable new data types, the implicit conversion will fail. In this case, you must use the CAST function for conversion, for example, `unix_timestamp(cast(... as datetime))`.

-   Return value

    Returns a date value of the BIGINT type in the UNIX format. If any input parameter is set to NULL, NULL is returned.

-   Example

    ```
    select unix_timestamp(datetime'2009-03-20 11:11:00'); -- The return value is 1237518660.
    ```


## FROM\_UNIXTIME

-   Syntax

    ```
    datetime from_unixtime(bigint unixtime)
    ```

-   Description

    Converts a date value of the BIGINT type in the UNIX format into a date value of the DATETIME type.

-   Parameter

    unixtime: a date value of the BIGINT type in the UNIX format. Its value is accurate to seconds. If the input value is of the STRING, DOUBLE, or DECIMAL type, it is implicitly converted into a value of the BIGINT type before calculation.

-   Return value

    Returns a value of the DATETIME type. If any input parameter is set to NULL, NULL is returned.

    **Note:** In HIVE-compatible mode where `set odps.sql.hive.compatible=true;` is executed, if the input value is of the STRING type, a date value of the STRING type is returned.

-   Example

    ```
    from_unixtime(123456789) = 1973-11-30 05:33:09
    ```


## WEEKDAY

-   Syntax

    ```
    bigint weekday (datetime date)
    ```

-   Description

    Returns the day of the week in which the specified date value falls.

-   Parameter

    date: a date value of the DATETIME type.

-   Return value

    Returns a value of the BIGINT type. If any input parameter is set to NULL, NULL is returned. Monday is treated as the first day of a week and its return value is 0. Otherdays in a week are numbered in ascending order starting from 0. The return value of Sunday is 6.


## WEEKOFYEAR

-   Syntax

    ```
    bigint weekofyear (datetime date)
    ```

-   Description

    Returns the week of the year in which the specified date value falls. Monday is treated as the first day of a week.

    **Note:** To determine whether a week belongs to a year or its next year, find the year in which more than four days of the week fall. If the week belongs to the year, it is treated as the last week of the year. If the week belongs to the next year, it is treated as the first week of the next year.

-   Parameter

    date: a date value of the DATETIME type.

-   Return value

    Returns a value of the BIGINT type. If any input parameter is set to NULL, NULL is returned.

-   Example

    ```
    select weekofyear(to_date("20141229", "yyyymmdd"));  
    -- Return result:
    +------------+
    | _c0        |
    +------------+
    | 1          |
    +------------+
    -- 20141229 indicates that the year is 2014, but most days of the week fall in year 2015. Therefore, the return value 1 indicates the first week of 2015.    
    select weekofyear(to_date("20141231", "yyyymmdd")); -- The return value is 1.  
    select weekofyear(to_date("20151229", "yyyymmdd")); -- The return value is 53.
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
    INT year(string date)
    ```

-   Description

    Returns the year in the format specified by date. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the STRING type. Its format must include `yyyy-mm-dd` and exclude redundant strings. Otherwise, NULL is returned.

-   Return value

    Returns a value of the INT type.

-   Examples

    ```
    year('1970-01-01 12:30:00') = 1970
    year('1970-01-01') = 1970
    year('70-01-01') = 70
    year('1970-01-01') = 1970
    year('1970/03/09') = null
    year(null) -- An error is returned.
    ```


## QUARTER

-   Syntax

    ```
    INT quarter (datetime/timestamp/string date)
    ```

-   Description

    Returns the quarter of the year in which the specified date value falls. The quarter is a number ranging from 1 to 4. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the DATETIME, TIMESTAMP, or STRING type. Its format must include `yyyy-mm-dd`. NULL is returned if the date value is of another data type.

-   Return value

    Returns a value of the INT type. If any input parameter is set to NULL, NULL is returned.

-   Examples

    ```
    quarter('1970-11-12 10:00:00') = 4
    quarter('1970-11-12') = 4
    ```


## MONTH

-   Syntax

    ```
    INT month(string date)
    ```

-   Description

    Returns the month in which the specified date value falls. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the STRING type. An error is returned if it is of another data type.

-   Return value

    Returns a value of the INT type.

-   Examples

    ```
    month('2014-09-01') = 9
    month('20140901') = null
    ```


## DAY

-   Syntax

    ```
    INT day(string date)
    ```

-   Description

    Returns the day in which the specified date value falls. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the STRING type. Its format can be `yyyy-mm-dd` or `yyyy-mm-dd hh:mi:ss`. An error is returned if it is of another data type.

-   Return value

    Returns a value of the INT type.

-   Examples

    ```
    day('2014-09-01') = 1
    day('20140901') = null
    ```


## DAYOFMONTH

-   Syntax

    ```
    INT dayofmonth(date)
    ```

-   Description

    Returns the day part of a date. For example, if the date is October 13 in 2017, 13 is returned after you run the command `int dayofmonth(2017-10-13)`. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the STRING type. An error is returned if it is of another data type.

-   Return value

    Returns a value of the INT type.

-   Examples

    ```
    dayofmonth('2014-09-01') = 1
    dayofmonth('20140901') = null
    ```


## HOUR

-   Syntax

    ```
    INT hour(string date)
    ```

-   Description

    Returns the hour part of the date.

-   Parameter

    date: a date value of the STRING type. An error is returned if it is of another data type. This function is an additional function of MaxCompute V2.0.

-   Return value

    Returns a value of the INT type.

-   Examples

    ```
    hour('2014-09-01 12:00:00') = 12
    hour('12:00:00') = 12
    hour('20140901120000') = null
    ```


## MINUTE

-   Syntax

    ```
    INT minute(string date)
    ```

-   Description

    Returns the minute part of a specified date. This function is an additional function of MaxCompute V2.0.

-   Parameter

    date: a date value of the STRING type. An error is returned if it is of another data type.

-   Return value

    Returns a value of the INT type.

-   Examples

    ```
    minute('2014-09-01 12:30:00') = 30
    minute('12:30:00') = 30
    minute('20140901120000') = null
    ```


## SECOND

-   Syntax

    ```
    INT second(string date)
    ```

-   Description

    Returns the second part of a date.

-   Parameter

    date: a date value of the STRING type. An error is returned if it is of another data type. This function is an additional function of MaxCompute V2.0.

-   Return value

    Returns a value of the INT type.

-   Examples

    ```
    second('2014-09-01 12:30:45') = 45
    second('12:30:45') = 45
    second('20140901123045') = null
    ```


## CURRENT\_TIMESTAMP

-   Syntax

    ```
    timestamp current_timestamp()
    ```

-   Description

    Returns the current timestamp. The return value is not fixed. This function is an additional function of MaxCompute V2.0.

-   Return value

    Returns a value of the TIMESTAMP type.

-   Example

    ```
    select current_timestamp(); -- The return value is '2017-08-03 11:50:30.661'.
    ```


## FROM\_UTC\_TIMESTAMP

-   Syntax

    ```
    timestamp from_utc_timestamp({any primitive type}*, string timezone)
    ```

-   Description

    Converts a UTC timestamp to a timestamp for a specified time zone. This function is an additional function of MaxCompute V2.0.

-   Parameters
    -   \{any primitive type\}\*: a value of the TIMESTAMP, DATETIME, TINYINT, SMALLINT, INT, or BIGINT type. If the value is of the TINYINT, SMALLINT, INT, or BIGINT type, the unit is milliseconds.
    -   timezone: the destination time zone to convert the timestamp, such as PST.

        This function supports only the Asia/Shanghai time zone and does not support the GMT+9 time zone.

-   Return value

    Returns a value of the TIMESTAMP type.

-   Examples

    ```
    SELECT from_utc_timestamp(1501557840000, 'PST'); -- The unit of the input parameter is milliseconds (ms), and the return value is 2017-08-01 04:24:00.
    SELECT from_utc_timestamp('1970-01-30 16:00:00','PST') ; -- The return value is 1970-01-30 08:00:00.0.
    SELECT from_utc_timestamp('1970-01-30','PST') ; -- The return value is 1970-01-29 16:00:00.0.
    ```


## ADD\_MONTHS

-   Syntax

    ```
    string add_months(string startdate, int nummonths)
    ```

-   Description

    Returns the date that is num\_months months later than startdate. This function is an additional function of MaxCompute V2.0.

-   Parameters
    -   startdate: a value of the STRING type. Its format must include `yyyy-mm-dd`. Otherwise, NULL is returned.
    -   num\_months: a value of the INT type.
-   Return value

    Returns a date value of the STRING type in the format of `yyyy-mm-dd`.

-   Examples

    ```
    add_months('2017-02-14',3) = '2017-05-14'
    add_months('17-2-14',3) = '0017-05-14'
    add_months('2017-02-14 21:30:00',3) = '2017-05-14'
    add_months('20170214',3) = null
    ```


## LAST\_DAY

-   Syntax

    ```
    string last_day(string date)
    ```

-   Description

    Returns the date of the last day of a month.

-   Parameter

    date: a date value of the STRING type in the format of `yyyy-MM-dd HH:mi:ss` or `yyyy-MM-dd`.

-   Return value

    Returns a date value of the STRING type in the format of `yyyy-mm-dd`.

-   Examples

    ```
    last_day('2017-03-04') = '2017-03-31'
    last_day('2017-07-04 11:40:00') = '2017-07-31'
    last_day('20170304') = null
    ```


## NEXT\_DAY

-   Syntax

    ```
    string next_day(string startdate, string week)
    ```

-   Description

    Returns the date of the first day that is later than startdate and matches the week value. That is, the date of the specified day in the next week.

-   Parameters
    -   startdate: a date value of the STRING type in the format of `yyyy-MM-dd HH:mi:ss` or `yyyy-MM-dd`.
    -   week: a value of the STRING type, which is the first two or three letters in the week name or the full week name, for example, MO, TUE, or FRIDAY.
-   Return value

    Returns a date value of the STRING type in the format of `yyyy-mm-dd`.

-   Examples

    ```
    next_day('2017-08-01','TU') = '2017-08-08'
    next_day('2017-08-01 23:34:00','TU') = '2017-08-08'
    next_day('20170801','TU') = null
    ```


## MONTHS\_BETWEEN

-   Syntax

    ```
    double months_between(datetime/timestamp/string date1, datetime/timestamp/string date2)
    ```

-   Description

    Returns the number of months between date1 and date2. This function is an additional function of MaxCompute V2.0.

-   Parameters
    -   date1: a date value of the DATETIME, TIMESTAMP, or STRING type in the format of `yyyy-MM-dd HH:mi:ss` or `yyyy-MM-dd`.
    -   date2: a date value of the DATETIME, TIMESTAMP, or STRING type in the format of `yyyy-MM-dd HH:mi:ss` or `yyyy-MM-dd`.
-   Return value

    Returns a value of the DOUBLE type.

    -   A positive value is returned if date1 is later than date2. A negative value is returned if date2 is later than date1.
    -   If both date1 and date2 correspond to the last days of two months, the return value is an integer that represents the number of months. Otherwise, the return value is computed by using the following formula: \(date1 - date2\)/31
-   Examples

    ```
    months_between('1997-02-28 10:30:00', '1996-10-30') = 3.9495967741935485
    months_between('1996-10-30','1997-02-28 10:30:00' ) = -3.9495967741935485
    months_between('1996-09-30','1996-12-31') = -3.0
    ```


## EXTRACT

-   Syntax

    ```
    INT EXTRACT(<datepart> from <timestamp>)
    ```

-   Description

    Extracts the date part specified by datepart from the date timestamp. This function is an additional function of MaxCompute V2.0.

-   Parameters
    -   datepart: a value that can be set to YEAR, MONTH, DAY, HOUR, or MINUTE.
    -   Timestamp: a date value of the TIMESTAMP type.
-   Return value

    Returns a value of the INT type.

-   Examples

    ```
    SET odps.sql.type.system.odps2=true;
    SELECT  extract(YEAR FROM '2019-05-01 11:21:00') year
             ,extract(MONTH FROM '2019-05-01 11:21:00') month
             ,extract(DAY FROM '2019-05-01 11:21:00') day
             ,extract(HOUR FROM '2019-05-01 11:21:00') hour
             ,extract(MINUTE FROM '2019-05-01 11:21:00') minute;
    
    -- Return result:
    +------+-------+------+------+--------+
    | year | month | day  | hour | minute |
    +------+-------+------+------+--------+
    | 2019 | 5     | 1    | 11   | 21     |
    +------+-------+------+------+--------+
    ```


