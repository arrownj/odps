---
keyword: [migrate data from Hadoop to MaxCompute, DataWorks data synchronization]
---

# Migrate data from Hadoop to MaxCompute

This topic describes how to use the data synchronization feature of DataWorks to migrate data from Hadoop Distributed File System \(HDFS\) to MaxCompute. Data synchronization between MaxCompute and Hadoop or Spark is supported.

-   MaxCompute is activated. A MaxCompute project is created.

    In this example, the bigdata\_DOC project in the China \(Hangzhou\) region is used. For more information, see [Activate MaxCompute](/intl.en-US/Prepare/Activate MaxCompute.md).

-   A Hadoop cluster is created.

    Before data migration, make sure that your Hadoop cluster works properly. You can use Alibaba Cloud E-MapReduce to create a Hadoop cluster. For more information, see [Create a cluster](/intl.en-US/Quick Start/Create a cluster.md).

    In this example, the following configurations are specified for the Hadoop cluster:

    -   E-MapReduce version: E-MapReduce V3.11.0
    -   Cluster type: Hadoop
    -   Required software services: HDFS 2.7.2, YARN 2.7.2, Hive 2.3.3, Ganglia 3.7.2, Spark 2.2.1, Hue 4.1.0, Zeppelin 0.7.3, Tez 0.9.1, Sqoop 1.4.6, Pig 0.14.0, ApacheDS 2.0.0, and Knox 0.13.0
    The Hadoop cluster is deployed on the classic network in the China \(Hangzhou\) region with the high availability \(HA\) mode disabled. A public IP address and an internal IP address are configured for the Elastic Compute Service \(ECS\) instance in the primary instance group.


1.  Prepare data.

    1.  Create test data in the Hadoop cluster.
        1.  Log on to the E-MapReduce console. In the top navigation bar, click Data Platform. On the Data Platform tab, find the E-MapReduce project that you created and click Edit Job in the Actions column. In the left-side navigation pane of the Edit Job tab, right-click the folder under which you want to create a job and select Create Job. In this example, create a Hive job whose name is doc. For more information, see [Edit jobs](/intl.en-US/Data Development/Edit jobs.md). In the job that you created, execute the CREATE TABLE statement to create a test table. In this example, copy and paste the following code into the code editor:

            ```
            CREATE TABLE IF NOT
            EXISTS hive_doc_good_sale(
               create_time timestamp,
               category STRING,
               brand STRING,
               buyer_id STRING,
               trans_num BIGINT,
               trans_amount DOUBLE,
               click_cnt BIGINT
               )
               PARTITIONED BY (pt string) ROW FORMAT
            DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n';
            ```

        2.  Click **Run** in the upper-right corner of the code editor. If the `Query executed successfully` message appears, the hive\_doc\_good\_sale table is created in the Hadoop cluster.
        3.  Insert test data into the table. You can import existing data records from other data sources such as Object Storage Service \(OSS\) or insert new data records. In this example, insert the following data records:

            ```
            insert into
            hive_doc_good_sale PARTITION(pt =1 ) values('2018-08-21','Coat','Brand A','lilei',3,500.6,7),('2018-08-22','Fresh food','Brand B','lilei',1,303,8),('2018-08-22','Coat','Brand C','hanmeimei',2,510,2),(2018-08-22,'Bathroom product','Brand A','hanmeimei',1,442.5,1),('2018-08-22','Fresh food','Brand D','hanmeimei',2,234,3),('2018-08-23','Coat','Brand B','jimmy',9,2000,7),('2018-08-23','Fresh food','Brand A','jimmy',5,45.1,5),('2018-08-23','Coat','Brand E','jimmy',5,100.2,4),('2018-08-24','Fresh food','Brand G','peiqi',10,5560,7),('2018-08-24','Bathroom product','Brand F','peiqi',1,445.6,2),('2018-08-24','Coat','Brand A','ray',3,777,3),('2018-08-24','Bathroom product','Brand G','ray',3,122,3),('2018-08-24','Coat','Brand C','ray',1,62,7) ;
            ```

        4.  After you insert the data, execute the `select * from hive_doc_good_sale where pt =1;` statement to check whether the data exists in the table that you created in the Hadoop cluster.
    2.  Create a MaxCompute table in the DataWorks console.
        1.  Log on to the [DataWorks console](https://workbench.data.aliyun.com/console). In the left-side navigation pane, click Workspaces. On the Workspaces page, find the workspace in which you want to create a table and click **Data Analytics** in the Actions column.
        2.  On the DataStudio page, right-click a workflow and choose **Create** \> **MaxCompute** \> **Table**.
        3.  In the Create Table dialog box, set the Please select an Engine type parameter to MaxCompute, enter a table name, and then click **Commit**.
        4.  On the configuration tab of the created table, click **DDL Statement**.
        5.  In **DDL Statement** dialog box, enter the CREATE TABLE statement and click **Generate Table Schema**. In the Confirm message, click OK. In this example, enter the following CREATE TABLE statement to create a MaxCompute table whose name is hive\_doc\_good\_sale:

            ```
            CREATE TABLE IF NOT EXISTS hive_doc_good_sale(
               create_time string,
               category STRING,
               brand STRING,
               buyer_id STRING,
               trans_num BIGINT,
               trans_amount DOUBLE,
               click_cnt BIGINT
               )
               PARTITIONED BY (pt string) ;
            ```

            When you create a table, you must consider the mapping between Hive data types and MaxCompute data types. For more information, see [Data type mappings](/intl.en-US/Development/SQL/Appendix/Data type mappings.md).

            You can also use the command-line tool odpscmd to create a MaxCompute table. For more information about how to install and configure the odpscmd tool, see [Install and configure the odpscmd client](/intl.en-US/Prepare/Install and configure the odpscmd client.md).

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5211359951/p11598.png)

            **Note:** If you need to resolve compatibility issues between Hive data types and MaxCompute data types, we recommend that you run the following commands by using the odpscmd tool:

            ```
            set odps.sql.type.system.odps2=true;
            set odps.sql.hive.compatible=true;
            ```

        6.  Click **Commit to Production Environment**. The MaxCompute table is created.
        7.  On the left-side navigation submenu, click **Workspace Tables**. On the Workspace Tables tab, view the created MaxCompute table.

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5211359951/p11599.png)

