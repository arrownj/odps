---
keyword: [RDS, MaxCompute]
---

# Migrate data from ApsaraDB RDS to MaxCompute based on dynamic partitioning

This topic describes how to use the data integration and data synchronization features of DataWorks to migrate data from ApsaraDB RDS to MaxCompute based on dynamic partitioning.

-   The DataWorks environment is ready.
    1.  MaxCompute is activated. For more information, see [Activate MaxCompute](/intl.en-US/Prepare/Activate MaxCompute.md).
    2.  DataWorks is activated. To activate DataWorks, go to the [DataWorks buy page](https://common-buy.aliyun.com/).
    3.  A workflow is created in the DataWorks console. In this example, a workflow is created in a DataWorks workspace in basic mode. For more information, see [Create a workflow]().
-   Connections to the source and destination data stores are created.
    -   A MySQL connection is created as the source connection. For more information, see [Configure a MySQL connection]().
    -   A MaxCompute connection is created as the destination connection. For more information, see [Configure a MaxCompute connection]().

## Migrate data from ApsaraDB RDS to MaxCompute based on dynamic partitioning

After the preceding preparations are made, configure a sync node to migrate data from ApsaraDB RDS to MaxCompute based on dynamic partitioning every day at a scheduled time. For more information about how to configure a sync node, see [Overview]().

1.  Login [DataWorks console](https://workbench.data.aliyun.com/console).

2.  Create a destination table in MaxCompute.

    1.  In the left-side navigation pane, click **Workspaces**.

    2.  In the top navigation bar, select the region where the target workspace resides. Find the target workspace and click **Data Analytics** in the Actions column.

    3.  Right-click a created **workflow**, Select **new** \> **MaxCompute** \> **table**.

    4.  In **create a table** page, select the engine type, and enter **table name**.

    5.  On the table editing page, click **DDL Statement**.

    6.  In the DDL Statement dialog box, enter the following statement and click **Generate Table Schema**:

        ```
        CREATE TABLE IF NOT EXISTS ods_user_info_d (
        uid STRING COMMENT 'User ID',
        gender STRING COMMENT 'Gender',
        age_range STRING COMMENT 'Age range',
        zodiac STRING COMMENT 'Zodiac sign'
        )
        PARTITIONED BY (
        dt STRING
        );                           
        ```

    7.  Click **Submit to Production Environment**.

3.  Create a batch sync node.

    1.  Go to the data analytics page. Right-click the specified workflow and choose **new** \> **data integration** \> **offline synchronization**.

    2.  In **create a node** dialog box, enter **node name**, and click **submit**.

    3.  Configure the source and destination for the batch sync node.

        ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/1950644061/p102236.png)

4.  Configure the partition parameter.

    1.  In the right-side navigation pane of the node configuration tab, click the **Properties** tab.

    2.  In the **General** section, set the **Arguments** parameter. The default value is `${bizdate}` in the format of yyyymmdd.

        **Note:** The value of the **Arguments** parameter in the General section on the Properties tab is the same as that of the **Partition Key Column** parameter in the **Target** section on the node configuration page. When the sync node is scheduled and run, the value of the partition parameter of the destination table is replaced with the date that is one day before the node is run, which is known as the data timestamp. By default, the data generated on the day before the node is run is migrated. To use the date when the node is run as the value of the partition parameter of the destination table, you must customize the partition parameter.

        You can specify a date in one of the following formats for the partition parameter:

        -   N years later: `$[add_months(yyyymmdd,12*N)]`
        -   N years ago: `$[add_months(yyyymmdd,-12*N)]`
        -   N months ago: `$[add_months(yyyymmdd,-N)]`
        -   N weeks later: `$[yyyymmdd+7*N]`
        -   N months later: `$[add_months(yyyymmdd,N)]`
        -   N weeks ago: `$[yyyymmdd-7*N]`
        -   N days later: `$[yyyymmdd+N]`
        -   N days ago: `$[yyyymmdd-N]`
        -   N hours later: `$[hh24miss+N/24]`
        -   N hours ago: `$[hh24miss-N/24]`
        -   N minutes later: `$[hh24miss+N/24/60]`
        -   N minutes ago: `$[hh24miss-N/24/60]`
        **Note:**

        -   Keep the value calculation formula in brackets \[\]. For example, `key1=$[yyyy-mm-dd]`.
        -   The default unit of the calculation result is day. For example, `$[hh24miss-N/24/60]` refers to the calculation result of `(yyyymmddhh24miss - (N/24/60 × 1 day))`. The format hh24miss is used to align the value.
        -   The unit of add\_months is month. For example, `$[add_months(yyyymmdd,12 N)-M/24/60]` refers to the calculation result of `(yyyymmddhh24miss - (12 × N × 1 month)) - (M/24/60 × 1 day)`. The format `yyyymmdd` is used to align the value.
5.  Click ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0311359951/p100471.png)icon to run the code.

