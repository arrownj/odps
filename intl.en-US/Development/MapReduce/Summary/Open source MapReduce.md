---
keyword: compatibility with open source MapReduce
---

# Open source MapReduce

This topic describes the background of the compatibility with open source MapReduce and shows you how to use the Hadoop MapReduce plug-in.

## Background

MaxCompute provides native MapReduce with a set of programming models and operations. The input and output of the operations are data in MaxCompute tables. The data is organized in the record format, which can demonstrate how the data is processed.

The operations of MaxCompute MapReduce differ from those of Hadoop MapReduce. To migrate Hadoop MapReduce jobs to MaxCompute, you must rewrite the MapReduce code, compile and debug the code by calling MaxCompute operations, compress the final code into a JAR package, and upload the package to the MaxCompute platform. This process is tedious and labor-intensive for development and testing. An ideal solution is that the Hadoop MapReduce code can be run in MaxCompute with little or no modification at all.

To achieve the ideal solution, MaxCompute provides a plug-in to adapt Hadoop MapReduce to MaxCompute MapReduce. The plug-in enables Hadoop MapReduce jobs to be compatible with MaxCompute at the binary level. You can set configurations without the need to modify the code. Then, run the original Hadoop MapReduce JAR packages on MaxCompute. The plug-in is in the test phase and does not support custom comparators or key types.

In the following example, the WordCount program is used to introduce the basic usage of the plug-in.

## Download the Hadoop MapReduce plug-in

Download the [Hadoop MapReduce plug-in](http://odps-repo.oss-cn-hangzhou.aliyuncs.com/openmr%2Fhadoop2openmr-1.0.jar) package named openmr\_hadoop2openmr-1.0.jar.

**Note:** The openmr\_hadoop2openmr-1.0.jar package contains the dependencies of Hadoop 2.7.2. To avoid version conflicts, do not include Hadoop dependencies in the JAR packages of your jobs.

## Prepare a JAR package for WordCount

Compile and export a JAR package named wordcount\_test.jar with the following source code of WordCount:

```
package com.aliyun.odps.mapred.example.hadoop;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import java.io.IOException;
import java.util.StringTokenizer;
public class WordCount {
    public static class TokenizerMapper
        extends Mapper<Object, Text, Text, IntWritable>{
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
        public void map(Object key, Text value, Context context
        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }
    public static class IntSumReducer
        extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();
        public void reduce(Text key, Iterable<IntWritable> values,
            Context context
        ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
                
```

## Prepare test data

1.  Execute the following statements to create the input table wc\_in and the output table wc\_out:

    ```
    create table if not exists wc_in(line string);
    create table if not exists wc_out(key string, cnt bigint);
    ```

2.  Run the Tunnel upload command to import sample data to the input table wc\_in.

    Import the following data of the data.txt file to wc\_in:

    ```
    hello maxcompute
    hello mapreduce
    ```

    Run the following command on the MaxCompute client to import the preceding data from data.txt to wc\_in:

    ```
    tunnel upload data.txt wc_in;
    ```


## Configure the mapping between HDFS file paths and MaxCompute tables

Configure the mapping between Hadoop Distributed File System \(HDFS\) file paths and MaxCompute tables in the wordcount-table-res.conf file. The following code is an example of the file:

```
{
  "file:/foo": {
    "resolver": {
      "resolver": "com.aliyun.odps.mapred.hadoop2openmr.resolver.TextFileResolver",
      "properties": {
          "text.resolver.columns.combine.enable": "true",
          "text.resolver.seperator": "\t"
      }
    },
    "tableInfos": [
      {
        "tblName": "wc_in",
        "partSpec": {},
        "label": "__default__"
      }
    ],
    "matchMode": "exact"
  },
  "file:/bar": {
    "resolver": {
      "resolver": "com.aliyun.odps.mapred.hadoop2openmr.resolver.BinaryFileResolver",
      "properties": {
          "binary.resolver.input.key.class" : "org.apache.hadoop.io.Text",
          "binary.resolver.input.value.class" : "org.apache.hadoop.io.LongWritable"
      }
    },
    "tableInfos": [
      {
        "tblName": "wc_out",
        "partSpec": {},
        "label": "__default__"
      }
    ],
    "matchMode": "fuzzy"
  }
}
```

Set the following parameters in the preceding configuration:

The preceding configuration is a JSON file that describes the mapping between HDFS file paths and MaxCompute tables. You must configure both the input and output. Each HDFS file path matches three configuration items: resolver, tableInfos, and matchMode.

-   resolver: specifies how to process data in the specified files. The following two built-in resolvers are available: com.aliyun.odps.mapred.hadoop2openmr.resolver.TextFileResolver and com.aliyun.odps.mapred.hadoop2openmr.resolver.BinaryFileResolver. After you specify the resolver, you must configure the required properties for the resolver to parse data.
    -   TextFileResolver: regards the input or output as plaintext if the data is of the plaintext type. When you configure an input resolver, you must configure the text.resolver.columns.combine.enable and text.resolver.seperator properties. If you set text.resolver.columns.combine.enable to true, all columns in the input table are combined into a single string based on the delimiter specified by text.resolver.seperator. Otherwise, the first two columns in the input table are used as the key and value fields.
    -   BinaryFileResolver: converts binary data into a data type that is supported by MaxCompute, such as BIGINT, BOOLEAN, or DOUBLE. When you configure an output resolver, you must configure the binary.resolver.input.key.class and binary.resolver.input.value.class properties. binary.resolver.input.key.class specifies the key type of the intermediate result, and binary.resolver.input.value.class specifies the value type.
-   tableInfos: specifies the MaxCompute table that corresponds to the specified HDFS file path. You can set only the tblName parameter. The values of the partSpec and label parameters must be the same as those in the preceding sample configuration.
-   matchMode: specifies the path matching mode. The valid values are exact and fuzzy. If you set this parameter to fuzzy, you can use a regular expression to match the HDFS input path in fuzzy mode.

## Submit a job

Run the following command on the MaxCompute client to submit a job. For more information about how to install and configure the MaxCompute client, see [Client](/intl.en-US/Tools and Downloads/Client.md).

```
jar -DODPS_HADOOPMR_TABLE_RES_CONF=./wordcount-table-res.conf -classpath hadoop2openmr-1.0.jar,wordcount_test.jar com.aliyun.odps.mapred.example.hadoop.WordCount /foo/bar;
```

**Note:**

-   wordcount-table-res.conf: the mapped configuration file configured with /foo/bar.
-   wordcount\_test.jar: the JAR package of your Hadoop MapReduce program.
-   com.aliyun.odps.mapred.example.hadoop.WordCount: the class name of the job that you want to run.
-   /foo/bar: the path on HDFS, which is mapped to wc\_in and wc\_out in the JSON configuration file.
-   After you configure the mapping, you must use the Data Integration service of DataWorks to import the HDFS input file to wc\_in for MapReduce computing, and export the result table wc\_out to your HDFS output directory /bar.
-   Before you run the preceding command, make sure that hadoop2openmr-1.0.jar, wordcount\_test.jar, and wordcount-table-res.conf have been stored in the current directory of the MaxCompute client. Otherwise, modify the configuration and the -classpath parameter in the preceding command as needed.

The following figure shows the running process.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0982659951/p1957.jpg)

After the job is run, you can view the result table wc\_out to check whether the job is successful and whether the results meet expectations.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0982659951/p1959.jpg)

