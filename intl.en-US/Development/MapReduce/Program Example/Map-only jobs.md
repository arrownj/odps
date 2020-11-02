---
keyword: map-only job example
---

# Map-only jobs

For map-only jobs, a mapper directly generates key-value pairs to MaxCompute tables. You only need to specify output tables instead of the key-value metadata for the output of a mapper.

## Preparations

1.  Prepare the JAR package of the test program. In this example, the JAR package is named mapreduce-examples.jar and saved in the data\\resources directory on your computer.
2.  Prepare test tables and resources that are used to run a map-only job.
    1.  Create tables.

        ```
        create table wc_in (key string, value string);
        create table wc_out(key string, cnt bigint);
        ```

    2.  Add resources.

        ```
        add jar data\resources\mapreduce-examples.jar -f;
        ```

3.  Use Tunnel to import data.

    ```
    tunnel upload data wc_in;
    ```

    The following data is imported to the wc\_in table:

    ```
     hello,odps
     hello,odps
    ```


## Procedure

Run a map-only job on the MaxCompute client.

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.MapOnly wc_in wc_out map
```

## Expected results

If the job succeeds, the wc\_out table contains the following data:

```
+------------+------------+
| key        | cnt        |
+------------+------------+
| hello      | 1          |
| hello      | 1          |
+------------+------------+
```

## Sample code

```
package com.aliyun.odps.mapred.open.example;
import java.io.IOException;
import com.aliyun.odps.data.Record;
import com.aliyun.odps.mapred.JobClient;
import com.aliyun.odps.mapred.MapperBase;
import com.aliyun.odps.mapred.conf.JobConf;
import com.aliyun.odps.mapred.utils.SchemaUtils;
import com.aliyun.odps.mapred.utils.InputUtils;
import com.aliyun.odps.mapred.utils.OutputUtils;
import com.aliyun.odps.data.TableInfo;
public class MapOnly {
    public static class MapperClass extends MapperBase {
        @Override
            public void setup(TaskContext context) throws IOException {
            boolean is = context.getJobConf().getBoolean("option.mapper.setup", false);
            // The main function executes the following logic only if option.mapper.setup is set to true in the jobconf file:
            if (is) {
                Record result = context.createOutputRecord();
                result.set(0, "setup");
                result.set(1, 1L);
                context.write(result);
            }
        }
        @Override
            public void map(long key, Record record, TaskContext context) throws IOException {
            boolean is = context.getJobConf().getBoolean("option.mapper.map", false);
            // The main function executes the following logic only if option.mapper.map is set to true in the jobconf file:
            if (is) {
                Record result = context.createOutputRecord();
                result.set(0, record.get(0));
                result.set(1, 1L);
                context.write(result);
            }
        }
        @Override
            public void cleanup(TaskContext context) throws IOException {
            boolean is = context.getJobConf().getBoolean("option.mapper.cleanup", false);
            // The main function executes the following logic only if option.mapper.cleanup is set to true in the jobconf file:
            if (is) {
                Record result = context.createOutputRecord();
                result.set(0, "cleanup");
                result.set(1, 1L);
                context.write(result);
            }
        }
    }
    public static void main(String[] args) throws Exception {
        if (args.length ! = 2 && args.length ! = 3) {
            System.err.println("Usage: OnlyMapper <in_table> <out_table> [setup|map|cleanup]");
            System.exit(2);
        }
        JobConf job = new JobConf();
        job.setMapperClass(MapperClass.class);
        // For map-only jobs, the number of reducers must be explicitly set to 0.
        job.setNumReduceTasks(0);
        // Configure information about input and output tables.
        InputUtils.addTable(TableInfo.builder().tableName(args[0]).build(), job);
        OutputUtils.addTable(TableInfo.builder().tableName(args[1]).build(), job);
        if (args.length == 3) {
            String options = new String(args[2]);
            // You can specify key-value pairs in the jobconf file, and use getJobConf of the context to query the configurations in a mapper.
            if (options.contains("setup")) {
                job.setBoolean("option.mapper.setup", true);
            }
            if (options.contains("map")) {
                job.setBoolean("option.mapper.map", true);
            }
            if (options.contains("cleanup")) {
                job.setBoolean("option.mapper.cleanup", true);
            }
        }
        JobClient.runJob(job);
    }
}
```

