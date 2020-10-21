---
keyword: MapReduce
---

# 编写MapReduce（可选）

本文将为您介绍安装好MaxCompute客户端后，如何快速编写和运行MapReduce WordCount示例程序。

## 前提条件

-   编写、编译、运行MapReduce前，需要首先安装JDK 1.6或以上版本。

    **说明：** 如果您使用Maven，可以从[Maven库](http://search.maven.org/)中搜索odps-sdk-mapred获取不同版本的Java SDK，相关配置信息如下。

    ```
    <dependency>
        <groupId>com.aliyun.odps</groupId>
        <artifactId>odps-sdk-mapred</artifactId>
        <version>0.26.2-public</version>
    </dependency>
    ```

-   请参见[安装并配置客户端](/cn.zh-CN/准备工作/安装并配置客户端.md)对MaxCompute客户端进行部署。更多关于MaxCompute客户端的使用，请参见[MaxCompute客户端](/cn.zh-CN/工具及下载/客户端.md)。
-   确保您购买的资源非[按量计费开发者版](/cn.zh-CN/规格类型/按量计费开发者版.md)。按量计费开发者版资源，仅支持MaxCompute SQL（支持使用UDF）、PyODPS作业，暂不支持MapReduce、Spark等其它作业。

## 操作步骤

1.  安装并配置好客户端后，运行bin目录下的MaxCompute客户端（Linux系统下运行./bin/odpscmd，Windows下运行./bin/odpscmd.bat），进入相应项目空间中。
2.  输入建表语句，创建输入和输出表，如下所示。

    ```
    --创建输入表wc_in。
    CREATE TABLE wc_in (key STRING, value STRING);
    --创建输出表wc_out。
    CREATE TABLE wc_out (key STRING, cnt BIGINT);
    ```

    更多创建表的语句请参见[创建表](/cn.zh-CN/开发/常用命令/表操作.md)。

3.  在表wc\_in中插入数据。您可以通过以下两种方式插入数据：
    -   使用Tunnel命令上传数据。

        需要插入的数据如下所示，请在本地创建kv.txt文件保存数据，假设kv.txt文件本地存放路径为D:\\。

        ```
        238,val_238
        186,val_86
        186,val_86
        ```

        执行如下命令，上传数据。

        ```
        Tunnel upload D:\kv.txt wc_in;
        ```

    -   执行如下SQL语句直接插入数据。

        ```
        INSERT INTO TABLE wc_in VALUES ('238',' val_238'),('186','val_86'),('186','val_86');
        ```

4.  开发MapReduce程序并上传MaxCompute。

    在Eclipse或MaxCompute Studio中创建一个项目工程，并在此工程中编写MapReduce程序。本地调试通过后，将编译好的程序（JAR包，例如Word-count-1.0.jar）导出并上传至MaxCompute。

    **说明：** 本文中，您直接使用[WordCount示例](/cn.zh-CN/开发/MapReduce/示例程序/WordCount示例.md)中的示例代码生成Word-count-1.0.jar包即可，无需自己开发。

5.  在MaxCompute客户端，将JAR包添加为项目空间资源（例如，此处的JAR包名为word-count-1.0.jar）。

    ```
    ADD JAR word-count-1.0.jar;
    ```

6.  在MaxCompute客户端运行Jar命令。

    ```
    Jar -resources word-count-1.0.jar -classpath /home/resources/word-count-1.0.jar com.taobao.jingfan.WordCount wc_in wc_out;
    ```

    **说明：** 如果您在Java程序中使用了任何资源，请务必将此资源加入-resources参数。Jar命令的详细介绍请参见[作业提交](/cn.zh-CN/开发/MapReduce/功能介绍/MapReduce作业提交.md)。

7.  在MaxCompute客户端查看结果。

    ```
    SELECT * FROM wc_out;
    ```


