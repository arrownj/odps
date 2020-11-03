---
keyword: [create functions, drop functions, view the function list]
---

# Function operations

This topic describes how to create functions, drop functions, and view the function list.

You can use common commands to manage functions on the MaxCompute client. You can also use the visualized online data development tools of DataWorks to create and search for resources.

## Create a function

**Command format**

```
CREATE FUNCTION <function_name> AS <package_to_class> USING <resource_list>;
```

**Parameters**

-   function\_name: the name of the user-defined function \(UDF\) to create. Function names must be unique. Each function name can be used only once.

    **Note:** Generally, UDFs cannot overwrite system built-in functions. Only the project owner can overwrite built-in functions. If you use a UDF that overwrites a built-in function, warning information is included in the Summary parameter after the SQL statement is executed.

-   package\_to\_class: the package name. You must enclose the package name within single quotation marks \(' '\).
    -   For a Java UDF, specify this name as a fully qualified class name from the top-level package name to the UDF class name.
    -   For a Python UDF, specify this name in the format ofPython\_script\_name.class\_name.
-   resource\_list: the resource list that is used by the UDF.
    -   This resource list must include the resource that contains the UDF code. Make sure that this resource has been uploaded to MaxCompute.
    -   If the code reads resource files by using the Distributed Cache API, this resource list must also contain the list of resource files that are read by the UDF.
    -   The resource list consists of multiple resource names. Separate the resource names with commas \(,\) and enclose the resource list in single quotation marks \(' '\).
    -   To specify the project that contains the resource, write the expression as `<project_name>/resources/<resource_name>`.

**Examples**

-   Create the `my_lower` function. In this example, the Java UDF class `org.alidata.odps.udf.examples.Lower` is in my\_lower.jar.

    ```
    CREATE FUNCTION my_lower AS 'org.alidata.odps.udf.examples.Lower' USING 'my_lower.jar';
    ```

-   Create the `my_lower` function. In this example, the Python UDF class MyLower is in the pyudf\_test.py script of the `test_project` project.

    ```
    create function my_lower as 'pyudf_test.MyLower' using 'test_project/resources/pyudf_test.py';
    ```

-   Create the `test_udtf` function. In this example, the Java UDF class `com.aliyun.odps.examples.udf.UDTFResource` is in udtfexample1.jar. The function depends on the resource file file\_resource.txt, the table resource table\_resource1, and the archive resource test\_archive.zip.

    ```
    create function test_udtf as 'com.aliyun.odps.examples.udf.UDTFResource' using 'udtfexample1.jar, file_resource.txt, table_resource1,test_archive.zip';
    ```


## Drop a function

**Command format**

```
DROP FUNCTION <function_name>;
```

**Parameters**

function\_name: the name of an existing function.

**Examples**

```
DROP FUNCTION my_lower;
```

## View a function

**Command format**

```
DESC FUNCTION <function_name>;
```

**Parameters**

function\_name: the name of an existing function.

## View the function list

**Command format**

-   View all UDFs within the current project.

    ```
    LIST FUNCTIONS; 
    ```

-   View all UDFs within a specified project.

    ```
    LIST FUNCTIONS -p project_name;
    ```


