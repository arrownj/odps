---
keyword: use MaxCompute SDK for Java to return error logs
---

# Return error logs

This topic describes how to use MaxCompute SDK for Java to return error logs.

## Description

MaxCompute SDK for Java provides the abstract class RetryLogger. For more information, see the [MaxCompute SDK for Java documentation](http://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-core/0.29.11-oversea-public?spm=a2c4e.11153940.blogcont687000.12.2b19745bCdnCwm&file=0.29.11-oversea-public).

```
public static abstract class RetryLogger {
/**
 * The callback function that enables RestClient to retry after an error occurs.
 * @param e
 *     The error.
 * @param retryCount
 *     The number of retries.
 * @param retrySleepTime
 *     The retry interval.
 */
public abstract void onRetryLog(Throwable e, long retryCount, long retrySleepTime);
}
```

You can create a RetryLogger subclass and use `odps.getRestClient().setRetryLogger(new UserRetryLogger());` to return logs when the system initializes MaxCompute objects.

## Example

```
// init odps
odps.getRestClient().setRetryLogger(new UserRetryLogger());
// your retry logger
public class UserRetryLogger extends RetryLogger {
 @Override
 public void onRetryLog(Throwable e, long retryCount, long sleepTime) {
   if (e ! = null && e instanceof OdpsException) {
     String requestId = ((OdpsException) e).getRequestId();
             if (requestId ! = null) {
                   System.err.println(String.format(
           "Warning: ODPS request failed, requestID:%s, retryCount:%d, will retry in %d seconds.", requestId, retryCount, sleepTime));
       return;
     }
   }
   System.err.println(String.format(
       "Warning: ODPS request failed:%s, retryCount:%d, will retry in %d seconds.", e.getMessage(), retryCount, sleepTime));
 }
}
```

