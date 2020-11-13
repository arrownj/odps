---
keyword: [Quick BI, SQL Workbench/J, psql, Tableau Desktop]
---

# Use common tools to connect to MaxCompute Lightning

This topic describes how to use common client tools to connect to MaxCompute Lightning. You can also use the tools supported by PostgreSQL.

## Alibaba Cloud Quick BI

1.  Log on to the [Quick BI console](http://das.base.shuju.aliyun.com/console.htm). In the left-side navigation pane, click **Data Sources**.
2.  In the upper-right corner of the page that appears, click **Create Data Source**.
3.  In the dialog box that appears, click the Cloud Data Sources or User-created Data Sources tab. Then, click PostgeSQL to add a data source.
4.  In the dialog box that appears, configure the parameters and test connectivity.

    |Parameter|Description|
    |:--------|:----------|
    |Database Address|The endpoint of MaxCompute Lightning in the region. You can enter the public endpoint or the endpoint of the classic network or a virtual private cloud \(VPC\).|
    |Database|The name of the MaxCompute project that you want to access.|
    |Schema|The name of the MaxCompute project.|
    |Username/Password|The AccessKey ID and AccessKey secret of your Alibaba Cloud account.|
    |SSL|You must select this option.|
    |Test Connection|If the connection is normal, click OK to add the data source.|


## SQL Workbench/J

SQL Workbench/J is a popular, free, and cross-platform tool that is used to analyze SQL queries. You can use SQL Workbench/J to connect to MaxCompute Lightning based on the PostgreSQL driver.

1.  Download and install [SQL Workbench/J](http://www.sql-workbench.eu/downloads.html).
2.  Start SQL Workbench/J and create a database connection.

    Select the PostgreSQL driver, enter the MaxCompute Lightning URL of a MaxCompute project, and then enter the username and password. The username and password are the AccessKey ID and AccessKey secret of your Alibaba Cloud account.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7092659951/p11161.jpg)

    You can also click Extended Properties and set ssl to true.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/8628525061/p47189.jpg)

3.  After MaxCompute Lightning is connected, view, query, and analyze the table data of a MaxCompute project in the SQL Workbench/J workspace.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/8092659951/p11163.jpg)


## psql

The PostgreSQL client \(psql\) is a command line tool of PostgreSQL. By default, it is installed when you install the PostgreSQL database on your computer.

You can run commands on psql to connect to MaxCompute Lightning. The syntax for the connection to MaxCompute Lightning is the same as that for a PostgreSQL database.

```
psql -h <endpoint> -U <userid> -d <databasename> -p <port>
```

Parameters:

-   endpoint: the endpoint of MaxCompute Lightning. For more information, see [Endpoints of MaxCompute Lightning](/intl.en-US/Development/Interactive SQL (Lightning)/Endpoints of MaxCompute Lightning.md).
-   userid: the AccessKey ID of your Alibaba Cloud account.
-   databasename: the name of the MaxCompute project.
-   port: Set the value to 443.

After the command is executed, enter the AccessKey secret that corresponds to the AccessKey ID specified by userid in the prompt.

The following figure shows an example.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/8092659951/p11164.jpg)

**Note:** By default, psql uses SSL to connect to MaxCompute Lightning.

## Tableau Desktop

1.  Start Tableau Desktop, select the PostgreSQL data source, and configure the connection. When you configure the connection, select **SSL Connection**.
2.  After the logon, use Tableau to create a worksheet for visual analysis.

**Note:** By default, the TIME function returns the UTC time. If you want to return time in the time zone of UTC+08:00, use the following configuration:

```
SET timezone = 'Asia/Shanghai';
```

We recommend that you use TDC files supported by Tableau to customize the data sources that connect to MaxCompute Lightning. This improves the performance of MaxCompute Lightning. Procedure:

1.  Save the following content as a file named postgresql.tdc:

    ```
    <? xml version='1.0' encoding='utf-8' ? >
    <connection-customization class='postgres' enabled='true' version='8.10'>
    <vendor name='postgres'/>
    <driver name='postgres'/>
    <customizations>
    <customization name='CAP_CREATE_TEMP_TABLES' value='no' />
    <customization name='CAP_STORED_PROCEDURE_TEMP_TABLE_FROM_BUFFER' value='no' />
    <customization name='CAP_CONNECT_STORED_PROCEDURE' value='no' />
    <customization name='CAP_SELECT_INTO' value='no' />
    <customization name='CAP_SELECT_TOP_INTO' value='no' />
    <customization name='CAP_ISOLATION_LEVEL_SERIALIZABLE' value='yes' />
    <customization name='CAP_SUPPRESS_DISCOVERY_QUERIES' value='yes' />
    <customization name='CAP_SKIP_CONNECT_VALIDATION' value='yes' />
    <customization name='CAP_ODBC_TRANSACTIONS_SUPPRESS_EXPLICIT_COMMIT' value='yes' />
    <customization name='CAP_ODBC_TRANSACTIONS_SUPPRESS_AUTO_COMMIT' value='yes' />
    <customization name='CAP_ODBC_REBIND_SKIP_UNBIND' value='yes' />
    <customization name='CAP_FAST_METADATA' value='no' />
    <customization name='CAP_ODBC_METADATA_SUPPRESS_SELECT_STAR' value='yes' />
    <customization name='CAP_ODBC_METADATA_SUPPRESS_EXECUTED_QUERY' value='yes' />
    <customization name='CAP_ODBC_UNBIND_AUTO' value='yes' />
    <customization name='SQL_TXN_CAPABLE' value='0' />
    <customization name='CAP_ODBC_CURSOR_FORWARD_ONLY' value='yes' />
    <customization name='CAP_ODBC_TRANSACTIONS_COMMIT_INVALIDATES_PREPARED_QUERY' value='yes' />
    </customizations>
    </connection-customization>
    ```

2.  Save the file to the \\My Documents\\My Tableau Repository\\Datasources directory. If you use Tableau Server, save the file to C:\\ProgramData\\Tableau\\Tableau Server\\data\\tabsvc\\vizqlserver\\Datasources in Windows or /var/opt/tableau/tableau\_server/data/tabsvc/vizqlserver/Datasources/ in Linux.
3.  Restart Tableau and use the PostgreSQL data source to connect to MaxCompute Lightning. For more information about how to customize a data source by using a TDC file, see [Tableau documentation](https://onlinehelp.tableau.com/current/pro/desktop/en-us/odbc_customize.html#global_tdc).

## FineReport

1.  Start FineReport and choose **Server** \> **Define database connection**.
2.  Add a JDBC connection.

    The following table describes the parameters.

    |Parameter|Description|
    |:--------|:----------|
    |Databases|Set the value to Postgre.|
    |Driver|Set the value to org.postgresql.Driver built in FineReport.|
    |URL|`jdbc:postgresql://<MaxCompute Lightning Endpoint>:443/<Project_Name>?ssl=true&prepareThreshold=0`.

Example: `jdbc:postgresql://lightning.cn-shanghai.maxcompute.aliyun.com:443/lightning_demo?ssl=true&prepareThreshold=0`. |
    |Username/Password|The AccessKey ID and AccessKey secret of your Alibaba Cloud account.|


