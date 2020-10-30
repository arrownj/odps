---
keyword: [MaxCompute data encryption, data encryption]
---

# Data encryption

MaxCompute uses Key Management Service \(KMS\) to encrypt data for storage. In this way, MaxCompute can provide static data protection to meet corporate governance and security compliance requirements. This topic describes the data encryption feature of MaxCompute, the limits of the feature, how to enable data encryption for a MaxCompute project, and the billing information.

## Overview

MaxCompute uses KMS to manage keys for data encryption and decryption. The data encryption feature of MaxCompute has the following characteristics:

-   MaxCompute uses KMS to encrypt or decrypt data in MaxCompute by project. Before you use the data encryption feature, make sure that [KMS is activated](https://common-buy-intl.alibabacloud.com/?spm=a3c0i.14467404.2075577850.3.26123c27Xw3mMk&commodityCode=kms_intl#/) in the target region.
-   KMS creates and manages customer master keys \(CMKs\) for you and keeps them secure.
-   MaxCompute supports the AES-256, AESCTR, and RC4 encryption algorithms.
-   MaxCompute can use the DataWorks default key and Bring Your Own Keys \(BYOKs\) to encrypt or decrypt data.
    -   When you create a MaxCompute project, you can set **Key** to **Dataworks Default Key**.

        MaxCompute automatically creates a key for the MaxCompute project in KMS and uses it as the CMK of the project. You can view the key information in the [KMS console](https://kms.console.aliyun.com/).

    -   To meet business and security requirements in different scenarios, MaxCompute can use BYOKs to encrypt or decrypt data.

        You can create BYOKs in the KMS console and select one as the CMK when you create a MaxCompute project. For more information about how to create a CMK in KMS, see [CreateKey](/intl.en-US/API Reference/Key/CreateKey.md).

    -   If a MaxCompute project needs to use a BYOK, you must complete Resource Access Management \(RAM\) authorization as prompted when you create the project.

## Limits

The data encryption feature of MaxCompute has the following limits:

-   If the data encryption feature is enabled for a MaxCompute project, you cannot use Hologres or Lightning to query data in the project.
-   You can enable the data encryption feature for a MaxCompute project only when you create the project. If you need to enable the feature for an existing MaxCompute project, you must [submit a ticket](https://workorder-intl.console.aliyun.com/) to the MaxCompute team.
-   Your operations such as the disable and delete operations on BYOKs in KMS affect data encryption and decryption in the corresponding MaxCompute projects. MaxCompute caches historical configurations. Your operations in KMS take effect in a delayed manner within 24 hours.

## Procedure for enabling data encryption

To enable the data encryption feature for a MaxCompute project, perform the following steps:

1.  Optional. Go to the [page for activating KMS](https://common-buy-intl.alibabacloud.com/?spm=a3c0i.14467404.2075577850.3.26123c27Xw3mMk&commodityCode=kms_intl#/), select **I agree with Key Management Service Agreement of Service**, and then click **Enable Now**.

    ![Activate KMS](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4343948951/p133962.png)

    **Note:** Skip this step if you have activated KMS in the target region.

2.  Log on to the [DataWorks console](https://workbench.data.aliyun.com/console). In the left-side navigation pane, click **Workspaces**.
3.  On the **Workspaces** page, select a region in the upper-left corner and click **Create Workspace**. In the **Create Workspace** pane, set the parameters in the **Basic Settings** step and click **Next**. For more information, see [Create a project](/intl.en-US/Prepare/Create a project.md).
4.  In the **Select Engines and Services** step, select **MaxCompute** in the **Compute Engines** section.
5.  In the **Perform ODPS service account authorization** dialog box, click **Authorization**.

    ![Perform ODPS service account authorization](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4343948951/p133964.png)

6.  On the **Cloud Resource Access Authorization** page, click **Confirm Authorization Policy**.

    ![Confirm Authorization Policy](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4343948951/p133965.png)

7.  Return to the **Perform ODPS service account authorization** dialog box and close it. In the **Select Engines and Services** step, select **MaxCompute** again and click **Next**.
8.  Set the parameters in the **Engine Details** step. Set the Whether to encrypt parameter to **Encryption** to enable the data encryption feature.

    In this example, create a workspace in the basic mode.

    ![Engine Details step](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4343948951/p133967.png)

    |Section|Parameter|Description|
    |-------|---------|-----------|
    |**MaxCompute**|**Instance display name**|The display name of the compute engine instance. The display name must be 3 to 27 characters in length. It must start with a letter and can contain letters, digits, and underscores \(\_\).|
    |**Resource Group**|The quota group that determines the computing resources and disk spaces for the compute engine instance. For more information, see [Use MaxCompute Management](/intl.en-US/Management/Resource and job management/MaxCompute Management.md).|
    |**MaxCompute Data Type Edition**|The data type edition of MaxCompute. Valid values: **MaxCompute V2.0 Data Type Edition \(Recommended\)**, **MaxCompute V1.0 Data Type Edition \(Suitable for Early MaxCompute Projects\)**, and **Hive-Compatible Data Type Edition \(Suitable for MaxCompute Projects Migrated from Hadoop\)**. For more information, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).|
    |**MaxCompute Project Name**|The name of the MaxCompute project to create. If you select Basic Mode \(Production Environment Only\) for Mode in the Basic Settings step, the value is set to the name you specified for the workspace. You can change the value. If you select Standard Mode \(Development and Production Environments\) for Mode in the Basic Settings step, the value is fixed to Name you specified for the workspace\_dev in the Development Environment section. In the Production Environment section, the value is set to the name you specified for the workspace, and you can change the value.|
    |**Account for Accessing MaxCompute**|The identity that you can use to access the MaxCompute project. For the development environment, the value is fixed to **Node Owner**.For the production environment, the valid values are **Alibaba Cloud Account** and **RAM User**. |
    |**Whether to encrypt**|Specifies whether to enable the data encryption feature for the MaxCompute project.|
    |**Key**|The type of the key to be used in the MaxCompute project. Valid values: Dataworks Default Key and BYOK. If you select Dataworks Default Key, the key that MaxCompute automatically creates for the project in KMS will be used in the project.|
    |**Algorithm**|The encryption algorithm that is supported by the key. Valid values: AES256, AESCTR, and RC4.|

9.  Click **Create Workspace**.

    After the data encryption feature is enabled, MaxCompute automatically encrypts and decrypts data that is written to and read from the MaxCompute project, respectively.


## Billing

You are not charged for enabling the data encryption feature for MaxCompute projects. During data encryption and decryption, MaxCompute interacts with the API of KMS, and you are charged for using KMS. For more information about billing, see [Billing](/intl.en-US/Pricing/Billing.md).

