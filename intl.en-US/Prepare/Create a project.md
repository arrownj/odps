---
keyword: [MaxCompute projects, workspaces, create projects]
---

# Create a project

A project is the basic operation unit in MaxCompute. User isolation and access control are implemented based on projects. After you activate MaxCompute, you must create projects to use MaxCompute. This topic describes how to create a project in the MaxCompute or DataWorks console. After you create a project, you can use MaxCompute in the project.

-   [DataWorks](https://common-buy-intl.alibabacloud.com/?commodityCode=dide_create_post_intl#/buy) and [MaxCompute](/intl.en-US/Prepare/Activate MaxCompute.md) are activated by using your Alibaba Cloud account or RAM user.
-   The same region is selected to activate DataWorks and MaxCompute.
-   The RAM user for you to log on to the official Alibaba Cloud website and to create a MaxCompute project is valid and granted the required permissions. For more information, see [Create RAM users](/intl.en-US/Prepare/Create RAM users.md).

You can use the MaxCompute or DataWorks console to create MaxCompute projects.

Take note of the following items:

-   After you create a workspace by using your Alibaba Cloud account, you have all operation permissions on the objects in your workspace. Only authorized users can access your workspace.
-   After a RAM user creates a workspace, the RAM user and its Alibaba Cloud account have operation permissions on all the objects in the workspace. Other users cannot access the workspace without authorization.
-   A RAM user can use MaxCompute after it joins in a workspace created by using its Alibaba Cloud account. Therefore, a RAM user does not need to create workspaces.

    **Note:** RAM users have limited permissions. We recommend that you create a workspace by using your Alibaba Cloud account and grant the required permissions on the workspace to your RAM user. For more information about the required permissions, see [Users and roles](/intl.en-US/Prepare/Users and roles.md).


## Use the MaxCompute console to create a project

DataWorks is used to manage projects of MaxCompute. Before you create a MaxCompute project, create a DataWorks workspace.

1.  Log on to the [MaxCompute console](https://workbench.data.aliyun.com/#/MCEngines), and select a region in the upper-left corner.

    ![MaxCompute console](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/8070276061/p177140.png)

2.  On the **Project management** tab, click **Create project**.

3.  In the **Create Workspace** panel, configure parameters to **create a DataWorks workspace**, and click **Create project**.

    ![Create a DataWorks workspace](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/3572945061/p177146.png)

    |Section|Parameter|Description|
    |:------|:--------|:----------|
    |**Basic Information**|**Workspace Name**|The name of the workspace. The name must be 3 to 23 characters in length, and can contain only letters, digits, and underscores \(\_\). It must start with a letter.|
    |**Display Name**|The display name of the workspace. The name must be 1 to 23 characters in length, and can contain only letters, digits, and underscores \(\_\). It must start with a letter.|
    |**Mode**|The mode of the workspace. Valid values: Basic Mode \(Production Environment Only\) and Standard Mode \(Development and Production Environments\). For more information, see [Basic mode and standard mode]().    -   Basic Mode \(Production Environment Only\): One DataWorks workspace is associated with only one MaxCompute project. A workspace in this mode cannot isolate the development environment from the production environment. In this workspace, you can perform only basic data development but cannot control the data development process or table permissions.
    -   Standard Mode \(Development and Production Environments\): One DataWorks workspace is associated with one MaxCompute development project and one MaxCompute production project. In this workspace, you can develop code in a standard manner and strictly control table permissions. Developers are prohibited from managing tables in the production environment without authorization. This ensures data security. |
    |**Description**|The description of the workspace.|
    |**Advanced Settings**|**Download SELECT Query Result**|Specifies whether workspace members can download the query results returned by SELECT statements in DataStudio. If you turn off the **Download SELECT Query Result** switch, workspace members cannot download the query results.|

4.  In the **Create Workspace** panel, configure parameters to **create a MaxCompute project** and click **Create project**.

    ![Create a MaxCompute project](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/4572945061/p177188.png)

    |Parameter|Description|
    |---------|-----------|
    |**Instance display name**|The name of the workspace. The name must be 3 to 23 characters in length, and can contain only letters, digits, and underscores \(\_\). It must start with a letter.|
    |**Payment mode**|The payment mode of MaxCompute. You must select the payment mode that you use to activate MaxCompute.|
    |**Quota group**|The quota group that determines the computing resources and disk spaces for the compute engine instance. For more information, see [Use MaxCompute Management](/intl.en-US/Management/Resource and job management/MaxCompute Management.md).|
    |**MaxCompute data type**|The data type edition of MaxCompute. Valid values: **MaxCompute V2.0 Data Type Edition \(Recommended\)**, **MaxCompute V1.0 Data Type Edition \(Suitable for Early MaxCompute Projects\)**, and **Hive-Compatible Data Type Edition \(Suitable for MaxCompute Projects Migrated from Hadoop\)**. For more information, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).|
    |**Whether to encrypt**|Specifies whether to enable the data encryption feature for the MaxCompute project.|
    |**Key**|The type of the key to be used in the MaxCompute project. Valid values: Dataworks Default Key and BYOK. If you select Dataworks Default Key, the key that MaxCompute automatically creates for the project in KMS will be used in the project.|
    |**Algorithm**|The encryption algorithm that is supported by the key. Valid values: AES256, AESCTR, and RC4.|
    |**Project Name**|The name of the MaxCompute project. If you create a DataWorks workspace in basic mode, the project name is set to the name you specified for the DataWorks workspace by default. If you create a DataWorks workspace in standard mode, the project name in the development environment is fixed in the format of Workspace name\_dev. In the production environment, the project name is set to the name you specified for the DataWorks workspace by default.|
    |**Access identity**|The identity that you use to access the MaxCompute project. In the development environment, the value is fixed to **Node Owner**.In the production environment, the valid values are **Alibaba Cloud Account** and **RAM User**. |

    After the MaxCompute project is created, you can view this project on the **Project management** tab.

    ![Complete the creation](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/8070276061/p177161.png)


## Use the DataWorks console to create a project

For more information, see [Create a workspace]().

