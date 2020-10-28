---
keyword: [TableTunnel, DownloadSession]
---

# Simple download example

This topic describes how to use the MaxCompute Java SDK to download data.

## Use the DownloadSession operation of TableTunnel to download data

To download table data, perform the following steps:

1.  Create a TableTunnel object.
2.  Create a DownloadSession object.
3.  Create a RecordReader object to read records.

**Example**

```
import java.io.IOException;
import java.util.Date;
import com.aliyun.odps.Column;
import com.aliyun.odps.Odps;
import com.aliyun.odps.PartitionSpec;
import com.aliyun.odps.TableSchema;
import com.aliyun.odps.account.Account;
import com.aliyun.odps.account.AliyunAccount;
import com.aliyun.odps.data.Record;
import com.aliyun.odps.data.RecordReader;
import com.aliyun.odps.tunnel.TableTunnel;
import com.aliyun.odps.tunnel.TableTunnel.DownloadSession;
import com.aliyun.odps.tunnel.TunnelException;
public class DownloadSample {
     private static String accessId = "<your access id>";
     private static String accessKey = "<your access Key>";
     private static String odpsUrl = "http://service.odps.aliyun.com/api";
     private static String tunnelUrl = "http://dt.cn-shanghai.maxcompute.aliyun-inc.com";
     // Set the tunnelUrl parameter. By default, data is transmitted on the Internet. If you need to transmit data on an internal network, you must set the tunnelUrl parameter based on your business needs.
     private static String project = "<your project>";
     private static String table = "<your table name>";
     private static String partition = "<your partition spec>";
     public static void main(String args[]) {
         Account account = new AliyunAccount(accessId, accessKey);
         Odps odps = new Odps(account);
         odps.setEndpoint(odpsUrl);
         odps.setDefaultProject(project);
         TableTunnel tunnel = new TableTunnel(odps);
         tunnel.setEndpoint(tunnelUrl);// Set the tunnelUrl parameter.
         PartitionSpec partitionSpec = new PartitionSpec(partition);
           try {
                  DownloadSession downloadSession = tunnel.createDownloadSession(project, table,
                                      partitionSpec);
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
         private static void consumeRecord(Record record, TableSchema schema) {
                 for (int i = 0; i < schema.getColumns().size(); i++) {
                         Column column = schema.getColumn(i);
                         String colValue = null;
                         switch (column.getType()) {
                         case BIGINT: {
                                 Long v = record.getBigint(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case BOOLEAN: {
                                 Boolean v = record.getBoolean(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case DATETIME: {
                                 Date v = record.getDatetime(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case DOUBLE: {
                                 Double v = record.getDouble(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case STRING: {
                                 String v = record.getString(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         default:
                                 throw new RuntimeException("Unknown column type: "
                                                 + column.getType());
                         }
                         System.out.print(colValue == null ? "null" : colValue);
                         if (i ! = schema.getColumns().size())
                                 System.out.print("\t");
                 }
                 System.out.println();
         }
 }
```

**Note:**

In the preceding example, the Tunnel endpoint of the classic network in the China \(Shanghai\) region is used. For more information about endpoints and regions, see [Configure endpoints](/intl.en-US/Prepare/Configure endpoints.md).

In this example, the `System.out.println` statement is executed to print data to facilitate testing. In actual use, you can modify the code to write data to a text file.

## Use the DownloadSession operation of InstanceTunnel to download data

```
Odps odps = OdpsUtils.newDefaultOdps(); // Initialize a MaxCompute object.
    Instance i = SQLTask.run(odps, "select * from wc_in;");
    i.waitForSuccess();
    // Create an InstanceTunnel object.
    InstanceTunnel tunnel = new InstanceTunnel(odps);
    // Create a DownloadSession object based on the instance ID of the MaxCompute job.
    InstanceTunnel.DownloadSession session = tunnel.createDownloadSession(odps.getDefaultProject(), i.getId());
    long count = session.getRecordCount();
    // The number of records that are returned.
    System.out.println(count);
    // The method that is used to obtain data is the same as that in TableTunnel.
    TunnelRecordReader reader = session.openRecordReader(0, count);
    Record record;
    while ((record = reader.read()) ! = null) {
      for (int col = 0; col < session.getSchema().getColumns().size(); ++col) {
        // Print the data of the wc_in table. All fields of the wc_in table are strings.
        System.out.println(record.get(col));
      }
    }
    reader.close();
```

## Use the `SQLTask.getResultSet()` static method to download data

```
Odps odps = OdpsUtils.newDefaultOdps(); // Initialize a MaxCompute object.
    Instance i = SQLTask.run(odps, "select * from wc_in;");
    i.waitForSuccess();
    // Obtain the record iterator based on the instance ID.
    ResultSet rs = SQLTask.getResultSet(i);
    for (Record r : rs) {
      // The number of records that are returned.
      System.out.println(rs.getRecordCount());
      for (int col = 0; col < rs.getTableSchema().getColumns().size(); ++col) {
        // Print the data of the wc_in table. All fields of the wc_in table are strings.
        System.out.println(r.get(col));
      }
    }
```

