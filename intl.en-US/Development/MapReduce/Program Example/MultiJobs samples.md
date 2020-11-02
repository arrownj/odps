---
keyword: MultiJobs samples
---

# MultiJobs samples

This topic describes examples of running MultiJobs in MapReduce.

## Prerequisites

1.  The JAR package of a test program is prepared. In this example, the name of the JAR package is mapreduce-examples.jar, and the local storage path is data\\resources.
2.  The tables and resources are prepared to test the MultiJobs operation.
    1.  Create test tables.

        ```
        create table mr_empty (key string, value string);
        create table mr_multijobs_out (value bigint);
        ```

    2.  Add test resources.

        ```
        add table mr_multijobs_out as multijobs_res_table -f;
        add jar data\resources\mapreduce-examples.jar -f;
        ```


## Procedure

Run MultiJobs on the MaxCompute client.

```
jar -resources mapreduce-examples.jar,multijobs_res_table -classpath data\resources\mapreduce-examples.jar 
 com.aliyun.odps.mapred.open.example.MultiJobs mr_multijobs_out;
```

## Expected output

When the MultiJobs operation is complete, the following output is returned in the mr\_multijobs\_out table:

```
+------------+
| value      |
+------------+
| 0          |
+------------+
```

## Sample code

```
package com.aliyun.odps.mapred.open.example;
import java.io.IOException;
import java.util.Iterator;
import com.aliyun.odps.data.Record;
import com.aliyun.odps.data.TableInfo;
import com.aliyun.odps.mapred.JobClient;
import com.aliyun.odps.mapred.MapperBase;
import com.aliyun.odps.mapred.RunningJob;
import com.aliyun.odps.mapred.TaskContext;
import com.aliyun.odps.mapred.conf.JobConf;
import com.aliyun.odps.mapred.utils.InputUtils;
import com.aliyun.odps.mapred.utils.OutputUtils;
import com.aliyun.odps.mapred.utils.SchemaUtils;
/**
     * MultiJobs
     *
     * Running multiple job
     *
     **/
public class MultiJobs {
    public static class InitMapper extends MapperBase {
        @Override
            public void setup(TaskContext context) throws IOException {
            Record record = context.createOutputRecord();
            long v = context.getJobConf().getLong("multijobs.value", 2);
            record.set(0, v);
            context.write(record);
        }
    }
    public static class DecreaseMapper extends MapperBase {
        @Override
            public void cleanup(TaskContext context) throws IOException {
            /** Obtain the variable values that are defined in the main function from JobConf. */
            long expect = context.getJobConf().getLong("multijobs.expect.value", -1);
            long v = -1;
            int count = 0;
            /** Read the data from the output table of the previous job. */
            Iterator<Record> iter = context.readResourceTable("multijobs_res_table");
            while (iter.hasNext()) {
                Record r = iter.next();
                v = (Long) r.get(0);
                if (expect ! = v) {
                    throw new IOException("expect: " + expect + ", but: " + v);
                }
                count++;
            }
            if (count ! = 1) {
                throw new IOException("res_table should have 1 record, but: " + count);
            }
            Record record = context.createOutputRecord();
            v--;
            record.set(0, v);
            context.write(record);
            /** Set the counter. The counter value can be obtained in the main function after the job is completed. */
            context.getCounter("multijobs", "value").setValue(v);
        }
    }
    public static void main(String[] args) throws Exception {
        if (args.length ! = 1) {
            System.err.println("Usage: TestMultiJobs <table>");
            System.exit(1);
        }
        String tbl = args[0];
        long iterCount = 2;
        System.err.println("Start to run init job.") ;
        JobConf initJob = new JobConf();
        initJob.setLong("multijobs.value", iterCount);
        initJob.setMapperClass(InitMapper.class);
        InputUtils.addTable(TableInfo.builder().tableName("mr_empty").build(), initJob);
        OutputUtils.addTable(TableInfo.builder().tableName(tbl).build(), initJob);
        initJob.setMapOutputKeySchema(SchemaUtils.fromString("key:string"));
        initJob.setMapOutputValueSchema(SchemaUtils.fromString("value:string"));
        /** Explicitly set the number of reducers to 0 for Map-only jobs. */
        initJob.setNumReduceTasks(0);
        JobClient.runJob(initJob);
        while (true) {
            System.err.println("Start to run iter job, count: " + iterCount);
            JobConf decJob = new JobConf();
            decJob.setLong("multijobs.expect.value", iterCount);
            decJob.setMapperClass(DecreaseMapper.class);
            InputUtils.addTable(TableInfo.builder().tableName("mr_empty").build(), decJob);
            OutputUtils.addTable(TableInfo.builder().tableName(tbl).build(), decJob);
            /** Explicitly set the number of reducers to 0 for Map-only jobs. */
            decJob.setNumReduceTasks(0);
            RunningJob rJob = JobClient.runJob(decJob);
            iterCount--;
            /** If the specified number of iterations is reached, exit the loop. */
            if (rJob.getCounters().findCounter("multijobs", "value").getValue() == 0) {
                break;
            }
        }
        if (iterCount ! = 0) {
            throw new IOException("Job failed.") ;
        }
    }
}
```