2.  Synchronize data.

    1.  Create a custom resource group.

        In most cases, the network where a MaxCompute project resides is not connected to a data node in a Hadoop cluster. To resolve this connectivity issue, you can create a custom resource group to run your DataWorks sync node on the primary node of the Hadoop cluster. The primary node and data nodes are usually connected.

        1.  View information about the data nodes of the Hadoop cluster.
            1.  Log on to the [E-MapReduce console](https://emr.console.aliyun.com/). In the top navigation bar, click **Cluster Management**.
            2.  On the Cluster Management tab, click the name of the Hadoop cluster that you created. On the cluster details page, click **Instances** in the left-side navigation pane. On the Instances page, view information about the data nodes of the Hadoop cluster.

                You can also click the ECS ID of the primary node to go to the details page of the ECS instance. On the Instance Details page, click Connect to log on to the ECS instance and run the `hadoop dfsadmin –report` command to view information about the data nodes.

                ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5211359951/p11601.png)

                In this example, each data node has only an internal IP address and cannot communicate with the default resource group of DataWorks. Therefore, you must create a custom resource group to run your DataWorks sync node on the primary node.

        2.  Create a custom resource group for Data Integration.
            1.  Log on to the DataWorks console. In the left-side navigation pane, click Workspaces. On the Workspaces page, find the workspace in which you want to create a custom resource group and click **Data Integration** in the Actions column. On the Data Integration page, click **Custom Resource Group** in the left-side navigation pane. On the Custom Resource Groups page, click **Add Resource Group** in the upper-right corner.

                **Note:** The preceding steps to create a custom resource group apply only to DataWorks Professional Edition and more advanced editions.

            2.  When you add a server, enter information such as the ECS UUID and server IP address. If the network type is classic network, enter the hostname. If the network type is virtual private cloud \(VPC\), enter the ECS UUID. You can add scheduling resources whose network type is classic network only in the China \(Shanghai\) region in DataWorks V2.0. In other regions, you must add scheduling resources whose network type is VPC regardless of the network type of your ECS instances.

                For the server IP address, enter the public IP address of the primary node because the internal IP address may be unreachable. To query the ECS UUID, log on to the primary node and run the dmidecode \| grep UUID command. You can also use this command to query the ECS UUID even if your Hadoop cluster is not deployed in the E-MapReduce environment.

                ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5211359951/p11603.png)

            3.  After you add a server, make sure that the primary node and DataWorks are connected. If you add an ECS instance, you must configure a security group.
                -   If you use an internal IP address, add the internal IP address to a security group. For more information, see [Configure a security group]().
                -   If you use a public IP address, set the Internet inbound and outbound rules for a security group. In this example, all ports are opened to allow traffic from the Internet. In actual scenarios, we recommend that you set specific security group rules for security reasons.
            4.  After you complete the preceding steps, install an agent for the custom resource group as prompted. If the status of the ECS instance is **Available**, the custom resource group is added.

                If the status of the ECS instance is **Unavailable**, log on to the primary node and run the `tail –f/home/admin/alisatasknode/logs/heartbeat.log` command to check whether the heartbeat packets between DataWorks and the primary node timed out.

    2.  Create connections.

        After you create a workspace in DataWorks, the MaxCompute connection odps\_first is created by default. In this example, you only need to create a connection for the Hadoop cluster. For more information, see [t1695542.md\#]().

        1.  On the Data Integration page of DataWorks, click Connection in the left-side navigation pane.
        2.  On the **Data Source** page, click **New data source** in the upper-right corner.
        3.  In the Add data source dialog box, select **HDFS**.
        4.  In the Add HDFS data source dialog box, set the parameters as required.

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5211359951/p11608.png)

            |Parameter|Description|
            |:--------|:----------|
            |**Data Source Name**|The name of the connection. The name can contain letters, digits, and underscores \(\_\), and must start with a letter.|
            |**Description**|The description of the connection. The description can be up to 80 characters in length.|
            |**Applicable environment**|The environment in which the connection is used. Valid values: **Development** and **Production**. **Note:** This parameter is displayed only when the workspace is in standard mode. |
            |**DefaultFS**|The address of the NameNode in the HDFS. If the Hadoop cluster is in HA mode, the address is `hdfs://IP address of emr-header-1:8020`. If the Hadoop cluster is in non-HA mode, the address is `hdfs://IP address of emr-header-1:9000`. In this example, the primary node emr-header-1 is connected to DataWorks over the Internet. Therefore, enter the public IP address and allow traffic from the Internet. |

        5.  Click **Test connectivity**.
        6.  After the connectivity test is passed, click **Complete**.

            **Note:** If the network type of the Hadoop cluster is VPC, the connectivity test is not supported.

    3.  Configure a sync node.
        1.  On the **Data Analytics** tab of the DataStudio page, move the pointer over the Create icon and choose **Data Integration** \> **Batch Synchronization**.
        2.  In the Create Node dialog box, set the **Node Name** parameter and click **Commit**.
        3.  On the configuration tab of the created sync node, click the **Switch to Code Editor** icon in the top toolbar.
        4.  In the Confirm message, click **OK**. The code editor appears.
        5.  Click the **Apply Template** icon in the top toolbar.
        6.  In the Apply Template dialog box, set the **Source Connection Type**, **Connection**, **Target Connection Type**, and **Connection** parameters and click **OK**.
        7.  After the template is applied, the basic settings of HDFS Reader are configured. You can configure the data store and source table for HDFS Reader. In this example, the following script is used. For more information, see [Configure HDFS Reader]().

            ```
            {
              "configuration": {
                "reader": {
                  "plugin": "hdfs",
                  "parameter": {
                    "path": "/user/hive/warehouse/hive_doc_good_sale/", 
                    "datasource": "HDFS1",
                    "column": [
                      {
                        "index": 0,
                        "type": "string"
                      },
                      {
                        "index": 1,
                        "type": "string"
                      },
                      {
                        "index": 2,
                        "type": "string"
                      },
                      {
                        "index": 3,
                        "type": "string"
                      },
                      {
                        "index": 4,
                        "type": "long"
                      },
                      {
                        "index": 5,
                        "type": "double"
                      },
                      {
                        "index": 6,
                        "type": "long"
                      }
                    ],
                    "defaultFS": "hdfs://121.199.11.138:9000",
                    "fieldDelimiter": ",",
                    "encoding": "UTF-8",
                    "fileType": "text"
                  }
                },
                "writer": {
                  "plugin": "odps",
                  "parameter": {
                    "partition": "pt=1",
                    "truncate": false,
                    "datasource": "odps_first",
                    "column": [
                      "create_time",
                      "category",
                      "brand",
                      "buyer_id",
                      "trans_num",
                      "trans_amount",
                      "click_cnt"
                    ],
                    "table": "hive_doc_good_sale"
                  }
                },
                "setting": {
                  "errorLimit": {
                    "record": "1000"
                  },
                  "speed": {
                    "throttle": false,
                    "concurrent": 1,
                    "mbps": "1",
                  }
                }
              },
              "type": "job",
              "version": "1.0"
            }
            ```

            where, the path parameter specifies the directory where the source data is stored in the Hadoop cluster. You can log on to the primary node and run the `hdfs dfs –ls /user/hive/warehouse/hive_doc_good_sale` command to check the directory. For a partitioned table, the data synchronization feature of DataWorks can automatically recurse to the partition where the data is stored.

            ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6211359951/p11611.png)

        8.  After the configuration is complete, click the **Run** icon. If the sync node is run, the data is synchronized. If the sync node fails, check logs for troubleshooting.

