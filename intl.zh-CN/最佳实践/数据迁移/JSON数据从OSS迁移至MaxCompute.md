---
keyword: OSS迁移至MaxCompute
---

# JSON数据从OSS迁移至MaxCompute

本文为您介绍如何通过DataWorks数据集成，将JSON数据从OSS迁移至MaxCompute，并使用MaxCompute内置字符串函数GET\_JSON\_OBJECT提取JSON信息。

-   [开通MaxCompute](/intl.zh-CN/准备工作/开通MaxCompute.md)。
-   [开通DataWorks](https://common-buy.aliyun.com/)。
-   在DataWorks上完成创建业务流程，本例使用DataWorks简单模式。详情请参见[创建业务流程]()。
-   将JSON文件重命名为后缀为`.txt`的文件，并上传至OSS。本文中OSS Bucket地域为华东2（上海）。示例文件如下。

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


## 将JSON数据从OSS迁移至MaxCompute

1.  新增OSS数据源。详情请参见[配置OSS数据源]()。

2.  在DataWorks上新建数据表，用于存储迁移的JSON数据。

    1.  登录[DataWorks控制台](https://workbench.data.aliyun.com/console)。

    2.  在**新建表**页面，选择引擎类型并输入**表名**。

    3.  在表的编辑页面，单击**DDL模式**。

    4.  在DDL模式对话框，输入如下建表语句，单击**生成表结构**。

        ```
        create table mqdata (mq_data string);
        ```

    5.  单击**提交到生产环境**。

3.  新建离线同步节点。

    1.  进入数据开发页面，右键单击指定业务流程，选择**新建** \> **数据集成** \> **离线同步**。

    2.  在**新建节点**对话框中，输入**节点名称**，并单击**提交**。

    3.  在顶部菜单栏上，单击![转化脚本](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8793359951/p100995.png)图标。

    4.  在脚本模式下，单击顶部菜单栏上的![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8793359951/p101002.png)图标。

    5.  在**导入模板**对话框中选择**来源类型**、**数据源**、**目标类型**及**数据源**，并单击**确定**。

    6.  修改JSON代码后，单击![运行](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8793359951/p101008.png)按钮。

        示例代码如下。

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


## 结果验证

1.  新建ODPS SQL节点。

    1.  右键单击业务流程，选择**新建** \> **MaxCompute** \> **ODPS SQL**。

    2.  在**新建函数**对话框中，输入**函数名称**，单击**提交**。

    3.  在ODPS SQL节点编辑页面输入如下语句。

        ```
        --查询表mq_data数据。
        SELECT * from mqdata;
        --获取JSON文件中的EXPENSIVE值。
        SELECT GET_JSON_OBJECT(mqdata.MQdata,'$.expensive') FROM mqdata;
        ```

    4.  单击![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9993359951/p100471.png)图标运行代码。

    5.  您可以在**运行日志**查看运行结果。


