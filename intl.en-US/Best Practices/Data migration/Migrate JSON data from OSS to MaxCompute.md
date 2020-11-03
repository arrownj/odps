---
keyword: migrate data from OSS to MaxCompute
---

# Migrate JSON data from OSS to MaxCompute

This topic describes how to use the data integration feature of DataWorks to migrate JSON data from Object Storage Service \(OSS\) to MaxCompute and use the GET\_JSON\_OBJECT function to extract JSON objects.

-   [MaxCompute is activated](/intl.en-US/Prepare/Activate MaxCompute.md).
-   [DataWorks is activated](https://common-buy.aliyun.com/).
-   A workflow is created in the DataWorks console. In this example, a workflow is created in a DataWorks workspace in basic mode. For more information, see [Create a workflow]().
-   A TXT file that contains JSON data is uploaded to an OSS bucket. In this example, the OSS bucket is in the China \(Shanghai\) region. The TXT file contains the following JSON data:

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


## Migrate JSON data from OSS to MaxCompute

1.  Add an OSS connection. For more information, see [t1695549.md\#]().

2.  Create a table in DataWorks to store the JSON data to be migrated from OSS.

    1.  Login [DataWorks console](https://workbench.data.aliyun.com/console).

    2.  In **create a table** page, select the engine type, and enter **table name**.

    3.  On the table editing page, click **DDL Statement**.

    4.  In the DDL Statement dialog box, enter the following statement and click **Generate Table Schema**:

        ```
        create table mqdata (mq_data string);
        ```

    5.  Click **Submit to Production Environment**.

3.  Create a batch synchronization node.

    1.  Go to the data analytics page. Right-click the specified workflow and choose **new** \> **data integration** \> **offline synchronization**.

    2.  In **create a node** dialog box, enter **node name**, and click **submit**.

    3.  In the top navigation bar, choose ![Conversion script](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/3411359951/p100995.png)icon.

    4.  In script mode, click ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4411359951/p101002.png)icon.

    5.  In **import Template** dialog box **SOURCE type**, **data source**, **target type** and **data source**, and click **confirm**.

    6.  Modify JSON code and click the ![Run icon](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4411359951/p101008.png) icon.

        Sample code:

        ```
        {
            "type": "job",
            "steps": [
                {
                    "stepType": "oss",
                    "parameter": {
                        "fieldDelimiterOrigin": "^",
                        "nullFormat": "",
                        "compress": "",
                        "datasource": "OSS_userlog",
                        "column": [
                            {
                                "name": 0,
                                "type": "string",
                                "index": 0
                            }
                        ],
                        "skipHeader": "false",
                        "encoding": "UTF-8",
                        "fieldDelimiter": "^",
                        "fileFormat": "binary",
                        "object": [
                            "applog.txt"
                        ]
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
                            "mqdata"
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


## Use the GET\_JSON\_OBJECT function to extract JSON objects

1.  Create an ODPS SQL node.

    1.  Right-click the workflow and choose **new** \> **MaxCompute** \> **ODPS SQL**.

    2.  In **create a function** dialog box, enter **function name**, click **submit**.

    3.  On the configuration tab of the ODPS SQL node, enter the following statements:

        ```
        --Query data in the mqdata table.
        SELECT * from mqdata;
        --Obtain the value of the expensive field.
        SELECT GET_JSON_OBJECT(mqdata.MQdata,'$.expensive') FROM mqdata;
        ```

    4.  Click ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0311359951/p100471.png) icon to run the code.

    5.  You can **operation Log** view the results.


