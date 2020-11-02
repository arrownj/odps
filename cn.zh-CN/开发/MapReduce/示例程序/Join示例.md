---
keyword: [MapReduce, Join]
---

# Join示例

MaxCompute MapReduce框架自身并不支持Join逻辑，但您可以在自己的Map或Reduce函数中实现数据的Join。

## 测试准备

1.  准备好测试程序的JAR包，假设名字为mapreduce-examples.jar，本地存放路径为data\\resources。
2.  准备好Join的测试表和资源。
    1.  创建测试表。 对表mr\_Join\_src1与表mr\_Join\_src2执行Join操作，输出表mr\_Join\_out。

        ```
        create table mr_Join_src1(key bigint, value string);
        create table mr_Join_src2(key bigint, value string);
        create table mr_Join_out(key bigint, value1 string, value2 string);
        ```

    2.  添加测试资源。

        ```
        add jar data\resources\mapreduce-examples.jar -f;
        ```

3.  使用Tunnel导入数据。

    ```
    tunnel upload data1 mr_Join_src1;
    tunnel upload data2 mr_Join_src2;
    ```

    导入mr\_Join\_src1数据的内容。

    ```
     1,hello
     2,odps
    ```

    导入mr\_Join\_src2数据的内容。

    ```
    1,odps
    3,hello
    4,odps
    ```


## 测试步骤

在MaxCompute客户端中执行Join。

```
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.Join mr_Join_src1 mr_Join_src2 mr_Join_out;
```

## 预期结果

作业成功结束后，输出表mr\_Join\_out中的内容如下。 其中value1是mr\_Join\_src1的value值，value2是mr\_Join\_src2的value值。

```
+------------+------------+------------+
| key        | value1     | value2     |
+------------+------------+------------+
|  1         | hello      |  odps      | 
+------------+------------+------------+
```

## 代码示例

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
        /**reduce函数每次的输入会是key相同的所有record。*/
        @Override
            public void reduce(Record key, Iterator<Record> values, TaskContext context)
            throws IOException {
            long k = key.getBigint(0);
            List<Object[]> leftValues = new ArrayList<Object[]>();
            /**由于设置了outputKeySortColumn是key + tag组合，这样可以保证reduce函数的输入record中，left表的record数据在前面。*/
            while (values.hasNext()) {
                Record value = values.next();
                long tag = (Long) key.get(1);
                /**左表的数据会先缓存到内存中。*/
                if (tag == 0) {
                    leftValues.add(value.toArray().clone());
                } else {
                    /**右表的数据会与所有左表的数据进行join输出，此时左表的数据已经全部在内存里了。*/
                    /**这个实现只是一个功能展示，性能较低，不建议用于实际生产。*/
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
        if (args.length != 3) {
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

