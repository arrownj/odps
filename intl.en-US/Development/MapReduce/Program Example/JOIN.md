---
keyword: [MapReduce, JOIN]
---

# JOIN

The MaxCompute MapReduce framework does not support JOIN operations. However, you can join data by using the custom map or reduce function.

## Preparation

1.  Prepare the JAR package of the test program. Assume that the JAR package in this topic is named mapreduce-examples.jar and locally saved in data\\resources.
2.  Prepare test tables and upload the JAR package to the specified MaxCompute project.
    1.  Create test tables. Perform the JOIN operation on the mr\_Join\_src1 and mr\_Join\_src2 tables and write the joined data to the mr\_Join\_out table.

        ```
        create table mr_Join_src1(key bigint, value string);
        create table mr_Join_src2(key bigint, value string);
        create table mr_Join_out(key bigint, value1 string, value2 string);
        ```

    2.  Upload the JAR package to the specified MaxCompute project.

        ```
        add jar data\resources\mapreduce-examples.jar -f;
        ```

3.  Run the tunnel upload command to import data to the mr\_Join\_src1 and mr\_Join\_src2 tables.

    ```
    tunnel upload data1 mr_Join_src1;
    tunnel upload data2 mr_Join_src2;
    ```

    The following data is imported to the mr\_Join\_src1 table:

    ```
     1,hello
     2,odps
    ```

    The following data is imported to the mr\_Join\_src2 table:

    ```
    1,odps
    3,hello
    4,odps
    ```


## Procedure

Perform the JOIN operation on the MaxCompute client.

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.Join mr_Join_src1 mr_Join_src2 mr_Join_out;
```

## Expected result

If the job succeeds, the following data is written to the mr\_Join\_out table, where, value1 indicates the value in the mr\_Join\_src1 table and value2 indicates the value in the mr\_Join\_src2 table:

```
+------------+------------+------------+
| key        | value1     | value2     |
+------------+------------+------------+
|  1         | hello      |  odps      | 
+------------+------------+------------+
```

## Sample code

```
package com.aliyun.odps.mapred.open.example;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import com.aliyun.odps.data.Record;
import com.aliyun.odps.data.TableInfo;
import com.aliyun.odps.mapred.JobClient;
import com.aliyun.odps.mapred.MapperBase;
import com.aliyun.odps.mapred.ReducerBase;
import com.aliyun.odps.mapred.conf.JobConf;
import com.aliyun.odps.mapred.utils.InputUtils;
import com.aliyun.odps.mapred.utils.OutputUtils;
import com.aliyun.odps.mapred.utils.SchemaUtils;
/**
     * Join, mr_Join_src1/mr_Join_src2(key bigint, value string), mr_Join_out(key
     * bigint, value1 string, value2 string)
     *
     */
public class Join {
    public static final Log LOG = LogFactory.getLog(Join.class);
    public static class JoinMapper extends MapperBase {
        private Record mapkey;
        private Record mapvalue;
        private long tag;
        @Override
            public void setup(TaskContext context) throws IOException {
            mapkey = context.createMapOutputKeyRecord();
            mapvalue = context.createMapOutputValueRecord();
            tag = context.getInputTableInfo().getLabel().equals("left") ? 0 : 1;
        }
        @Override
            public void map(long key, Record record, TaskContext context)
            throws IOException {
            mapkey.set(0, record.get(0));
            mapkey.set(1, tag);
            for (int i = 1; i < record.getColumnCount(); i++) {
                mapvalue.set(i - 1, record.get(i));
            }
            context.write(mapkey, mapvalue);
        }
    }
    public static class JoinReducer extends ReducerBase {
        private Record result = null;
        @Override
            public void setup(TaskContext context) throws IOException {
            result = context.createOutputRecord();
        }
        /** Each input of the reduce function is records that have the same key. */
        @Override
            public void reduce(Record key, Iterator<Record> values, TaskContext context)
            throws IOException {
            long k = key.getBigint(0);
            List<Object[]> leftValues = new ArrayList<Object[]>();
            /** Records are sorted based on the combination of the key and tag. This ensures that records in the mr_Join_src1 table are passed to the reduce function first when the reduce function performs the JOIN operation. */
            while (values.hasNext()) {
                Record value = values.next();
                long tag = (Long) key.get(1);
                /** Data in the mr_Join_src1 table is first cached in memory. */
                if (tag == 0) {
                    leftValues.add(value.toArray().clone());
                } else {
                    /** Data in the mr_Join_src2 table is joined with all data in the mr_Join_src1 table. */
                    /** The sample code has poor performance and is only used as an example. We recommend that you do not use the code in your production environment. */
                    for (Object[] leftValue : leftValues) {
                        int index = 0;
                        result.set(index++, k);
                        for (int i = 0; i < leftValue.length; i++) {
                            result.set(index++, leftValue[i]);
                        }
                        for (int i = 0; i < value.getColumnCount(); i++) {
                            result.set(index++, value.get(i));
                        }
                        context.write(result);
                    }
                }
            }
        }
    }
    public static void main(String[] args) throws Exception {
        if (args.length ! = 3) {
            System.err.println("Usage: Join <input table1> <input table2> <out>");
            System.exit(2);
        }
        JobConf job = new JobConf();
        job.setMapperClass(JoinMapper.class);
        job.setReducerClass(JoinReducer.class);
        job.setMapOutputKeySchema(SchemaUtils.fromString("key:bigint,tag:bigint"));
        job.setMapOutputValueSchema(SchemaUtils.fromString("value:string"));
        job.setPartitionColumns(new String[]{"key"});
        job.setOutputKeySortColumns(new String[]{"key", "tag"});
        job.setOutputGroupingColumns(new String[]{"key"});
        job.setNumReduceTasks(1);
        InputUtils.addTable(TableInfo.builder().tableName(args[0]).label("left").build(), job);
        InputUtils.addTable(TableInfo.builder().tableName(args[1]).label("right").build(), job);
        OutputUtils.addTable(TableInfo.builder().tableName(args[2]).build(), job);
        JobClient.runJob(job);
    }
}
```