1.  On the left-side navigation submenu of the DataStudio page, click **Ad-Hoc Query**.
2.  On the Ad-Hoc Query tab, move the pointer over the Create icon and choose **Create** \> **ODPS SQL**.
3.  In the Create Node dialog box, set the parameters and click Commit. In the code editor of the created ad hoc query node, execute the following SQL statement to view the data that is synchronized to the hive\_doc\_good\_sale table:

    ```
    --Check whether the data is written to MaxCompute.
    select * from hive_doc_good_sale where pt=1;
    ```

    You can also run the `select * FROM hive_doc_good_sale where pt =1;` command by using the odpscmd tool to query the synchronized data.


You can perform the same steps if you want to migrate data from MaxCompute to Hadoop. However, you must exchange the reader and writer in the preceding script. To migrate data from MaxCompute to Hadoop, you can use the following script:

```

{
  "configuration": {
    "reader": {
      "plugin": "odps",
      "parameter": {
      "partition": "pt=1",
      "isCompress": false,
      "datasource": "odps_first",
      "column": [
        "create_time",
        "category",
        "brand",
      "buyer_id",
      "trans_num",
      "trans_amount",
      "click_cnt"
    ],
    "table": "hive_doc_good_sale"
    }
  },
  "writer": {
    "plugin": "hdfs",
    "parameter": {
    "path": "/user/hive/warehouse/hive_doc_good_sale",
    "fileName": "pt=1",
    "datasource": "HDFS_data_source",
    "column": [
      {
        "name": "create_time",
        "type": "string"
      },
      {
        "name": "category",
        "type": "string"
      },
      {
        "name": "brand",
        "type": "string"
      },
      {
        "name": "buyer_id",
        "type": "string"
      },
      {
        "name": "trans_num",
        "type": "BIGINT"
      },
      {
        "name": "trans_amount",
        "type": "DOUBLE"
      },
      {
        "name": "click_cnt",
        "type": "BIGINT"
      }
    ],
    "defaultFS": "hdfs://47.99.162.100:9000",
    "writeMode": "append",
    "fieldDelimiter": ",",
    "encoding": "UTF-8",
    "fileType": "text"
    }
  },
  "setting": {
    "errorLimit": {
      "record": "1000"
  },
  "speed": {
    "throttle": false,
    "concurrent": 1,
    "mbps": "1",
  }
  }
},
"type": "job",
"version": "1.0"
}
```

Before you run a sync node to migrate data from MaxCompute to Hadoop, you must configure the Hadoop cluster. For more information, see [Configure HDFS Writer](). After the sync node is run, you can copy the file that is synchronized.

