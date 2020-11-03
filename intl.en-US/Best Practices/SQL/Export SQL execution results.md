---
keyword: export SQL execution results
---

# Export SQL execution results

This topic describes how to export SQL execution results in MaxCompute.

**Note:** This topic provides examples based on Alibaba Cloud MaxCompute SDK for Java.

## Overview

You can use the following methods to export the execution results of SQL statements:

-   If the amount of data is small, use [SQLTask](/intl.en-US/SDK Reference/Java SDK/Java SDK.md) to obtain all query results.
-   If you want to export the query results of a table or a partition, use [Tunnel](/intl.en-US/Development/Data upload and download/Run Tunnel commands to upload and download data/Tunnel commands.md).
-   If the SQL statements are complex, use Tunnel and SQLTask to export the query results.
-   Use [DataWorks](https://data.aliyun.com/product/ide?) to execute SQL statements, synchronize data, perform timed scheduling, and configure task dependencies.
-   Use the open source tool DataX to export data from MaxCompute to specified destination data sources.

## Use SQLTask to export data

SQLTask uses Alibaba Cloud MaxCompute SDK to call SQLTask.getResult\(i\) to execute SQL statements and obtain the query results. For more information, see [SQLTask](/intl.en-US/SDK Reference/Java SDK/Java SDK.md#section_fpg_45b_wdb).

When you use SQLTask, note the following rules:

-   SQLTask.getResult\(i\) is used to export the results of SELECT statements. You cannot use it to export the execution results of other MaxCompute SQL statements such as SHOW TABLES.
-   You can use READ\_TABLE\_MAX\_ROW to specify the maximum number of data records that the SELECT statement returns to a client. For more information, see [Specify project properties](/intl.en-US/Development/Common commands/Project operations.md).
-   The SELECT statement returns a maximum of 10,000 data records to a client. You can execute the SELECT statement on a client such as SQLTask. This is equivalent to appending a LIMIT N clause to the SELECT statement.

    This rule does not apply if you execute the CREATE TABLE XX AS SELECT or INSERT INTO/OVERWRITE TABLE statement to solidify the results into a specified table.


## Use Tunnel to export data

If a query returns all data of a table or a partition, you can use Tunnel to export the data. For more information, see [Tunnel commands](/intl.en-US/Development/Data upload and download/Run Tunnel commands to upload and download data/Tunnel commands.md) and [MaxCompute Tunnel overview](/intl.en-US/Development/Data upload and download/Tunnel SDK/MaxCompute Tunnel overview.md).

The following example shows how to run a Tunnel command to export data. If the Tunnel command cannot be used to export data, you can compile the Tunnel SDK to export data. For more information, see [MaxCompute Tunnel overview](/intl.en-US/Development/Data upload and download/Tunnel SDK/MaxCompute Tunnel overview.md).

```
tunnel d wc_out c:\wc_out.dat;
2016-12-16 19:32:08 - new session: 201612161932082d3c9b0a012f68e7 total lines: 3
2016-12-16 19:32:08 - file [0]: [0, 3), c:\wc_out.dat
downloading 3 records into 1 file
2016-12-16 19:32:08 - file [0] start
2016-12-16 19:32:08 - file [0] OK. total: 21 bytes
download OK
```

## Use SQLTask and Tunnel to export data

SQLTask cannot be used to process more than 10,000 data records, whereas Tunnel can. These two methods complement each other. You can use SQLTask and Tunnel together to export more than 10,000 data records.

The following sample code provides an example to show how to use SQLTask and Tunnel to export data:

```
private static final String accessId = "userAccessId";
    private static final String accessKey = "userAccessKey";
    private static final String endPoint = "http://service.cn-shanghai.maxcompute.aliyun.com/api";
    private static final String project = "userProject";
    private static final String sql = "userSQL";
    private static final String table = "Tmp_" + UUID.randomUUID().toString().replace("-", "_");// Use a random string as the name of the temporary table.
    private static final Odps odps = getOdps();

    public static void main(String[] args) {
        System.out.println(table);
        runSql();
        tunnel();
    }

    /*
     * Download the results that are returned by SQLTask.
     * */
    private static void tunnel() {
        TableTunnel tunnel = new TableTunnel(odps);
        try {
            DownloadSession downloadSession = tunnel.createDownloadSession(
                    project, table);
            System.out.println("Session Status is : "
                    + downloadSession.getStatus().toString());
            long count = downloadSession.getRecordCount();
            System.out.println("RecordCount is: " + count);
            RecordReader recordReader = downloadSession.openRecordReader(0,
                    count);
            Record record;
            while ((record = recordReader.read()) ! = null) {
                consumeRecord(record, downloadSession.getSchema());
            }
            recordReader.close();
        } catch (TunnelException e) {
            e.printStackTrace();
        } catch (IOException e1) {
            e1.printStackTrace();
        }
    }

    /*
     * Save the data.
     * If the amount of data is small, you can directly copy the data from the output. You can also use Java.io to write the data to a local file or a remote storage system to save the data.
     * */
    private static void consumeRecord(Record record, TableSchema schema) {
        System.out.println(record.getString("username")+","+record.getBigint("cnt"));
    }

    /*
     * Execute an SQL statement to save the query results to a temporary table.
     * The time-to-live (TTL) of the saved data is one day. Saved data does not consume much storage space. The storage of the system is not affected even if an error occurs when the system deletes the data.
     * */
    private static void runSql() {
        Instance i;
        StringBuilder sb = new StringBuilder("Create Table ").append(table)
                .append(" lifecycle 1 as ").append(sql);
        try {
            System.out.println(sb.toString());
            i = SQLTask.run(getOdps(), sb.toString());
            i.waitForSuccess();

        } catch (OdpsException e) {
            e.printStackTrace();
        }
    }

    /*
     * Initialize the connection information of MaxCompute.
     * */
    private static Odps getOdps() {
        Account account = new AliyunAccount(accessId, accessKey);
        Odps odps = new Odps(account);
        odps.setEndpoint(endPoint);
        odps.setDefaultProject(project);
        return odps;
    }
```

## Use DataWorks to synchronize and export data

DataWorks allows you to execute SQL statements and configure data synchronization tasks to generate and export data.

1.  Log on to the [DataWorks console](https://workbench.data.aliyun.com/console).
2.  In the left-side navigation pane, click **Workspaces**.
3.  On the Workspaces page, find the workspace that you want to manage and click **Data Analytics** in the Actions column.
4.  Create a business process.
    1.  On the Data Analytics page, right-click **Business process** and select **New business process**.
    2.  Enter a name in the **Business Name** field.
    3.  Click **New**.
5.  Create an ODPS SQL node.
    1.  Right-click the business process and choose **New** \> **MaxCompute** \> **ODPS SQL**.
    2.  Enter runsql in the **Node name** field and click **Submit**.
    3.  Configure the ODPS SQL node and click the **Save** icon.
6.  Create a data synchronization node.
    1.  Right-click the business process and choose **New** \> **Data Integration** \> **Offline synchronization**.
    2.  Enter sync2mysql in the **Node name** field and click **Submit**.
    3.  Specify a data source and a data destination.
    4.  Configure the mapping between columns in the source and destination tables.
    5.  Configure channel control.
    6.  Click the **Save** icon.
7.  Configure a dependency between the data synchronization node and the ODPS SQL node. Configure the ODPS SQL node as the output node and the data synchronization node as the export node.
8.  Configure workflow scheduling or use the default settings. Then, click the **Run** icon. The following information shows the operational log for data synchronization:

    ```
    2016-12-17 23:43:46.394 [job-15598025] INFO JobContainer - 
    Task start time: 2016-12-17 23:43:34
    Task end time: 2016-12-17 23:43:46
    Total execution time: 11s
    Average amounts of data per task: 31.36 KB/s
    Write speed: 1,668 rec/s
    Read records: 16,689
    Failed read and write attempts: 0
    ```

9.  Execute the following SQL statement to query the data synchronization results:

    ```
    select count(*) from result_in_db;
    ```


