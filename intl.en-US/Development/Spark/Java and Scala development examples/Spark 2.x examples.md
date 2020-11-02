---
keyword: Spark-2.x
---

# Spark 2.x examples

This topic describes how to configure Spark 2.x dependencies and provides some examples.

## Configure dependencies for Spark 2.x

If you want to submit your Spark 2.x application by using Spark on MaxCompute, you must add the following dependencies to the pom.xml file. For more information about pom.xml, see [pom.xml](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/pom.xml).

```
<properties>
    <spark.version>2.3.0</spark.version>
    <cupid.sdk.version>3.3.8-public</cupid.sdk.version>
    <scala.version>2.11.8</scala.version>
    <scala.binary.version>2.11</scala.binary.version>
</properties>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-mllib_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>cupid-sdk</artifactId>
    <version>${cupid.sdk.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>hadoop-fs-oss</artifactId>
    <version>${cupid.sdk.version}</version>
</dependency>
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-spark-datasource_${scala.binary.version}</artifactId>
    <version>${cupid.sdk.version}</version>
</dependency>
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-library</artifactId>
    <version>${scala.version}</version>
</dependency>
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-actors</artifactId>
    <version>${scala.version}</version>
</dependency>
```

In the preceding code, set the scope parameter based on the following instructions:

-   Set scope to provided for all packages that are released in the Apache Spark community, such as spark-core and spark-sql.
-   Set scope to compile for the odps-spark-datasource module.

## WordCount example

-   [WordCount.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/WordCount.scala)
-   How to submit

    ```
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.WordCount \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## GraphX PageRank example

-   [PageRank.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/graphx/PageRank.scala)
-   How to submit

    ```
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.graphx.PageRank \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## Mllib Kmeans-ON-OSS example

