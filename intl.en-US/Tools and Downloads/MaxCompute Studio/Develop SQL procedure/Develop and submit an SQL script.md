---
keyword: [write an SQL script, submit an SQL script]
---

# Develop and submit an SQL script

This topic describes how to develop and submit an SQL script in MaxCompute Studio. The SQL script development includes writing and running an SQL script.

-   A MaxCompute project connection is created. For more information, see [Manage project connections](/intl.en-US/Tools and Downloads/MaxCompute Studio/Manage project connections.md).
-   A MaxCompute Script module is created. For more information, see [t12125.md\#](/intl.en-US/Tools and Downloads/MaxCompute Studio/Develop SQL procedure/Create MaxCompute Script module.md).

## Write an SQL script

1.  In the **Project** tool window, click a project name, right-click **scripts**, and then choose **New** \> **MaxCompute SQL Script**.

    ![Create](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4734130061/p1845.png)

2.  In the **New MaxCompute SQL Script** dialog box, specify the required parameters and click **OK**.

    ![Script Name](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/3481871951/p1846.png)

    -   **Script Name**: the script name.
    -   **MaxCompute Project**: the MaxCompute project in which you want to write an SQL script. You can click the **+** button to create a MaxCompute project connection. For more information, see [Manage project connections](/intl.en-US/Tools and Downloads/MaxCompute Studio/Manage project connections.md).
3.  Write an SQL script in the compiler. For more information about the SQL syntax, see [MaxCompute SQL overview](/intl.en-US/Development/SQL/MaxCompute SQL overview.md).

    **Note:**

    -   You can configure cross-project resource sharing. For example, you can bind a script to Project A and grant access permissions on table1 in Project B to the script.
    -   MaxCompute Studio allows you to set an SQL script compiler. For more information, seeOverview.

## Submit an SQL script

Before you submit an SQL script, you must configure relevant settings as required. MaxCompute Studio provides multiple compiler features. You can set these features in the toolbar at the top of the compiler. You can set the following compilation parameters:

-   Compiler Mode:
    -   **Statement Mode**: In this mode, the compiler separates the SQL statements in the script with semicolons \(`;`\), and commits the statements one by one to MaxCompute for execution.
    -   **Script Mode**: In this latest development mode, the compiler submits a whole script to MaxCompute at a time. We recommend that you use this mode because this mode improves the overall execution efficiency.
-   Type System: You can set this parameter to avoid compatibility issues related to SQL statements. Valid values:
    -   **Legacy TypeSystem**: indicates the legacy type system of MaxCompute.
    -   **MaxCompute TypeSystem**: indicates the new type system introduced by MaxCompute V2.0.
    -   **Hive Compatible TypeSystem**: indicates the type system in the Hive compatibility mode introduced by MaxCompute V2.0.
-   Execution Mode:
    -   **Default Version**: indicates a stable version.
    -   **MaxCompute Query Acceleration**: indicates the new MaxCompute query acceleration \(MCQA\) feature.
    -   **Rerun When Acceleration Fails**: reruns the task when the query acceleration fails.

1.  In the toolbar or side bar, click the ![Run](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4734130061/p95128.png) icon to submit an SQL script to MaxCompute.

    ![Run](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/9616721061/p1916.png)

    **Note:** If a variable exists in the SQL script, such as $\{bizdate\} in the preceding figure, a dialog box appears, which prompts you to enter the variable value.

2.  Before the SQL job runs, IntelliJ IDEA reminds you of the estimated cost of the SQL job. Confirm the estimated cost and click **OK** in the **Confirmation** message.

    **Note:**

    -   In the toolbar, click the ![Refresh](../images/p95126.png) icon to update metadata that is used in the SQL script, such as tables and user-defined functions \(UDFs\). This feature applies when MaxCompute Studio cannot detect the tables and functions that exist in MaxCompute.
    -   SQL scripts are compiled based on the metadata that you add in the **Project Explorer** window. If no errors are detected, the scripts are submitted to MaxCompute for execution.
    -   Operational logs are generated during the SQL script execution. If SQL scripts are executed in MaxCompute, the job details tab is opened. You can view the basic information about the execution.
3.  On the **Result** tab, view the execution results.

    If multiple statements are executed one by one, the execution result for each statement is displayed.


