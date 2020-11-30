---
keyword: query editor
---

# Query editor

The MaxCompute console provides a query editor for you to execute SQL statements and analyze data. This topic describes how to use the query editor.

## Overview

The query editor of MaxCompute is integrated into the DataAnalysis tool of DataWorks. You can use the query editor to edit MaxCompute SQL statements, query data, analyze data in workbooks, share data online, and download data. You can use the query editor to perform the following operations:

-   Edit and execute SQL statements and authorization commands, such as ACL-based authorization commands.
-   Use and test MaxCompute based on its public datasets. The public datasets are open to users by default.
-   Analyze, download, and share query results by using EXCEL online.

## Scenarios

The query editor can be used in the following scenarios:

-   When you use and test MaxCompute for the first time, you can use the query editor to experience the core features of MaxCompute based on the data in the public datasets.
-   If you are a data analyst, you can use the query editor to query data. Then, you can switch to the analysis mode to analyze the query results by using EXCEL. To reduce the frequency at which data is transferred and ensure data security, you can also download the query results to your machine for analysis.
-   If you are a security administrator, you can find a required project and click **Project permission management** in the Actions column to manage role permissions. However, this feature is in trial use. You must run commands to manage permissions in most scenarios. The query editor allows you to run most of the security commands without the need to perform additional configurations.

## Open the query editor

1.  Log on to the [MaxCompute console](https://workbench.data.aliyun.com/#/MCEngines), select a region in the upper-left corner, and then click **Query editing** to open the query editor.

    ![Query editing](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/5158824061/p170391.png)

2.  In the **Select Datasource** dialog box, select **MaxCompute** for **Type** and select an existing project from the **Workspace** drop-down list.

    ![Select a project](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/2577404061/p170358.png)

    **Note:** If you select a workspace that is in standard mode and submit a job in the query editor, the job is marked with dev. This indicates that the job is submitted in a development project.

3.  Click **OK**. The query editor page appears.

## Query mode and analysis mode

The query editor can work in query mode or analysis mode. In the upper-left corner of the DataAnalysis page, you can click **Query mode** or **Analysis mode** to switch between them. This feature is integrated into the DataAnalysis tool of DataWorks. For more information, see [Data analysis](). The components displayed in query mode are different from those displayed in analysis mode.

-   Query mode

    ![Query mode](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/4921276061/p170822.png)

    |No.|Description|
    |---|-----------|
    |1|The working mode of the query editor. In this figure, the query editor works in query mode. You can click Query mode to switch to the Analysis mode.|
    |2|The name of the workspace.|
    |3|Tables in the workspace, tables that have been used, and public datasets.|
    |4|The editor, which allows you to execute SQL statements and authorization commands, such as ACL-based authorization commands. On the query editor page, you can edit a script without the need to create a file. You can also save the script as a file.|
    |5|The tabs that allow you to view the saved query files, history, logs, and results.|

-   Analysis mode

    ![Analysis mode](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/4921276061/p170823.png)

    |No.|Description|
    |---|-----------|
    |1|The working mode of the query editor. In this figure, the query editor works in analysis mode. You can click Analysis mode to switch to the Query mode.|
    |2|The toolbar that you use to edit and manage data in a workbook.|
    |3|The query results.|


