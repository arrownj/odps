---
keyword: [Tunnel SDK, upload data]
---

# Simple upload

Tunnel SDK is a tunnel service provided by MaxCompute to upload and download large amounts of offline data.

## Procedure

1.  Create a TableTunnel object.
2.  Create an UploadSession object.
3.  Create a RecordWriter object and use it to write records.
4.  Commit the upload session.

## Example

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
import com.aliyun.odps.data.RecordWriter;
import com.aliyun.odps.tunnel.TableTunnel;
import com.aliyun.odps.tunnel.TunnelException;
import com.aliyun.odps.tunnel.TableTunnel.UploadSession;
public class UploadSample {
    private static String accessId = "<your access id>";
    private static String accessKey = "<your access Key>";
    private static String odpsUrl = "http://service.odps.aliyun.com/api";
    private static String tunnelUrl = "http://dt.cn-shanghai.maxcompute.aliyun-inc.com";
    // By default, data is transmitted on the Internet. If you need to transmit data on an internal network, you must set the tunnelUrl parameter based on your business needs.
    // In this example, the Tunnel endpoint of the classic network in the China (Shanghai) region is used.
    private static String project = "<your project>";
    private static String table = "<your table name>";
    private static String partition = "<your partition spec>";
    public static void main(String args[]) {
        // Prepare necessary resources. You only need to run the preceding code once.
        Account account = new AliyunAccount(accessId, accessKey);
        Odps odps = new Odps(account);
        odps.setEndpoint(odpsUrl);
        odps.setDefaultProject(project);
        try {
            TableTunnel tunnel = new TableTunnel(odps);
            // Set the tunnelUrl parameter.
        tunnel.setEndpoint(tunnelUrl);
            // Set the partition to which you want to upload data.
        PartitionSpec partitionSpec = new PartitionSpec(partition);
            // Create a session with a 24-hour lifecycle for the specified partition in the specified table on the server.
        // You can upload a total of 20,000 data blocks within the 24-hour lifecycle of the session.
        // It takes several seconds to create a session. In addition, a large number of resources are consumed and temporary directories are created on the server for each session that you create.
        // Therefore, we recommend that you upload data of the same partition in the same session.
            UploadSession uploadSession = tunnel.createUploadSession(project, table, partitionSpec);
            System.out.println("Session Status is : " + uploadSession.getStatus().toString());
            TableSchema schema = uploadSession.getSchema();
        // After data is prepared, create a Writer to start writing data to a block.
        // If a block is uploaded, you cannot upload it again. If the CloseWriter is successful, the block is uploaded. If the block upload fails, you can upload the block again.
        // A session contains a maximum of 20,000 block IDs, which are numbered 0 to 19999. If you have more blocks left after 20,000 blocks are uploaded in a session, commit the session. Then, create a new session to upload remaining blocks.
        // If you write less than 64 MB data to a block each time, many small files are generated, which poses negative impacts on the computing performance. We recommend that you write 64 MB or more data to a block each time. You can write a maximum of 100 GB data to a block.
        // If you upload only a small amount of data in a created session, many small files and empty directories are generated. What is worse, the upload performance deteriorates because you spend several seconds to create the session but maybe you upload only dozens of milliseconds of data in the session.
          // A Writer times out and is automatically disconnected if it writes less than 4 KB data in two consecutive minutes after it is created.
           // We recommend that you prepare writable data in the memory before you create the Writer.
            RecordWriter recordWriter = uploadSession.openRecordWriter(0);
            Record record = uploadSession.newRecord();
            for (int i = 0; i < schema.getColumns().size(); i++) {
                Column column = schema.getColumn(i);
                switch (column.getType()) {
                    case BIGINT:
                        record.setBigint(i, 1L);
                        break;
                    case BOOLEAN:
                        record.setBoolean(i, true);
                        break;
                    case DATETIME:
                        record.setDatetime(i, new Date());
                        break;
                    case DOUBLE:
                        record.setDouble(i, 0.0);
                        break;
                    case STRING:
                        record.setString(i, "sample");
                        break;
                    default:
                        throw new RuntimeException("Unknown column type: " + column.getType());
                }
            }
            for (int i = 0; i < 10; i++) {
        // Data is transmitted to the server when every 8 KB of data is written by the Writer.
        // If no data is transmitted within 120 seconds, the server automatically shuts down the connection to the Writer and the Writer becomes unavailable. In this case, you must create a new Writer to write data.
                recordWriter.write(record);
            }
            recordWriter.close();
            uploadSession.commit(new Long[]{0L});
            System.out.println("upload success!") ;
        } catch (TunnelException e) {
            // We recommend that you retry several times.
            e.printStackTrace();
        } catch (IOException e) {
            // We recommend that you retry several times.
            e.printStackTrace();
        }
    }
}
```

The following section describes a constructor example:

PartitionSpec\(String spec\): Use a string to construct an object of the PartitionSpec class.

Parameter description:

spec: the string used to define a partition, for example, pt='1',ds='2'.

Configure the following line in your code:

```
private static String partition = "pt='XXX',ds='XXX'";
```

**Note:** In this topic, the Tunnel endpoint of the classic network in the China \(Shanghai\) region is used. For more information about endpoints and regions, see [Configure endpoints](/intl.en-US/Prepare/Configure endpoints.md).

