---
keyword: [SQLTask, Tunnel]
---

# Use SQLTask and Tunnel to export a large amount of data

This topic describes how to use SQLTask and Tunnel to export a large amount of data.

## Background information

-   SQL Task

    SQLTask is an SQL class of MaxCompute SDK for Java. SQLTask allows you to execute SQL statements and obtain the returned results. For more information, see the "SQLTask" section in [Overview](/intl.en-US/SDK Reference/Java SDK/Java SDK.md).

    The `SQLTask.getResult(i)` method returns a list. You can repeatedly use this method in an SQL statement to obtain the complete SQL computation result. However, if you use this method, the SELECT statement can return a maximum of 10,000 data entries to the MaxCompute client upon each query. For more information, see the valid values of READ\_TABLE\_MAX\_ROW in [Project operations](/intl.en-US/Development/Common commands/Project operations.md). That is, if you execute the SELECT statement on the MaxCompute client or by using the SQLTask SDK, you limit the number of data entries to return in each query result to 10,000. If you use the CREATE TABLE XX AS SELECT or INSERT INTO/OVERWRITE TABLE statement to insert the query result into a table, the number of data entries to return is not limited.

-   Tunnel

    If the query result to be exported is all data in a table or a specific partition, you can use the Tunnel command-line tool to export all data at a time. MaxCompute provides the Tunnel command-line tool and Tunnel SDK. For more information, see [Tunnel commands](/intl.en-US/Development/Data upload and download/Run Tunnel commands to upload and download data/Tunnel commands.md) and [MaxCompute Tunnel overview](/intl.en-US/Development/Data upload and download/Tunnel SDK/MaxCompute Tunnel overview.md).


## Export more than 10,000 data entries by using SQLTask and Tunnel

SQLTask cannot process more than 10,000 data entries in each query, but Tunnel can. Therefore, you can use SQLTask and Tunnel together to export more than 10,000 data entries. The following sample code is for your reference:

```
private static final String accessId = "userAccessId";
private static final String accessKey = "userAccessKey";
private static final String endPoint = "http://service.cn-shanghai.maxcompute.aliyun.com/api";
private static final String project = "userProject";
private static final String sql = "userSQL";
private static final String table = "Tmp_" + UUID.randomUUID().toString().replace("-", "_");// Use a random string as the name of the temporary table for storing the exported data.
private static final Odps odps = getOdps();
public static void main(String[] args) {
    System.out.println(table);
    runSql();
    tunnel();
}
/*
     * Download the query result that is returned by using SQLTask.
     * */
private static void tunnel() {
    TableTunnel tunnel = new TableTunnel(odps);
    try {
        DownloadSession downloadSession = tunnel.createDownloadSession(project, table);
        System.out.println("Session Status is : "+ downloadSession.getStatus().toString());
        long count = downloadSession.getRecordCount();
        System.out.println("RecordCount is: " + count);
        RecordReader recordReader = downloadSession.openRecordReader(0, count);
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
     * If only a small amount of data is returned, you can configure the client to display the query result and copy the data. In actual scenarios, you can also use the java.io package to write the query result to a local file or a file on the remote server.
     * */
private static void consumeRecord(Record record, TableSchema schema) {
    System.out.println(record.getString("username")+","+record.getBigint("cnt"));
}
/*
     * Execute the SQL statement and save the query result as a temporary table so that you can use Tunnel to download the result.
     * Set the lifecycle of the saved data to only one day. This prevents the data that fails to be deleted from occupying too much storage space.
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
     * Initialize the connection of the MaxCompute project.
     * */
private static Odps getOdps() {
    Account account = new AliyunAccount(accessId, accessKey);
    Odps odps = new Odps(account);
    odps.setEndpoint(endPoint);
    odps.setDefaultProject(project);
    return odps;
}
```

