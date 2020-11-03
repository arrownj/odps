---
keyword: query geolocations of IP addresses
---

# Use MaxCompute to query geolocations of IP addresses

This topic describes how to use MaxCompute to query geolocations of IP addresses. To query geolocations of IP addresses, you can download an IP address geolocation library, upload the library to MaxCompute, create a user-defined function \(UDF\), and then execute an SQL statement.

-   [Activate MaxCompute](/intl.en-US/Prepare/Activate MaxCompute.md).
-   [Activate DataWorks](https://common-buy.aliyun.com/?commodityCode=dide_create_post#/buy).
-   Create a DataWorks on the workflow. This example uses DataWorks simple mode. For more information, see [create a workflow]().

You can send an HTTP request to call the API operation provided by the IP address geolocation library of Taobao to query the geolocation of an IP address. The API operation returns the geolocation information in a string, as shown in the following figure.

![Query API operation](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2486834061/p31905.png)

However, HTTP requests are not allowed in MaxCompute. You can query geolocations of IP addresses in MaxCompute by using one of the following methods:

-   Execute SQL statements to download IP address data from the IP address geolocation library of Taobao to a local computer. Then, send HTTP requests to query the data.

    **Note:** This method is inefficient. In addition, the query frequency must be less than 10 queries per second \(QPS\). Otherwise, query requests are rejected by the IP address geolocation library of Taobao.

-   Download the IP address geolocation library to a local computer. Then, query the geolocation information from the local library.

    **Note:** This method is inefficient and is not suitable when the data needs to be uploaded and analyzed in a data warehouse service.

-   Maintain an IP address geolocation library and upload it to MaxCompute regularly. Then, query the geolocation information in MaxCompute.

    **Note:** This method is efficient. However, you must maintain the IP address geolocation library regularly.


## Download an IP address geolocation library

1.  Obtain an IP address geolocation library. In this example, the sample IP address geolocation library [ipdata.txt.utf8](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/102762/cn_zh/1547530733280/ipdata.txt.utf8) is used. This IP address geolocation library is an incomplete library in UTF-8 format.

2.  Download the ipdata.txt.utf8 file. The following figure shows the data in the file.

    ![Check the data format](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/3511359951/p31907.png)

    The following content describes the data in the sample IP address geolocation library.

    -   The format of the data is UTF-8.
    -   The first two strings in a data record are the start IP address and end IP address of an IP address range, in the format of decimal integer literal. The third and fourth strings are equivalent to the first two strings, but are expressed in dotted decimal notation. The decimal integer literal format makes it easy to check whether an IP address is within a specific IP address range.
    **Note:** You can also use your own IP address geolocation library.


## Upload the IP address library

1.  Execute the following statement on the MaxCompute client to create the ipresource table that is used to store IP address geolocation data:

    ```
    DROP TABLE IF EXISTS ipresource ;
    
    CREATE TABLE IF NOT EXISTS ipresource 
    (
        start_ip BIGINT
        ,end_ip BIGINT
        ,start_ip_arg string
        ,end_ip_arg string
        ,country STRING
        ,area STRING
        ,city STRING
        ,county STRING
        ,isp STRING
    );
    ```

2.  Run the following Tunnel command to upload data in the ipdata.txt.utf8 file to the ipresource table:

    ```
    odps@ workshop_demo>tunnel upload D:/ipdata.txt.utf8 ipresource;
    ```

    In the preceding command, D:/ipdata.txt.utf8 is the local path of the ipdata.txt.utf8 file. For more information about the command, see [Tunnel commands](/intl.en-US/Development/Data upload and download/Run Tunnel commands to upload and download data/Tunnel commands.md).

    You can execute the following statement to check whether the data in the file is uploaded:

    ```
    --Count the number of the data records in the ipresource table.
    select count(*) from ipresource;
    ```

3.  Execute the following SQL statement to obtain the first 10 data records in the ipresource table:

    ```
    select * from ipresource limit 10;
    ```

    The following figure shows the result of the preceding SQL statement.

    ![View the sample data](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/3511359951/p31909.png)


## Create a UDF

1.  Go to the **DataStudio** page.

    1.  Login [DataWorks console](https://workbench.data.aliyun.com/console).

    2.  In the left-side navigation pane, click **Workspaces**.

    3.  In the top navigation bar, select the region where the target workspace resides. Find the target workspace and click **Data Analytics** in the Actions column.

2.  Create a Python resource.

    1.  Right-click the workflow and choose **new** \> **MaxCompute** \> **resource** \> **Python**.

    2.  In **create resource** dialog box, enter **resource Name** and Select **upload to ODPS**, click **confirm**.

    3.  Enter the following code in the code editor:

        ```
        from odps.udf import annotate
        @annotate("string->bigint")
        class ipint(object):
            def evaluate(self, ip):
                try:
                    return reduce(lambda x, y: (x << 8) + y, map(int, ip.split('.')))
                except:
                    return 0
        ```

    4.  Click **OK**.

3.  Create a function.

    1.  Right-click the workflow that you created and choose **Create** \> **MaxCompute** \> **Function**.

    2.  In the **Create Function** dialog box, set the **Function Name** parameter and click **Commit**.

        **Note:** If multiple MaxCompute compute engines are bound to the current workspace, you must select one from the **Engine Instance MaxCompute** drop-down list.

    3.  On the configuration tab of the function, set the parameters as required.

        ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2486834061/p100611.png)

        |Parameter|Description|
        |:--------|:----------|
        |**Function Type**|The type of the function. Valid values: **Mathematical Function**, **Aggregate Function**, **String Function**, **Date Function**, **Analytic Function**, and **Other**.|
        |**Engine Instance MaxCompute**|The MaxCompute engine that is bound to the current workspace. By default, you cannot change the engine.|
        |**Function Name**|The name of the function, that is, the name used to reference the function in SQL. The function name must be globally unique and cannot be changed after the function is created.|
        |**Owner**|The owner of the function. This parameter is automatically set.|
        |**Class Name**|Required. The name of the class that implements the function. **Note:** If the resource type is Python, enter the class name in the Python resource name.Class name format. Do not include the .py extension in the resource name. |
        |**Resources**|Required. The list of resources. You can search for existing resources in the current workspace in fuzzy match mode.Separate multiple resources with commas \(,\). |
        |**Description**|The description of the function.|
        |**Expression Syntax**|The syntax of the function, for example, `test`.|
        |**Parameter Description**|The description of supported input and output parameters.|
        |**Return Value**|Optional. The value to return. Example: 1.|
        |**Example**|Optional. The example of the function.|

4.  Click the ![Save icon](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6681659951/p103147.png) icon in the toolbar.

5.  Commit the function.

    1.  Click the ![Submit icon](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6681659951/p103148.png) icon in the toolbar to commit the function.

    2.  In the **Commit Node** dialog box, enter your comments in the **Change description** field.

    3.  Click **OK**.


## Execute an SQL statement to query the geolocation of an IP address

1.  Right-click the workflow and choose **new** \> **MaxCompute** \> **ODPS SQL**.

2.  In **create a node** dialog box, enter **node name**, and click **submit**.

3.  On the configuration tab of the ODPS SQL node, enter the following statement:

    ```
    select * from ipresource
    WHERE ipint('1.2.24.2') >= start_ip
    AND ipint('1.2.24.2') <= end_ip
    ```

4.  Click ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0311359951/p100471.png)icon to run the code.

5.  You can **operation Log** view the results.


