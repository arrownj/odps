---
keyword: create tables
---

# Create tables

This topic describes how to create MaxCompute tables to store raw data and processed data.

-   MaxCompute is activated. A DataWorks workspace in basic mode is created. For more information, see [Prepare the environment](/intl.en-US/Tutorials/Build an online operation analysis platform/Prepare the environment.md).
-   You have obtained the permissions to access data in Tablestore. If you use the same account to activate MaxCompute and Tablestore, you can obtain the permissions with one click in the [RAM console](https://ram.console.aliyun.com/?spm=a2c4e.11153940.0.0.7828675d6o14Gi#/role/authorize?request=%7B%22Requests%22:%20%7B%22request1%22:%20%7B%22RoleName%22:%20%22AliyunODPSDefaultRole%22,%20%22TemplateId%22:%20%22DefaultRole%22%7D%7D,%20%22ReturnUrl%22:%20%22https:%2F%2Fram.console.aliyun.com%2F%22,%20%22Service%22:%20%22ODPS%22%7D). If you use different accounts to activate MaxCompute and Tablestore, you can obtain the permissions by customizing an authorization policy. For more information, see [Access Tablestore data](/intl.en-US/Development/External table/Access Tablestore data.md).

1.  Go to the DataStudio page.

    1.  Log on to the DataWorks console. Click [Workspaces](https://workbench.data.aliyun.com/consolenew#/projectlist) in the left-side navigation pane. On the page that appears, select **China \(Shanghai\)** in the top navigation bar.

    2.  Find the workspace that you created and click **Data Analytics** in the Actions column.

2.  Create a workflow.

    1.  On the Data Analytics tab of the DataStudio page, right-click **Business Flow**, and select **Create Workflow**.

        ![Create Workflow](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/3315207951/p71203.png)

    2.  In the Create Workflow dialog box, specify **Workflow Name** and **Description**, and click **Create**. In this example, set Workflow Name to Workshop.

3.  Create tables.

    1.  Create an external table named ots\_user\_trace\_log.

        **Note:** The external table cannot be created by using DDL statements in wizard mode. To create the external table, you can manually add fields for this table in wizard mode. You can also create an ODPS SQL node in MaxCompute, and use DDL statements of the ODPS SQL node to create the external table. The following example describes how to use DDL statements of an ODPS SQL node to create an external table.

        1.  On the Data Analytics tab of the DataStudio page, unfold the Workshop workflow, right-click **MaxCompute**, and select **Create** \> **ODPS SQL**. In the Create Node dialog box, set Node Name to **workshop**, and click **Commit**.
        2.  Double-click the created ODPS SQL node. On the tab that appears, enter the following DDL statement in the code editor to create an external table named ots\_user\_trace\_log.

            DDL statement:

            ```
            CREATE EXTERNAL TABLE ots_user_trace_log (
                md5 string COMMENT 'The first eight characters in the MD5 value of the user ID (UID)',
                uid string COMMENT 'The UID',
                ts bigint COMMENT 'The timestamp when the user performs the operation',
                ip string COMMENT 'The IP address',
                status bigint COMMENT 'The status code returned by the server',
                bytes bigint COMMENT 'The number of bytes returned to the client',
                device string COMMENT 'The model of the terminal on which the client runs',
                system string COMMENT 'The version of the operating system in which the client runs, in the format of ios xxx or android xxx',
                customize_event string COMMENT 'The custom event. You can specify a custom event that is reported for logon, logoff, purchase, registration, click, background running, user switching, browsing, or comment',
                use_time bigint COMMENT 'The duration for which the app is used at a time. This field is available when you configure a custom event that is reported for logoff, beckground running, or user switching
                customize_event_content string COMMENT 'The content of the custom event. This field is available when you configure a custom event that is reported for browsing or comment'
            )
            STORED BY 'com.aliyun.odps.TableStoreStorageHandler'
            WITH SERDEPROPERTIES (
                'tablestore.columns.mapping'=':md5,:uid,:ts, ip,status,bytes,device,system,customize_event,use_time,customize_event_content',
                'tablestore.table.name'='user_trace_log'
            )
            LOCATION 'tablestore://workshop-bj-001.cn-beijing.ots.aliyuncs.com/';
            ```

            -   STORED BY: required. The value `com.aliyun.odps.TableStoreStorageHandler` specifies the storage handler that is built in MaxCompute to process the data stored in Tablestore. This parameter defines the logic of interactions between MaxCompute and Tablestore.
            -   SERDEPROPERITES: required. The WITH SERDEPROPERITES clause allows you to specify options. If the STORED BY parameter is set to com.aliyun.odps.TableStoreStorageHandler, you must specify the following options in the WITH SERDEPROPERITES clause:
                -   tablestore.columns.mapping: the columns of the Tablestore table that you want to access from MaxCompute. These columns include primary key columns and property columns.

                    **Note:**

                    -   Primary key columns are columns that begin with a colon \(:\). `:md5`, `:uid`, and :ts are primary key columns in this example. The remaining columns are property columns.
                    -   If you specify column mapping by using the tablestore.columns.mapping option, you must specify all the primary key columns in the Tablestore table. You need only to specify the property columns that you want to access from MaxCompute. The specified property columns must be contained in the Tablestore table. Otherwise, errors are returned when you query data in the created external table.
                -   tablestore.table.name: the name of the Tablestore table that you want to access from MaxCompute. If you specify an invalid table name or the specified table name does not exist, an error is returned and MaxCompute does not create the Tablestore table.
            -   LOCATION: the endpoint of Tablestore. Enter the endpoint of your Tablestore instance in this parameter. For more information, see [Prepare the environment](/intl.en-US/Tutorials/Build an online operation analysis platform/Prepare the environment.md).

                **Note:** If you enter the public endpoint of your Tablestore instance in the LOCATION clause, such as LOCATION 'tablestore://workshop-bj-001.cn-beijing.ots.aliyuncs.com/', an error that indicates a network failure may be returned. In this case, you can enter the endpoint of your Tablestore instance in the classic network, for example, LOCATION 'tablestore://workshop-bj-001.cn-beijing.ots-internal.aliyuncs.com/'.

        3.  Click the Save icon and then the Run icon.

            If the operational log indicates that the preceding statement is executed, the external table is created.

    2.  Create a table named ods\_user\_trace\_log.

        Use the same method as in Substep a to create the table. The ods\_user\_trace\_log table is a table at the operational data store \(ODS\) layer.

        ```
        CREATE TABLE IF NOT EXISTS ods_user_trace_log (
            md5 STRING COMMENT 'The first eight characters in the MD5 value of the UID',
            uid STRING COMMENT 'The UID',
            ts BIGINT COMMENT 'The timestamp when the user performs the operation',
            ip STRING COMMENT 'The IP address',
            status BIGINT COMMENT 'The status code returned by the server',
            bytes BIGINT COMMENT 'The number of bytes returned to the client',
            device STRING COMMENT 'The model of the terminal on which the client runs',
            system STRING COMMENT 'The version of the operating system in which the client runs, in the format of ios xxx or android xxx',
            customize_event STRING COMMENT 'The custom event. You can specify a custom event that is reported for logon, logoff, purchase, registration, click, background running, user switching, browsing, or comment',
            use_time BIGINT COMMENT 'The duration for which the app is used at a time. This field is available when you configure a custom event that is reported for logoff, background running, or user switching',
            customize_event_content STRING COMMENT 'The content of the custom event. This field is available when you configure a custom event that is reported for browsing or comment'
        )
        PARTITIONED BY (
            dt STRING
        );
        ```

    3.  Create a table named dw\_user\_trace\_log.

        Use the same method as in Substep a to create the table. The dw\_user\_trace\_log table is a table at the data warehouse \(DW\) layer.

        ```
        CREATE TABLE IF NOT EXISTS dw_user_trace_log (
            uid STRING COMMENT 'The UID',
            region STRING COMMENT 'The region in which the user resides. The region is obtained based on the IP address of the user',
            device_brand string comment 'The brand of the terminal on which the client runs',
            device STRING COMMENT 'The model of the terminal on which the client runs',
            system_type STRING COMMENT 'The type of the operating system in which the client runs. Valid values: Android, IOS, ipad, and Windows_phone',
            customize_event STRING COMMENT 'The custom event. You can specify a custom event that is reported for logon, logoff, purchase, registration, click, background running, user switching, browsing, or comment',
            use_time BIGINT COMMENT 'The duration for which the app is used at a time. This field is available when you configure a custom event that is reported for logoff, background running, or user switching',
            customize_event_content STRING COMMENT 'The content of the custom event. This field is available when you configure a custom event that is reported for browsing or comment'
        )
        PARTITIONED BY (
            dt STRING
        );
        ```

    4.  Create a table named rpt\_user\_trace\_log.

        Use the same method as in Substep a to create the table. The rpt\_user\_trace\_log table is a table at the application data store \(ADS\) layer.

        ```
        CREATE TABLE IF NOT EXISTS rpt_user_trace_log (
            country STRING COMMENT 'The country in which the user resides',
            province STRING COMMENT 'The province in which the user resides',
            city STRING COMMENT 'The city in which the user resides',
            device_brand string comment 'The brand of the terminal on which the client runs',
            device STRING COMMENT 'The model of the terminal on which the client runs',
            system_type STRING COMMENT 'The type of the operating system in which the client runs. Valid values: Android, IOS, ipad, and Windows_phone',
            customize_event STRING COMMENT 'The custom event. You can specify a custom event that is reported for logon, logoff, purchase, registration, click, background running, user switching, browsing, or comment',
            use_time BIGINT COMMENT 'The duration for which the app is used at a time. This field is available when you configure a custom event that is reported for logoff, background running, or user switching',
            customize_event_content STRING COMMENT 'The content of the custom event. This field is available when you configure a custom event that is reported for browsing or comment'
            pv bigint comment 'The number of page views (PVs)',
            uv bigint comment 'The number of unique visitors (UVs)'
        )
        PARTITIONED BY (
            dt STRING
        );
        ```

    5.  \(Optional\) If your workspace is in standard mode, the external table created in Substep a is committed only to the development environment. To commit the created external table to the production environment, perform the following steps:

        1.  On the Data Analytics tab of the DataStudio page, unfold your workflow under Business Flow, right-click Table, and then select Import Table.

            ![Import Table](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/4228397061/p186681.png)

        2.  In the Import Table dialog box, select the external table created in the development environment from the Table drop-down list, and click **OK**.

            **Note:** You can move the pointer over the table name to check whether the table is created in the production environment or in the development environment. If you select a table in the production environment, the system displays a message indicating that the table does not exist.

        3.  Double-click the external table that you imported. On the tab that appears, click **Commit to Production Environment** at the top of the tab.
4.  Verify the created tables.

    1.  After you create the tables, unfold the Workshop workflow under Business Flow, and choose **MaxCompute** \> **Table** to view all the created tables.

    2.  Unfold the Workshop workflow under Business Flow, right-click **Data Analytics** under **MaxCompute**, and then select **Create** \> **ODPS SQL**.

    3.  In the **Create Node** dialog box, specify **Node Name**, and click **Commit** to create an ODPS SQL node.

    4.  Enter the following SQL statements in the code editor of the ODPS SQL node, and click the ![Run](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/5392309951/p66566.png) icon.

        ```
        DESCRIBE ots_user_trace_log;
        DESCRIBE ods_user_trace_log;
        DESCRIBE dw_user_trace_log;
        DESCRIBE rpt_user_trace_log;
        ```

        The following information about the table schema is returned.

        ![Returned information](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/3315207951/p50640.png)


