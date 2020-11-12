---
keyword: TunnelBufferedWriter
---

# TunnelBufferedWriter

This topic describes TunnelBufferedWriter that is used to upload data.

Data upload procedure:

1.  Split data to data blocks.
2.  Call openRecordWriter\(id\) to specify an ID for each data block.
3.  Use one or more threads to upload the blocks. If a block fails to be uploaded, all the blocks are uploaded again.
4.  After all the blocks are uploaded, call session.commit\(\[ 1,2,3,…\]\) to provide the block IDs to the server for verification.

    The upload process is complex due to the limits on block management and connection timeouts on the server. Tunnel SDK provides an enhanced RecordWriter, TunnelBufferWriter, to simplify the upload process.


## Definition

```
public class TunnelBufferedWriter implements RecordWriter {
    public TunnelBufferedWriter(TableTunnel.UploadSession session, CompressOption option) throws IOException;
    public long getTotalBytes();
    public void setBufferSize(long bufferSize);
    public void setRetryStrategy(RetryStrategy strategy);
    public void write(Record r) throws IOException;
    public void close() throws IOException;
}
```

## Description

-   Lifecycle: from the time when the RecordWriter is created to the time when the data upload ends.
-   TunnelBufferedWriter instance: You can call openBufferedWriter of UploadSession to create a TunnelBufferedWriter instance.
-   Data upload: If you call Write, data entries are first written to the local cache. After the cache is full, multiple data entries are submitted to the server at a time to avoid a connection timeout. If the upload fails, the system automatically retries the upload operation.
-   Upload termination: Call close and then commit of UploadSession to terminate the upload process.
-   Buffer control: You can use setBufferSize to modify the memory occupied by the buffer \(in bytes\). We recommend that you set the memory to a value greater than or equal to 64 MB. This prevents the server from generating excessive small files, which may affect performance. The value ranges from 1 MB to 1000 MB. The default value is 64 MB.
-   Retry policy settings: The following policies are available: EXPONENTIAL\_BACKOFF, LINEAR\_BACKOFF, and CONSTANT\_BACKOFF. For example, EXPONENTIAL\_BACKOFF is used in the following code. The maximum number of retries is 6. The waiting time between retries starts from 4s and increases to 8s, 16s, 32s, 64s, and 128s in order. We recommend that you do not modify this configuration.

    ```
    RetryStrategy retry 
      = new RetryStrategy(6, 4, RetryStrategy.BackoffStrategy.EXPONENTIAL_BACKOFF)
    writer = (TunnelBufferedWriter) uploadSession.openBufferedWriter();
    writer.setRetryStrategy(retry);
    ```


