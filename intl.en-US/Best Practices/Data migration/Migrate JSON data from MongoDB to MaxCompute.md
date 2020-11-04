---
keyword: migrate data from MongoDB to MaxCompute
---

# Migrate JSON data from MongoDB to MaxCompute

This topic describes how to use the data integration feature of DataWorks to migrate JSON fields from MongoDB to MaxCompute.

-   MaxCompute is activated. For more information, see [Activate MaxCompute](/intl.en-US/Prepare/Activate MaxCompute.md).
-   DataWorks is activated. To activate DataWorks, go to the [DataWorks buy page](https://common-buy.aliyun.com/).
-   A workflow is created in the DataWorks console. In this example, a workflow is created in a DataWorks workspace in basic mode. For more information, see [Create a workflow]().

## Prepare test data in MongoDB

1.  Prepare an account.

    Create a user in your MongoDB database. The username and password of the user are used when you configure a MongoDB connection in DataWorks. In this example, run the following command to create a user:

    ```
    db.createUser({user:"bookuser",pwd:"123456",roles:["root"]})
    ```

    The username is bookuser, the password is 123456, and the permission is root.

2.  Prepare data.

    Upload the following test data to your MongoDB database. In this example, an [ApsaraDB for MongoDB](/intl.en-US/Quick Start for Standalone/Use a standalone ApsaraDB for MongoDB instance.md) instance in a VPC is used. You must apply for a public endpoint for the ApsaraDB for MongoDB instance to communicate with the default resource group of DataWorks.

    ```
    {
                            "store": {
                            "book": [
                            {
                            "category": "reference",
                            "author": "Nigel Rees",
                            "title": "Sayings of the Century",
                            "price": 8.95
                            },
                            {
                            "category": "fiction",
                            "author": "Evelyn Waugh",
                            "title": "Sword of Honour",
                            "price": 12.99
                            },
                            {
                            "category": "fiction",
                            "author": "J. R. R. Tolkien",
                            "title": "The Lord of the Rings",
                            "isbn": "0-395-19395-8",
                            "price": 22.99
                            }
                            ],
                            "bicycle": {
                            "color": "red",
                            "price": 19.95
                            }
                            },
                            "expensive": 10
                            }
    ```

3.  Log on to the ApsaraDB for MongoDB console. In this example, the name of the database is admin, and the name of the collection is userlog. You can run the following command to view the uploaded data:

    ```
    db.userlog.find().limit(10)
    ```


## Migrate JSON data from MongoDB to MaxCompute by using DataWorks

1.  Login [DataWorks console](https://workbench.data.aliyun.com/console).

2.  Create a destination table in the DataWorks console. This table is used to store the data that is migrated from MongoDB.

    1.  Right-click a created **workflow**, Select **new** \> **MaxCompute** \> **table**.

    2.  In **create a table** page, select the engine type, and enter **table name**.

    3.  On the table editing page, click **DDL Statement**.

    4.  In the DDL Statement dialog box, enter the following statement and click **Generate Table Schema**:

        ```
        create table mqdata (MQ data string);
        ```

    5.  Click **Commit to production environment**.

3.  Configure a MongoDB connection. For more information, see [Configure a MongoDB connection]().

4.  Create a batch sync node.

    1.  Go to the data analytics page. Right-click the specified workflow and choose **new** \> **data integration** \> **offline synchronization**.

    2.  In **create a node** dialog box, enter **node name**, and click **submit**.

    3.  In the top navigation bar, choose ![Conversion script](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/3411359951/p100995.png)icon.

    4.  In script mode, click ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4411359951/p101002.png)icon.

    5.  In **import Template** dialog box **SOURCE type**, **data source**, **target type** and **data source**, and click **confirm**.

    6.  Enter the following script in the code editor:

        ```
        {
            "type": "job",
            "steps": [
            {
                "stepType": "mongodb",
                "parameter": {
                    "datasource": "mongodb_userlog",// The name of the connection.
                    "column": [
                        {
                        "name": "store.bicycle.color", // The path of the JSON field. In this example, the color field is extracted.
                        "type": "document.String" // For fields other than top-level fields, the data type of such a field is the same as that of its top-level field. If the specified JSON field is a top-level field, such as the expensive field in this example, enter string.
                        }
                      ],
                    "collectionName": "userlog" // The name of the collection.
                    },
                "name": "Reader",
                "category": "reader"
                },
                {
                    "stepType": "odps",
                    "parameter": {
                    "partition": "",
                    "isCompress": false,
                    "truncate": true,
                    "datasource": "odps_first",
                    "column": [
                    "mqdata" // The name of the column in the MaxCompute table.
                    ],
                    "emptyAsNull": false,
                    "table": "mqdata"
                    },
                    "name": "Writer",
                    "category": "writer"
                    }
                    ],
                    "version": "2.0",
                    "order": {
                    "hops": [
                    {
                    "from": "Reader",
                    "to": "Writer"
                    }
                    ]
                    },
                    "setting": {
                    "errorLimit": {
                    "record": ""
                    },
                    "speed": {
                    "concurrent": 2,
                    "throttle": false,
                    }
                    }
                }
        ```

    7.  Click ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0311359951/p100471.png)icon to run the code.

    8.  You can **operation Log** view the results.


## Verify the execution result

1.  Right-click the workflow and choose **new** \> **MaxCompute** \> **ODPS SQL**.

2.  In **create a node** dialog box, enter **node name**, and click **submit**.

3.  On the configuration tab of the ODPS SQL node, enter the following statement:

    ```
    SELECT * from mqdata;
    ```

4.  Click ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0311359951/p100471.png)icon to run the code.

5.  You can **operation Log** view the results.