6.  You can **operation Log** view the results.


## Generate retroactive data

If you have a large amount of historical data in ApsaraDB RDS that is generated before the node is run, all historical data needs to be automatically migrated to MaxCompute and the partitions need to be automatically created. To generate retroactive data for the current sync node, you can use the Patch Data feature of DataWorks.

1.  Filter historical data in ApsaraDB RDS by date.

    You can set the **Filter** parameter in the **Source** section to filter data in ApsaraDB RDS.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5311359951/p14401.png)

2.  Generate retroactive data for the node. For more information, see [Retroactive instances]().

3.  View the process of extracting data from ApsaraDB RDS on the Run Log tab.

    The logs indicate that Partition 20180913 is automatically created in MaxCompute.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5311359951/p21001.png)

4.  Verify the execution result. Execute the following statement on the MaxCompute client to check whether the data is written to MaxCompute:

    ```
    SELECT count(*) from ods_user_info_d where dt = 20180913;
    ```


## Use hash functions to create partitions based on non-date fields

If you have a large amount of data or full data is migrated to partitions based on a non-date field for the first time, the partitions cannot be automatically created during the migration. In this case, you can map the values in a field in the source table to a corresponding partition in MaxCompute by using a hash function.

1.  Create an SQL script node. Execute the following statements to create a temporary table in MaxCompute and migrate data to the table:

    ```
    drop table if exists ods_user_t;
    CREATE TABLE ods_user_t ( 
            dt STRING,
            uid STRING,
            gender STRING,
            age_range STRING,
            zodiac STRING);
    -- Create a temporary table named ods_user_t and write data from Table ods_user_info_d to Table ods_user_t.
    insert overwrite table ods_user_t select dt,uid,gender,age_range,zodiac from ods_user_info_d;         
    ```

2.  Create a sync node named mysql\_to\_odps to migrate full data from ApsaraDB RDS to MaxCompute. Partitioning is not required.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5311359951/p33591.png)

3.  Execute the following SQL statements to migrate data from Table ods\_user\_t to Table ods\_user\_d based on dynamic partitioning:

    ```
    drop table if exists ods_user_d;
    // Create a MaxCompute partitioned table named ods_user_d, which is the destination table.
        CREATE TABLE ods_user_d (
        uid STRING,
            gender STRING,
            age_range STRING,
            zodiac STRING
    )
    PARTITIONED BY (
        dt STRING
    );
    // Create dynamic partitions for Table ods_user_d based on the dt field in Table ods_user_t. In Table ods_user_d, a partition is automatically created for each unique value in the dt field in Table ods_user_t.
    // For example, if the value of the dt field is 20181025 in some rows of Table ods_user_t, Partition dt=20181025 is created in Table ods_user_d.
    // The following SQL statement is used to migrate data from Table ods_user_t to Table ods_user_d based on dynamic partitioning.
    // The dt field is specified in the SELECT clause. This indicates that the partitions are automatically created based on this field.
    insert overwrite table ods_user_d partition(dt)select dt,uid,gender,age_range,zodiac from ods_user_t;
    // After data migration is complete, you may drop the temporary table to release storage space.
    drop table if exists ods_user_t;
    ```

    You can use SQL statements to migrate data in MaxCompute. For more information about SQL statements, see [Use partitioned tables in MaxCompute](https://yq.aliyun.com/articles/81775?spm=a2c4e.11153940.blogcont182972.20.72856a07pHyeLb).

4.  Configure the three nodes to form a workflow to run these nodes sequentially, as shown in the following figure.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6311359951/p21007.png)

5.  View the execution process. The last node represents the process of dynamic partitioning, as shown in the following figure.

    ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/1950644061/p103159.png)

6.  Verify the execution result. Execute the following statement on the MaxCompute client to check whether the data is written to MaxCompute:

    ```
    SELECT count(*) from ods_user_d where dt = 20180913;
    ```


