---
keyword: Sleep
---

# Sleep samples

This topic describes how to use MapReduce Sleep.

## Prerequisites

1.  The JAR package of the test program is prepared. In this topic, the JAR package is named mapreduce-examples.jar and stored in the local path data\\resources.
2.  Test resources for SleepJob are prepared.

    ```
    add jar data\resources\mapreduce-examples.jar -f;
    ```


## Procedure

Run Sleep on the MaxCompute client.

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar 
com.aliyun.odps.mapred.open.example.Sleep 10;
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar 
com.aliyun.odps.mapred.open.example.Sleep 100;
```

## Expected result

The job runs normally. The run time of different sleep durations can be compared to determine the effect.

## Sample code

```
package com.aliyun.odps.mapred.open.example;
import java.io.IOException;
import com.aliyun.odps.mapred.JobClient;
import com.aliyun.odps.mapred.MapperBase;
import com.aliyun.odps.mapred.conf.JobConf;
public class Sleep {
  private static final String SLEEP_SECS = "sleep.secs";
  public static class MapperClass extends MapperBase {
    /** No data is not entered, the map function is not executed, and the related logic can be written only into setup. */
    @Override
    public void setup(TaskContext context) throws IOException {
      try {
        /** Obtain the number of sleep seconds set in jobconf. */
        Thread.sleep(context.getJobConf().getInt(SLEEP_SECS, 1) * 1000);
      } catch (InterruptedException e) {
        throw new RuntimeException(e);
      }
    }
  }
  public static void main(String[] args) throws Exception {
    if (args.length ! = 1) {
      System.err.println("Usage: Sleep <sleep_secs>");
      System.exit(-1);
    }
    JobConf job = new JobConf();
    job.setMapperClass(MapperClass.class);
    /** This instance is also a MapOnly job and the number of reducers must be set to 0. */
    job.setNumReduceTasks(0);
    /** The number of mappers must be specified by the user because there is no input table. */
    job.setNumMapTasks(1);
    job.set(SLEEP_SECS, args[0]);
    JobClient.runJob(job);
  }
}
```