For more information about how to configure the `spark.hadoop.fs.oss.ststoken.roleArn` and `spark.hadoop.fs.oss.endpoint` parameters, see [OSS access notes](https://github.com/aliyun/MaxCompute-Spark/wiki/08.-Oss-Access%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E).

-   [KmeansModelSaveToOss.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/mllib/KmeansModelSaveToOss.scala)
-   How to submit

    ```
    # Edit code.
    val modelOssDir = "oss://bucket/kmeans-model" // Enter the path of the OSS bucket.
    val spark = SparkSession
      .builder()
      .config("spark.hadoop.fs.oss.credentials.provider", "org.apache.hadoop.fs.aliyun.oss.AliyunStsTokenCredentialsProvider")
      .config("spark.hadoop.fs.oss.ststoken.roleArn", "acs:ram::****:role/aliyunodpsdefaultrole")
      .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
      .appName("KmeansModelSaveToOss")
      .getOrCreate()
    
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.mllib.KmeansModelSaveToOss \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## OSS UnstructuredData example

For more information about how to configure the `spark.hadoop.fs.oss.ststoken.roleArn` and `spark.hadoop.fs.oss.endpoint` parameters, see [OSS access notes](https://github.com/aliyun/MaxCompute-Spark/wiki/08.-Oss-Access%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E).

-   [SparkUnstructuredDataCompute.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/oss/SparkUnstructuredDataCompute.scala)
-   How to submit

    ```
    # Edit code.
    val pathIn = "oss://bucket/inputdata/" // Enter the path of the OSS bucket.
    val spark = SparkSession
      .builder()
      .config("spark.hadoop.fs.oss.credentials.provider", "org.apache.hadoop.fs.aliyun.oss.AliyunStsTokenCredentialsProvider")
      .config("spark.hadoop.fs.oss.ststoken.roleArn", "acs:ram::****:role/aliyunodpsdefaultrole")
      .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
      .appName("SparkUnstructuredDataCompute")
      .getOrCreate()
    
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.oss.SparkUnstructuredDataCompute \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## MaxCompute table I/O example

-   [SparkSQL.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/sparksql/SparkSQL.scala)
-   How to submit

    ```
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.sparksql.SparkSQL \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## Example of using PySpark to read data from or write data to a MaxCompute table

-   [spark\_sql.py](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/python/spark_sql.py)
-   How to submit

    ```
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --jars /path/to/odps-spark-datasource_2.11-3.3.8-public.jar \
        /path/to/MaxCompute-Spark/spark-2.x/src/main/python/spark_sql.py
    ```


## Example of using PySpark to write data to OSS

-   [spark\_oss.py](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/python/spark_oss.py)
-   How to submit

    ```
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    # For more information about OSS configuration, see [OSS access notes](https://github.com/aliyun/MaxCompute-Spark/wiki/08.-Oss-Access%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E).
    
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --jars /path/to/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar \
        /path/to/MaxCompute-Spark/spark-2.x/src/main/python/spark_oss.py
    # Compile [Spark-2.x](https://github.com/aliyun/MaxCompute-Spark/tree/master/spark-2.x) to obtain the spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar package.
    ```


## Spark Streaming LogHub example

-   [LogHubStreamingDemo.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/streaming/loghub/LogHubStreamingDemo.scala)
-   How to submit

    ```
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.streaming.loghub.LogHubStreamingDemo \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## Spark Streaming DataHub example

-   [DataHubStreamingDemo.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/streaming/datahub/DataHubStreamingDemo.scala)
-   How to submit

    ```
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.streaming.datahub.DataHubStreamingDemo \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## Spark Streaming Kafka example

-   [KafkaStreamingDemo.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/streaming/kafka/KafkaStreamingDemo.scala)
-   How to submit

    ```
    # For more information about how to configure the environment variables in the spark-defaults.conf file, see [Set up a Spark on MaxCompute development environment](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83).
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.streaming.kafka.KafkaStreamingDemo \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


**Note:** For more information, see [MaxCompute-Spark](https://github.com/aliyun/MaxCompute-Spark/wiki).

## Example of reading data from MaxCompute and writing data to HBase

-   Sample code

    ```
    object McToHbase {
      def main(args: Array[String]) {
        val spark = SparkSession
          .builder()
          .appName("spark_sql_ddl")
          .config("spark.sql.catalogImplementation", "odps")
          .config("spark.hadoop.odps.end.point","http://service.cn.maxcompute.aliyun.com/api")
          .config("spark.hadoop.odps.runtime.end.point","http://service.cn.maxcompute.aliyun-inc.com/api")
          .getOrCreate()
          val sc = spark.sparkContext
          val config = HBaseConfiguration.create()
          val zkAddress = ""
          config.set(HConstants.ZOOKEEPER_QUORUM, zkAddress);
          val jobConf = new JobConf(config)
          jobConf.setOutputFormat(classOf[TableOutputFormat])
          jobConf.set(TableOutputFormat.OUTPUT_TABLE,"test")
    
        try{
          import spark._
          spark.sql("select '7', 'long'").rdd.map(row => {
            val id = row(0).asInstanceOf[String]
            val name = row(1).asInstanceOf[String]
            val put = new Put(Bytes.toBytes(id))
            put.addColumn(Bytes.toBytes("cf"), Bytes.toBytes("a"), Bytes.toBytes(name))
            (new ImmutableBytesWritable, put)
        }).saveAsHadoopDataset(jobConf)
      } finally {
        sc.stop()
      }
    
      }
    }
    ```

-   Use IntelliJ IDEA to submit and run the sample code.

    **Note:** For more information about how to configure the Spark development environment, see [Running methods of Spark on MaxCompute](https://yq.aliyun.com/articles/728374?spm=a2c4e.11155435.0.0.34a821883bxhvI).


## Examples of reading data from and writing data to OSS objects

-   Sample code
    -   Example 1: Sample code for the Local mode

        ```
        package com.aliyun.odps.spark.examples
        import java.io.ByteArrayInputStream
        import com.aliyun.oss.{ OSSClientBuilder,OSSClient}
        import org.apache.spark.sql.SparkSession
        
        object SparkOSS {
          def main(args: Array[String]) {
            val spark = SparkSession
              .builder()
              .config("spark.master", "local[4]") // The code can run after you set spark.master to local[N]. N indicates the number of concurrent Spark jobs.
              .config("spark.hadoop.fs.oss.accessKeyId", "")
              .config("spark.hadoop.fs.oss.accessKeySecret", "")
              .config("spark.hadoop.fs.oss.endpoint", "oss-cn-beijing.aliyuncs.com")
              .appName("SparkOSS")
              .getOrCreate()
        
            val sc = spark.sparkContext
            try {
              // Read data from OSS objects.
              val pathIn = "oss://spark-oss/workline.txt"
              val inputData = sc.textFile(pathIn, 5)
                    // Write Resilient Distributed Datasets (RDD).
              inputData.repartition(1).saveAsTextFile("oss://spark-oss/user/data3")
        
            } finally {
              sc.stop()
            }
          }
        }
        ```

    -   Example 2: Sample code for the Local mode

        ```
        package com.aliyun.odps.spark.examples
        import java.io.ByteArrayInputStream
        import com.aliyun.oss.{ OSSClientBuilder,OSSClient}
        import org.apache.spark.sql.SparkSession
        
        object SparkOSS {
          def main(args: Array[String]) {
            val spark = SparkSession
              .builder()
              .config("spark.master", "local[4]") // The code can run after you set spark.master to local[N]. N indicates the number of concurrent Spark jobs.
              .config("spark.hadoop.fs.oss.accessKeyId", "")
              .config("spark.hadoop.fs.oss.accessKeySecret", "")
              .config("spark.hadoop.fs.oss.endpoint", "oss-cn-beijing.aliyuncs.com")
              .appName("SparkOSS")
              .getOrCreate()
        
            val sc = spark.sparkContext
            try {
              // Read data from OSS objects.
              val pathIn = "oss://spark-oss/workline.txt"
              val inputData = sc.textFile(pathIn, 5)
              val cnt = inputData.count
              inputData.count()
              println(s"count: $cnt")
        
              // Write data to OSS objects.
              val ossClient = new OSSClientBuilder().build("oss-cn-beijing.aliyuncs.com", "<accessKeyId>", "<accessKeySecret>")
              val filePath="user/data"
              ossClient.putObject("spark-oss",filePath , new ByteArrayInputStream(cnt.toString.getBytes()))
              ossClient.shutdown()
            } finally {
              sc.stop()
            }
          }
        }
        ```

    -   Example 3: Sample code for the Cluster mode

        ```
        package com.aliyun.odps.spark.examples
        import java.io.ByteArrayInputStream
        import com.aliyun.oss.{ OSSClientBuilder,OSSClient}
        import org.apache.spark.sql.SparkSession
        
        object SparkOSS {
          def main(args: Array[String]) {
            val spark = SparkSession
              .builder()
              .appName("SparkOSS")
              .getOrCreate()
        
            val sc = spark.sparkContext
            try {
              // Read data from OSS objects.
              val pathIn = "oss://spark-oss/workline.txt"
              val inputData = sc.textFile(pathIn, 5)
              val cnt = inputData.count
              inputData.count()
              println(s"count: $cnt")
        
              // inputData.repartition(1).saveAsTextFile("oss://spark-oss/user/data3")
              // Write data to OSS objects.
              val ossClient = new OSSClientBuilder().build("oss-cn-beijing.aliyuncs.com", "<accessKeyId>", "<accessKeySecret>")
              val filePath="user/data"
              ossClient.putObject("spark-oss",filePath , new ByteArrayInputStream(cnt.toString.getBytes()))
              ossClient.shutdown()
            } finally {
              sc.stop()
            }
          }
        }
        ```

-   How to submit

    -   Use IntelliJ IDEA to develop, test, and submit the code for the Local mode.
    -   Use an ODPS Spark node to submit and run the code in the DataWorks console. For more information, see [t1693700.md\#]().
    **Note:** For more information about how to configure the Spark development environment, see [Running modes of Spark on MaxCompute](https://yq.aliyun.com/articles/728374?spm=a2c4e.11155435.0.0.34a821883bxhvI).


## Example of reading data from MaxCompute and writing data to OSS

-   Sample code
    -   Sample code for the Local mode

        ```
        package com.aliyun.odps.spark.examples.userpakage
        
        import org.apache.spark.sql.{ SaveMode, SparkSession}
        
        object SparkODPS2OSS {
          def main(args: Array[String]): Unit = {
            val spark = SparkSession
              .builder()
              .appName("Spark2OSS")
              .config("spark.master", "local[4]") // The code can run after you set spark.master to local[N]. N indicates the number of concurrent Spark jobs.
              .config("spark.hadoop.odps.project.name", "")
              .config("spark.hadoop.odps.access.id", "")
              .config("spark.hadoop.odps.access.key", "")
              .config("spark.hadoop.odps.end.point", "http://service.cn.maxcompute.aliyun.com/api")
              .config("spark.sql.catalogImplementation", "odps")
              .config("spark.hadoop.fs.oss.accessKeyId","")
              .config("spark.hadoop.fs.oss.accessKeySecret","")
              .config("spark.hadoop.fs.oss.endpoint","oss-cn-beijing.aliyuncs.com")
              .getOrCreate()
        
            try{
        
              // Use Spark SQL to query tables.
              val data = spark.sql("select * from  user_detail")
             // Present the query results.
              data.show(10)
              // Store the query results to an OSS object.
              data.toDF().coalesce(1).write.mode(SaveMode.Overwrite).csv("oss://spark-oss/user/data3")
            }finally {
              spark.stop()
            }
        
          }
        }
        ```

    -   Sample code for the Cluster mode

        ```
        package com.aliyun.odps.spark.examples.userpakage
        import org.apache.spark.sql.{ SaveMode, SparkSession}
        
        object SparkODPS2OSS {
          def main(args: Array[String]): Unit = {
            val spark = SparkSession
              .builder()
              .appName("SparkODPS2OSS")
              .getOrCreate()
        
            try{
        
              // Use Spark SQL to query tables.
              val data = spark.sql("select * from  user_detail")
             // Present the query results.
              data.show(10)
              // Store the query results to an OSS object.
              data.toDF().coalesce(1).write.mode(SaveMode.Overwrite).csv("oss://spark-oss/user/data3")
        
            }finally {
              spark.stop()
            }
        
          }
        }
        ```

-   How to submit

    -   Use IntelliJ IDEA to develop, test, and submit the code for the Local mode.
    -   Use an ODPS Spark node to submit and run the code in the DataWorks console. For more information, see [t1693700.md\#]().
    **Note:** For more information about how to configure the Spark development environment, see [Running modes of Spark on MaxCompute](https://yq.aliyun.com/articles/728374?spm=a2c4e.11155435.0.0.34a821883bxhvI).


