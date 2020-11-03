---
keyword: MapReduce
---

# \(Optional\) Use MapReduce

This topic describes how to compile and run WordCount samples of MapReduce after you install the MaxCompute client.

## Prerequisites

-   JDK 1.6 or a later version is installed.

    **Note:** If you use Maven, you can search for odps-sdk-mapred in the [Maven repository](http://search.maven.org/) to obtain the latest version of the SDK for Java. You can configure the Maven dependency in the following way:

    ```
    <dependency>
        <groupId>com.aliyun.odps</groupId>
        <artifactId>odps-sdk-mapred</artifactId>
        <version>0.26.2-public</version>
    </dependency>
    ```

-   The MaxCompute client is deployed. For more information, see [Install and configure the odpscmd client](/intl.en-US/Prepare/Install and configure the odpscmd client.md). For more information about how to use the MaxCompute client, see [MaxCompute client](/intl.en-US/Tools and Downloads/Client.md).

## Procedure

1.  Run ./bin/odpscmd in a Linux operating system or ./bin/odpscmd.bat in a Windows operating system to enter the required project.
2.  Execute the following statements to create the input and output tables.

    ```
    --Create the input table wc_in.
    CREATE TABLE wc_in (key STRING, value STRING);
    --Create the output table wc_out.
    CREATE TABLE wc_out (key STRING, cnt BIGINT);
    ```

    For more information about the statements used to create tables, see [Create a table](/intl.en-US/Development/Common commands/Table-level operations.md).

3.  Use one of the following methods to insert data into the wc\_in table:
    -   Run Tunnel commands to upload data.

        The following example shows the data that you want to insert into the table. You must create a kv.txt file on your machine and save the data to the file. Assume that the kv.txt file is saved in D:\\.

        ```
        238,val_238
        186,val_86
        186,val_86
        ```

        Run the following command to upload the data:

        ```
        Tunnel upload D:\kv.txt wc_in;
        ```

    -   Execute the following SQL statement to insert the data:

        ```
        INSERT INTO TABLE wc_in VALUES ('238',' val_238'),('186','val_86'),('186','val_86');
        ```

4.  Compile a MapReduce program and upload it to MaxCompute.

    Create a project in Eclipse or MaxCompute Studio and compile the MapReduce program in the project. After local debugging succeeds, export the JAR package of the compiled program, such as Word-count-1.0.jar, and upload it to MaxCompute.

    **Note:** In this topic, you can use the Word-count-1.0.jar package generated based on the sample code in [WordCount samples](/intl.en-US/Development/MapReduce/Program Example/WordCount.md).

5.  Add the JAR package as a project resource on the MaxCompute client. In this example, the JAR package is named Word-count-1.0.jar.

    ```
    ADD JAR word-count-1.0.jar;
    ```

6.  Run the JAR command on the MaxCompute client.

    ```
    Jar -resources word-count-1.0.jar -classpath /home/resources/word-count-1.0.jar com.taobao.jingfan.WordCount wc_in wc_out;
    ```

    **Note:** If the Java program uses resources, use the -resources option to specify the resources. For more information about the JAR command, see [Submit a job](/intl.en-US/Development/MapReduce/Function Introduction/Job submission.md).

7.  View the command output on the MaxCompute client.

    ```
    SELECT * FROM wc_out;
    ```


