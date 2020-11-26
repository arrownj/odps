---
keyword: [prepare data, generate data]
---

# Prepare data

This topic describes how to use the data demo to generate data that simulates data in a real environment and can be used in subsequent data analytics.

-   A Tablestore instance is created in the China \(Beijing\) region. The instance name and endpoint for connecting to the instance are obtained. You can log on to the Tablestore console and click the instance in the Instance Name column on the Overview or All Instances page to obtain the endpoint for connecting to the instance. If you connect to the instance from a region different from that of the instance, we recommend that you use the public endpoint. For more information, see [Prepare the environment](/intl.en-US/Tutorials/Build an online operation analysis platform/Prepare the environment.md).
-   The AccessKey ID and AccessKey secret of your Alibaba Cloud account are obtained. You can log on to the Alibaba Cloud Management Console by using your Alibaba Cloud account and view the AccessKey ID and AccessKey secret on the [Security Management](https://usercenter.console.aliyun.com/#/manage/ak) page.

    **Note:** The AccessKey ID and AccessKey secret of your Alibaba Cloud account are the credentials for accessing Alibaba Cloud APIs. Make sure that you keep your AccessKey pair safe.


1.  Download the data demo package.

    You can download one of the following data demo packages based on your operating system. In this topic, the Windows 7 64-bit operating system is used.

    -   [Data demo package for macOS](http://yunxi-demo.oss-cn-hangzhou.aliyuncs.com/workshop_demo_mac.zip)
    -   [Data demo package for Linux](http://yunxi-demo.oss-cn-hangzhou.aliyuncs.com/workshop_demo_linux.zip)
    -   [Data demo package for Windows 7 64-bit](http://yunxi-demo.oss-cn-hangzhou.aliyuncs.com/workshop_demo.zip)
2.  Configure the data demo.

    Decompress the data demo package and edit the app.conf file in the conf directory.

    ![Configure the environment](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/2632711061/p49640.png)

    The following content is in the app.conf file:

    ```
    endpoint = "https://workshop-bj-001.cn-beijing.ots.aliyuncs.com"
    instanceName = "workshop-bj-001"
    accessKeyId = "LTAIF24u7g******"
    accessKeySecret = "CcwFeF3sWTPy0wsKULMw34Px******"
    usercount = "200"
    daysCount = "7"
    ```

    You must specify the following parameters:

    -   endpoint: the endpoint used to connect to the Tablestore instance. We recommend that you use the public endpoint.
    -   instanceName: the name of the Tablestore instance.
    -   accessKeyId and accessKeySecret: the AccessKey pair that is used to access Alibaba Cloud APIs.
3.  Start the data demo to prepare test data.

    1.  Start Command Prompt in Windows, go to the directory where the data demo resides, and run the following command to view the usage of commands that are related to the data demo:

        ```
        workshop_demo.exe -h
        ```

        The command returns the following results:

        ```
        workshop_demo.exe -h  
        * prepare: Prepare test data, create data tables, and generate behavior logs of a week for users based on the user count specified in the app.conf file.
        * raw ${userid} ${date} ${Top log count}: Query a specified number of logs of the specified user on the specified date.
        * new/day_active/month_active/day_pv/month_pv: Query data of the following specific types in the result table. new: new users. day_active: daily active users. month_active: monthly active users. day_pv: daily page views (PVs). month_pv: monthly PVs.
        ```

    2.  Run the following command to prepare test data:

        ```
        workshop_demo.exe prepare
        ```

        The following figure shows the command output.

        ![Prepare test data](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/2632711061/p49641.png)

    In this process, the data demo automatically creates two tables in Tablestore. The following tables describe the columns in the created tables.

    -   Raw log table: user\_trace\_log

        |Column|Data type|Description|
        |------|---------|-----------|
        |md5|STRING|The first eight characters of the MD5 value of the user ID. This column is a primary key column.|
        |uid|STRING|The user ID. This column is a primary key column.|
        |ts|BIGINT|The timestamp when the user performed the operation. This column is a primary key column.|
        |ip|STRING|The IP address of the client that sends the request.|
        |status|BIGINT|The status code returned by the server.|
        |bytes|BIGINT|The number of bytes returned to the client.|
        |device|STRING|The model of the terminal used by the user.|
        |system|STRING|The version of the operating system used by the user. The values in this column are in the format of iosxxx or androidxxx.|
        |customize\_event|STRING|The custom event, including logon, exit, purchase, registration, click, background running, user switch, and browse.|
        |use\_time|BIGINT|The use duration of the application at a time. This column is available when the custom event is exit, background running, or user switch.|
        |customize\_event\_content|STRING|The content of the custom event.|

    -   Analysis result table: analysis\_result

        |Column|Data type|Description|
        |------|---------|-----------|
        |metric|STRING|The metric of data. Valid values: new, day\_active, month\_active, day\_pv, month\_pv. This column is a primary key column.|
        |ds|STRING|The data timestamp, in the format of yyyy-mm-dd or yyyy-mm. This column is a primary key column.|
        |num|BIGINT|The value of the specified metric.|

4.  Verify data.

    -   Query detailed logs of a specified user.

        Run the following command to query a specified number of logs of a specified user on a specified date. In the command, set the date to that when the logs are generated.

        ```
        raw ${userid} ${date} ${Top log count}
        ```

        In the preceding command, $\{userid\} indicates the user ID, $\{date\} indicates the date when the logs were generated, and $\{Top log count\} indicates the number of logs to query. For example, if a table was created on June 15, 2019, you can run the `workshop_demo.exe raw 00010 "2019-06-15" 20` command to query 20 logs for the user whose ID is 00010.

        ![Detailed logs](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/2632711061/p49643.png)

        **Note:** Tablestore is schema-free. Therefore, you do not need to pre-define attribute columns. Different events in the customize\_event column have different event content. Therefore, the data demo generates both a custom event and its content in a data record.

    -   Query data in the analysis result table.

        You can run the `workshop_demo.exe day_active` command to query the number of daily active users.

        ![Daily active users](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/2632711061/p49644.png)


