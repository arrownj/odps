---
keyword: Spark-2.x
---

# Spark-2.x示例

本文为您介绍Spark-2.x依赖的配置以及Spark-2.x示例说明。

## 配置Spark-2.x的依赖

通过MaxCompute提供的Spark客户端提交应用时，需要在pom.xml文件中添加以下依赖。 pom.xml文件请参见[pom.xml](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/pom.xml)。

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

上述代码中Scope的定义如下：

-   spark-core、spark-sql等所有Spark社区发布的包，设置Scope为provided。
-   odps-spark-datasource设置Scope为compile。

## WordCount示例

-   [WordCount.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/WordCount.scala)
-   提交方式

    ```
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.WordCount \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## GraphX PageRank示例

-   [PageRank.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/graphx/PageRank.scala)
-   提交方式

    ```
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.graphx.PageRank \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## Mllib Kmeans-ON-OSS示例

`spark.hadoop.fs.oss.ststoken.roleArn`和`spark.hadoop.fs.oss.endpoint`的填写请参见[Oss-Access文档说明](https://github.com/aliyun/MaxCompute-Spark/wiki/08.-Oss-Access%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E)。

-   [KmeansModelSaveToOss.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/mllib/KmeansModelSaveToOss.scala)
-   提交方式

    ```
    # 编辑代码。
    val modelOssDir = "oss://bucket/kmeans-model" // 填写具体的OSS Bucket路径。
    val spark = SparkSession
      .builder()
      .config("spark.hadoop.fs.oss.credentials.provider", "org.apache.hadoop.fs.aliyun.oss.AliyunStsTokenCredentialsProvider")
      .config("spark.hadoop.fs.oss.ststoken.roleArn", "acs:ram::****:role/aliyunodpsdefaultrole")
      .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
      .appName("KmeansModelSaveToOss")
      .getOrCreate()
    
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.mllib.KmeansModelSaveToOss \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## OSS UnstructuredData示例

`spark.hadoop.fs.oss.ststoken.roleArn`和`spark.hadoop.fs.oss.endpoint`的填写请参见[Oss-Access文档说明](https://github.com/aliyun/MaxCompute-Spark/wiki/08.-Oss-Access%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E)。

-   [SparkUnstructuredDataCompute.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/oss/SparkUnstructuredDataCompute.scala)
-   提交方式

    ```
    # 编辑代码。
    val pathIn = "oss://bucket/inputdata/" // 填写具体的OSS Bucket路径。
    val spark = SparkSession
      .builder()
      .config("spark.hadoop.fs.oss.credentials.provider", "org.apache.hadoop.fs.aliyun.oss.AliyunStsTokenCredentialsProvider")
      .config("spark.hadoop.fs.oss.ststoken.roleArn", "acs:ram::****:role/aliyunodpsdefaultrole")
      .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
      .appName("SparkUnstructuredDataCompute")
      .getOrCreate()
    
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.oss.SparkUnstructuredDataCompute \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## MaxCompute Table读写示例

-   [SparkSQL.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/sparksql/SparkSQL.scala)
-   提交方式

    ```
    cd /path/to/MaxCompute-Spark/spark-2.x
    mvn clean package
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.sparksql.SparkSQL \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## MaxCompute Table读写PySpark示例

-   [spark\_sql.py](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/python/spark_sql.py)
-   提交方式

    ```
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --jars /path/to/odps-spark-datasource_2.11-3.3.8-public.jar \
        /path/to/MaxCompute-Spark/spark-2.x/src/main/python/spark_sql.py
    ```


## PySpark写OSS示例

-   [spark\_oss.py](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/python/spark_oss.py)
-   提交方式

    ```
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    # OSS相关配置请参见[Oss Access文档说明](https://github.com/aliyun/MaxCompute-Spark/wiki/08.-Oss-Access%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E)。
    
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --jars /path/to/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar \
        /path/to/MaxCompute-Spark/spark-2.x/src/main/python/spark_oss.py
    # spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar可以通过[Spark-2.x](https://github.com/aliyun/MaxCompute-Spark/tree/master/spark-2.x)编译得到。
    ```


## 支持Spark Streaming Loghub示例

-   [LogHubStreamingDemo.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/streaming/loghub/LogHubStreamingDemo.scala)
-   提交方式

    ```
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.streaming.loghub.LogHubStreamingDemo \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## 支持Spark Streaming Datahub示例

-   [DataHubStreamingDemo.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/streaming/datahub/DataHubStreamingDemo.scala)
-   提交方式

    ```
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.streaming.datahub.DataHubStreamingDemo \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


## 支持Spark Streaming Kafka示例

-   [KafkaStreamingDemo.scala](https://github.com/aliyun/MaxCompute-Spark/blob/master/spark-2.x/src/main/scala/com/aliyun/odps/spark/examples/streaming/kafka/KafkaStreamingDemo.scala)
-   提交方式

    ```
    # 环境变量spark-defaults.conf的配置请参见[搭建开发环境](https://github.com/aliyun/MaxCompute-Spark/wiki/02.-%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。
    cd $SPARK_HOME
    bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.streaming.kafka.KafkaStreamingDemo \
        /path/to/MaxCompute-Spark/spark-2.x/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
    ```


**说明：** 更多信息请参见[MaxCompute-Spark](https://github.com/aliyun/MaxCompute-Spark/wiki)。

## 从MaxCompute中读取数据写入HBase

-   示例代码

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

-   提交方式：通过IDEA提交并运行示例代码。

    **说明：** Spark开发环境的配置请参见[Spark在MaxCompute的运行方式](https://yq.aliyun.com/articles/728374?spm=a2c4e.11155435.0.0.34a821883bxhvI)。


## 读写OSS文件

-   示例代码
    -   示例1：Local模式下的示例代码。

        ```
        package com.aliyun.odps.spark.examples
        import java.io.ByteArrayInputStream
        import com.aliyun.oss.{OSSClientBuilder,OSSClient}
        import org.apache.spark.sql.SparkSession
        
        object SparkOSS {
          def main(args: Array[String]) {
            val spark = SparkSession
              .builder()
              .config("spark.master", "local[4]") // 需要设置spark.master为local[N]才能直接运行，N为并发数。
              .config("spark.hadoop.fs.oss.accessKeyId", "")
              .config("spark.hadoop.fs.oss.accessKeySecret", "")
              .config("spark.hadoop.fs.oss.endpoint", "oss-cn-beijing.aliyuncs.com")
              .appName("SparkOSS")
              .getOrCreate()
        
            val sc = spark.sparkContext
            try {
              //OSS文件的读取。
              val pathIn = "oss://spark-oss/workline.txt"
              val inputData = sc.textFile(pathIn, 5)
                    //RDD写入。
              inputData.repartition(1).saveAsTextFile("oss://spark-oss/user/data3")
        
            } finally {
              sc.stop()
            }
          }
        }
        ```

    -   示例2：Local模式下的示例代码。

        ```
        package com.aliyun.odps.spark.examples
        import java.io.ByteArrayInputStream
        import com.aliyun.oss.{OSSClientBuilder,OSSClient}
        import org.apache.spark.sql.SparkSession
        
        object SparkOSS {
          def main(args: Array[String]) {
            val spark = SparkSession
              .builder()
              .config("spark.master", "local[4]") // 需要设置spark.master为local[N]才能直接运行，N为并发数。
              .config("spark.hadoop.fs.oss.accessKeyId", "")
              .config("spark.hadoop.fs.oss.accessKeySecret", "")
              .config("spark.hadoop.fs.oss.endpoint", "oss-cn-beijing.aliyuncs.com")
              .appName("SparkOSS")
              .getOrCreate()
        
            val sc = spark.sparkContext
            try {
              //OSS文件的读取。
              val pathIn = "oss://spark-oss/workline.txt"
              val inputData = sc.textFile(pathIn, 5)
              val cnt = inputData.count
              inputData.count()
              println(s"count: $cnt")
        
              //OSS文件的写入。
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

    -   示例3：Cluster模式下的示例代码。

        ```
        package com.aliyun.odps.spark.examples
        import java.io.ByteArrayInputStream
        import com.aliyun.oss.{OSSClientBuilder,OSSClient}
        import org.apache.spark.sql.SparkSession
        
        object SparkOSS {
          def main(args: Array[String]) {
            val spark = SparkSession
              .builder()
              .appName("SparkOSS")
              .getOrCreate()
        
            val sc = spark.sparkContext
            try {
              //OSS文件的读取。
              val pathIn = "oss://spark-oss/workline.txt"
              val inputData = sc.textFile(pathIn, 5)
              val cnt = inputData.count
              inputData.count()
              println(s"count: $cnt")
        
              // inputData.repartition(1).saveAsTextFile("oss://spark-oss/user/data3")
              //OSS文件的写入。
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

-   提交方式：

    -   Local模式下的代码通过IDEA开发、测试并提交。
    -   在DataWorks上通过ODPS Spark节点提交并运行。详情请参见[t1693700.md\#]()。
    **说明：** Spark开发环境的配置请参见[Spark在MaxCompute的运行方式](https://yq.aliyun.com/articles/728374?spm=a2c4e.11155435.0.0.34a821883bxhvI)。


## 读MaxCompute写OSS

-   示例代码
    -   Local模式下的示例代码。

        ```
        package com.aliyun.odps.spark.examples.userpakage
        
        import org.apache.spark.sql.{SaveMode, SparkSession}
        
        object SparkODPS2OSS {
          def main(args: Array[String]): Unit = {
            val spark = SparkSession
              .builder()
              .appName("Spark2OSS")
              .config("spark.master", "local[4]")// 需设置spark.master为local[N]才能直接运行，N为并发数
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
        
              //通过SparkSql查询表
              val data = spark.sql("select * from  user_detail")
             //展示查询数据
              data.show(10)
              //将查询到的数据存储到一个OSS的文件中
              data.toDF().coalesce(1).write.mode(SaveMode.Overwrite).csv("oss://spark-oss/user/data3")
            }finally {
              spark.stop()
            }
        
          }
        }
        ```

    -   Cluster模式下的示例代码。

        ```
        package com.aliyun.odps.spark.examples.userpakage
        import org.apache.spark.sql.{SaveMode, SparkSession}
        
        object SparkODPS2OSS {
          def main(args: Array[String]): Unit = {
            val spark = SparkSession
              .builder()
              .appName("SparkODPS2OSS")
              .getOrCreate()
        
            try{
        
              //通过SparkSql查询表
              val data = spark.sql("select * from  user_detail")
             //展示查询数据
              data.show(10)
              //将查询到的数据存储到一个OSS的文件中
              data.toDF().coalesce(1).write.mode(SaveMode.Overwrite).csv("oss://spark-oss/user/data3")
        
            }finally {
              spark.stop()
            }
        
          }
        }
        ```

-   提交方式：

    -   Local模式下的代码通过IDEA开发、测试并提交。
    -   在DataWorks上通过ODPS Spark节点提交并运行。详情请参见[t1693700.md\#]()。
    **说明：** Spark开发环境的配置请参见[Spark在MaxCompute的运行方式](https://yq.aliyun.com/articles/728374?spm=a2c4e.11155435.0.0.34a821883bxhvI)。


