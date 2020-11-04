---
keyword: merge multiple rows of data into one row
---

# Merge multiple rows of data into one row

This topic describes how to use SQL statements to merge multiple rows of data into one row.

## Sample data

|class|gender|name|
|:----|:-----|:---|
|1|M|LiLei|
|1|F|HanMM|
|1|M|Jim|
|2|F|Kate|
|2|M|Peter|

## Examples

-   Example 1: Execute the following statement to concatenate the values in the name column into a single row based on the values in the class column:

    ```
    SELECT class, wm_concat(',', name) FROM students GROUP BY class;
    ```

    **Note:** The wm\_concat function is used to aggregate data. For more information, see [WM\_CONCAT](/intl.en-US/Development/SQL/Builtin functions/Aggregate functions.md).

    The following result is returned:

    |class|names|
    |:----|:----|
    |1|LiLei,HanMM,Jim|
    |2|Kate,Peter|

-   Example 2: Execute the following statement to count the numbers of males and females based on the values in the class column:

    ```
    SELECT 
    class
    ,SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) AS cnt_m
    ,SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS cnt_f
    FROM students
    GROUP BY class;
    ```

    The following result is returned:

    |class|cnt\_m|cnt\_f|
    |:----|:-----|:-----|
    |1|2|1|
    |2|1|1|


