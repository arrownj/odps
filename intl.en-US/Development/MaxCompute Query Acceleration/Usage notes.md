# Usage notes

MaxCompute Query Acceleration \(MCQA\) is a built-in feature of MaxCompute. MCQA uses the native MaxCompute SQL language and supports MaxCompute built-in functions and permission systems. This topic describes how to use the MCQA feature.

You can use one of the following methods to enable the MCQA feature:

-   Use the MaxCompute client. For more information, see [Enable MCQA on the MaxCompute client](#section_no8_r2v_vjh).
-   Use the DataWorks ad hoc query or data analytics feature. By default, the MCQA feature is enabled for DataWorks ad hoc queries or data analytics. For more information, see [Enable MCQA for DataWorks ad hoc queries or data analytics](#section_xrw_x3d_9ey).
-   Use the MaxCompute JDBC driver. For more information, see [JDBC](#section_aa3_pz9_z7v).
-   Use an Alibaba Cloud MaxCompute SDK. This method requires POM dependency. For more information, see [Enable MCQA based on Alibaba Cloud MaxCompute SDK for Java](#section_osk_mpx_11n).

## Enable MCQA on the MaxCompute client

To enable the MCQA feature on the MaxCompute client, perform the following steps:

1.  Download the latest version of the MaxCompute client odpscmd on the [Client](https://odps-repo.oss-cn-hangzhou.aliyuncs.com/odpscmd/latest/odpscmd_public.zip) page.

    **Note:** The client version must be V0.35.1 or later.

2.  Install and configure the client. For more information, see [Install and configure the odpscmd client](/intl.en-US/Prepare/Install and configure the odpscmd client.md).

3.  Modify the odps\_config.ini configuration file in the client installation directory conf, and add the following command at the end of the configuration file:

    ```
    enable_interactive_mode=true
    ```

4.  Run the MaxCompute client in the client installation directory bin. Run ./bin/odpscmd in Linux, and run ./bin/odpscmd.bat in Windows. If the following message appears, the MaxCompute client runs properly.

    ![Client runs properly](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/9388220061/p142338.png)

5.  Run a query job to verify that the MCQA feature is enabled.

    If the client interface returns results that contain the following message, the MCQA feature is enabled.

    ![Verify query acceleration](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/9388220061/p142345.png)


## Enable MCQA for DataWorks ad hoc queries or data analytics

MCQA feature is enabled for the DataWorks **Ad-Hoc Query** and **Manually Triggered Workflow** modules by default. You do not need to manually enable it. To disable the MCQA feature, [submit a ticket](https://workorder-intl.console.aliyun.com/).

Run a query job in the **Ad-Hoc Query** module. If the returned results contain the following message, the MCQA feature is enabled. For more information about the **ad hoc query** feature, see [Create an ad hoc query]().

![Ad-Hoc Query](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/1790749951/p142636.png)

Run a query job in the **Manually Triggered Workflow** module. If the returned results contain the following message, the MCQA feature is enabled. For more information about the **Manually Triggered Workflow** feature, see [Manage manually triggered workflows]().

![Manually Triggered Workflow](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/1790749951/p142637.png)

## JDBC

You can use the MaxCompute JDBC driver in the following situations:

-   Use the MaxCompute JDBC driver to connect to MaxCompute. For more information, see [Usage notes](/intl.en-US/JDBC Reference/Usage notes.md) in JDBC Reference. To enable the MCQA feature, you must modify the relevant configuration. For more information, see [Enable MCQA on MaxCompute JDBC](#section_t85_p8c_r2s).
-   Use the MaxCompute JDBC driver to connect to Tableau. Then, you can use Tableau to analyze MaxCompute data in a visualized manner. For more information, see [Configure MaxCompute JDBC on Tableau](/intl.en-US/JDBC Reference/Third-party tools integrated with JDBC/Configure MaxCompute JDBC on Tableau.md). To enable the MCQA feature, you must modify the relevant configuration. For more information, see [Enable MCQA on Tableau based on MaxCompute JDBC](#section_0gc_1pe_nm5).
-   Use the MaxCompute JDBC driver to connect to SQL Workbench/J. Then, you can use SQL Workbench/J to execute SQL statements on MaxCompute data. For more information, see [Configure MaxCompute JDBC on SQL Workbench/J](/intl.en-US/JDBC Reference/Third-party tools integrated with JDBC/Configure MaxCompute JDBC on SQL Workbench/J.md). To enable the MCQA feature, you must modify the relevant configuration. For more information, see [Enable MCQA on SQL Workbench/J based on MaxCompute JDBC](#section_afn_593_64k).

## Enable MCQA on MaxCompute JDBC

If you use the MaxCompute JDBC driver to connect to MaxCompute, perform the following steps to enable the MCQA feature. For more information about how to use the MaxCompute JDBC driver to connect to MaxCompute, see [Usage notes](/intl.en-US/JDBC Reference/Usage notes.md) in JDBC Reference.

1.  Download the [JDBC JAR package](http://mvnrepo.alibaba-inc.com/nexus/service/local/repositories/releases/content/com/aliyun/odps/odps-jdbc/3.1.5/odps-jdbc-3.1.5-jar-with-dependencies.jar) that supports the MCQA feature or download the [source code](https://github.com/aliyun/aliyun-odps-jdbc/blob/master/README.md) that can be compiled.

2.  Add the following dependency to the pom.xml file of your Maven project:

    ```
    <dependency>
      <groupId>com.aliyun.odps</groupId>
      <artifactId>odps-jdbc</artifactId>
      <version>3.2.0</version>
      <classifier>jar-with-dependencies</classifier>
    </dependency>
    ```

    **Note:** The version must be V3.2.0 or later.

3.  Create a Java program based on the source code and configure the required information. For more information, see [ODPS JDBC](https://github.com/aliyun/aliyun-odps-jdbc/blob/master/README.md).

    Specify the following information:

    ```
    String accessId = "your_access_id";
    String accessKey = "your_access_key";
    Connection conn = DriverManager.getConnection("jdbc:odps:http://service.odps.aliyun.com/api?project=<your_project_name>"&accessId&accessKey&charset=UTF-8&interactiveMode=true");
    Statement stmt = conn.createStatement();
    --Replace your_access_id and your_access_key with the AccessKey ID and AccessKey secret of your Alibaba Cloud account. Replace your_project_name with the name of the project that uses the MCQA feature.
    Connection conn = DriverManager.getConnection(conn, accessId, accessKey);
    Statement stmt = conn.createStatement();
    String tableName = "testOdpsDriverTable";
    stmt.execute("drop table if exists " + tableName);
    stmt.execute("create table " + tableName + " (key int, value string)");
    ```

    To enable the MCQA feature on the MaxCompute JDBC driver, you only need to modify the connection string `Connection conn` or the code.

    -   `Connection conn`: Add the `interactiveMode=true` property.
    -   Code: Add the `interactiveMode=true` property.

## Enable MCQA on Tableau based on MaxCompute JDBC

Add the `interactiveMode=true` property to enable the MCQA feature for a specified **server**. We recommend that you also add the `enableOdpsLogger=true` property to display logs. For more information, see [Configure MaxCompute JDBC on Tableau](/intl.en-US/JDBC Reference/Third-party tools integrated with JDBC/Configure MaxCompute JDBC on Tableau.md).

The following example shows how to configure a **server**:

```
http://service.cn-beijing.maxcompute.aliyun.com/api?project=****_beijing&interactiveMode=true&enableOdpsLogger=true
```

To perform operations on specific tables in a MaxCompute project, add the `table_list=table_name1, table_name2` property in the parameters of the **server**. Then, you specify the tables on which you want to perform Tableau operations. Separate tables with commas \(,\). An excessive number of tables may cause Tableau to open slowly. We recommend that you use this method to load only the required tables. The following example shows how to load specified tables in Tableau.

```
http://service.cn-beijing.maxcompute.aliyun.com/api?project=****_beijing&interactiveMode=true&enableOdpsLogger=true&table_list=orders,customers
```

For tables with a large number of partitions, we recommend that you do not use the data from all partitions as the data source. You can filter the required partitions or run customized SQL queries to obtain the required data.

## Enable MCQA on SQL Workbench/J based on MaxCompute JDBC

After you configure the MaxCompute JDBC driver, modify the JDBC URL that you specified on the profile configuration page. Then, you can use the MCQA feature on SQL Workbench/J. For more information about how to configure the profile, see [Configure MaxCompute JDBC on SQL Workbench/J](/intl.en-US/JDBC Reference/Third-party tools integrated with JDBC/Configure MaxCompute JDBC on SQL Workbench/J.md).

Specify the URL in the following format: `jdbc:odps:<MaxCompute_endpoint>? project=<MaxCompute_project_name>&accessId=<AccessKey ID>&accessKey=<AccessKey Secret>&charset=UTF-8&interactiveMode=true`, where:

-   maxcompute\_endpoint: the endpoint of the region where your MaxCompute service is deployed. For more information, see [Configure endpoints](/intl.en-US/Prepare/Configure endpoints.md).
-   maxcompute\_project\_name: the name of your MaxCompute project.
-   AccessKey ID: the AccessKey ID that has the permissions to access the specified project.
-   AccessKey Secret: the AccessKey secret that corresponds to the AccessKey ID.
-   charset=UTF-8: the character set encoding format.
-   interactiveMode: specifies whether to enable the MCQA feature. To enable the MCQA feature, set this parameter to `true`.

## Enable MCQA based on Alibaba Cloud MaxCompute SDK for Java

For more information about Alibaba Cloud MaxCompute SDK for Java, see [Overview](/intl.en-US/SDK Reference/Java SDK/Java SDK.md). You must configure the POM dependency by using Maven. The following sample code shows how to configure the POM dependency:

```
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>odps-sdk-core</artifactId>
  <version>0.35.5-public</version>
</dependency>
```

**Note:** The version must be V0.35.1 or later.

Create a Java program by using the following sample code:

```
import com.aliyun.odps.Odps;
import com.aliyun.odps.OdpsException;
import com.aliyun.odps.OdpsType;
import com.aliyun.odps.account.Account;
import com.aliyun.odps.account.AliyunAccount;
import com.aliyun.odps.data.Record;
import com.aliyun.odps.data.ResultSet;
import com.aliyun.odps.sqa.*;

import java.io.IOException;
import java.util.*;

public class SQLExecutorExample {

    public static void SimpleExample() {
        // Specify the account and project information.
        Account account = new AliyunAccount("your_access_id", "your_access_key");
        Odps odps = new Odps(account);
        odps.setDefaultProject("your_project_name");
        odps.setEndpoint("http://service.odps.aliyun.com/api");

        // Prepare to build an SQLExecutor.
        SQLExecutorBuilder builder = SQLExecutorBuilder.builder();

        SQLExecutor sqlExecutor = null;
        try {
            // run in offline mode or run in interactive mode
            if (false) {
                // Create an executor that runs offline SQL queries by default.
                sqlExecutor = builder.odps(odps).executeMode(ExecuteMode.OFFLINE).build();
            } else {
                // Create an executor that runs SQL queries in an accelerated manner by default.
                sqlExecutor = builder.odps(odps).executeMode(ExecuteMode.INTERACTIVE).build();
            }
            // You can pass special query settings.
            Map<String, String> queryHint = new HashMap<>();
            queryHint.put("odps.sql.mapper.split.size", "128");
            // Submit a query job. You can pass hints.
            sqlExecutor.run("select count(1) from test_table;", queryHint);

            // List some supported API operations that are used to query information.
            // UUID
            System.out.println("ExecutorId:" + sqlExecutor.getId());
            // Query the logview of the current query job.
            System.out.println("Logview:" + sqlExecutor.getLogView());
            // Query the instance on which the current query job is run. In the interactive mode, multiple query jobs may be run on the same instance.
            System.out.println("InstanceId:" + sqlExecutor.getInstance().getId());
            // Query the progress of the current query job. You can check the progress bar in the console.
            System.out.println("QueryStageProgress:" + sqlExecutor.getProgress());
            // Query the changelogs about the execution status for the current query job, such as rollback messages.
            System.out.println("QueryExecutionLog:" + sqlExecutor.getExecutionLog());

            // Provide two API operations to query results.
            if(false) {
                // Query the results of all query jobs. This is a synchronous operation and may occupy this thread until the query succeeds or fails.
                List<Record> records = sqlExecutor.getResult();
                printRecords(records);
            } else {
                // Query the ResultSet iterator of the query results. This is a synchronous operation and may occupy this thread until the query succeeds or fails.
                ResultSet resultSet = sqlExecutor.getResultSet();
                while (resultSet.hasNext()) {
                    printRecord(resultSet.next());
                }
            }

            // run another query
            sqlExecutor.run("select * from test_table;", new HashMap<>());
            if(false) {
                // Query all query results. This is a synchronous operation and may occupy this thread until the query succeeds or fails.
                List<Record> records = sqlExecutor.getResult();
                printRecords(records);
            } else {
                // Query the ResultSet iterator of the query results. This is a synchronous operation and may occupy this thread until the query succeeds or fails.
                ResultSet resultSet = sqlExecutor.getResultSet();
                while (resultSet.hasNext()) {
                    printRecord(resultSet.next());
                }
            }
        } catch (OdpsException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (sqlExecutor ! = null) {
                // Close the executor and release related resources.
                sqlExecutor.close();
            }
        }
    }

    // SQLExecutor can be reused by pool mode
    public static void ExampleWithPool() {
        // Specify the account and project information.
        Account account = new AliyunAccount("your_access_id", "your_access_key");
        Odps odps = new Odps(account);
        odps.setDefaultProject("your_project_name");
        odps.setEndpoint("http://service.odps.aliyun.com/api");

        // Run queries by using the connection pool.
        SQLExecutorPool sqlExecutorPool = null;
        SQLExecutor sqlExecutor = null;
        try {
            // Prepare the connection pool. Specify the connection pool size and the default execution mode.
            SQLExecutorPoolBuilder builder = SQLExecutorPoolBuilder.builder();
            builder.odps(odps)
                    .initPoolSize(1) // init pool executor number
                    .maxPoolSize(5)  // max executors in pool
                    .executeMode(ExecuteMode.INTERACTIVE); // run in interactive mode

            sqlExecutorPool = builder.build();
            // Obtain an executor from the connection pool. If no executor can be obtained from the connection pool, you can add executors. Make sure that the total number of executors does not exceed the upper limit.
            sqlExecutor = sqlExecutorPool.getExecutor();

            // Use the executor in the same way it is used in the preceding example.
            sqlExecutor.run("select count(1) from test_table;", new HashMap<>());
            System.out.println("InstanceId:" + sqlExecutor.getId());
            System.out.println("Logview:" + sqlExecutor.getLogView());

            List<Record> records = sqlExecutor.getResult();
            printRecords(records);
        } catch (OdpsException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            sqlExecutor.close();
        }
        sqlExecutorPool.close();
    }

    private static void printRecord(Record record) {
        for (int k = 0; k < record.getColumnCount(); k++) {

            if (k ! = 0) {
                System.out.print("\t");
            }

            if (record.getColumns()[k].getType().equals(OdpsType.STRING)) {
                System.out.print(record.getString(k));
            } else if (record.getColumns()[k].getType().equals(OdpsType.BIGINT)) {
                System.out.print(record.getBigint(k));
            } else {
                System.out.print(record.get(k));
            }
        }
    }

    private static void printRecords(List<Record> records) {
        for (Record record : records) {
            printRecord(record);
            System.out.println();
        }
    }

    public static void main(String args[]) {
        SimpleExample();
        ExampleWithPool();
    }
}
```

