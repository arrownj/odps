---
keyword: user authentication
---

# User authentication

This topic describes how to create an Alibaba Cloud account and an AccessKey pair, and use the created account to log on to MaxCompute for authentication.

## View the supported account system

MaxCompute supports both the Alibaba Cloud account system and RAM user system.

By default, MaxCompute projects recognize only the Alibaba Cloud account system. You can manually add support for the RAM user system.

-   View the account system that is supported by a MaxCompute project.

    ```
    list accountproviders;
    ```

-   Add support for the RAM user system.

    ```
    add accountprovider ram;
    ```

    After the support for the RAM user system is added, you can run the `list accountproviders;` command to check whether the supported account systems are updated.

    **Note:** MaxCompute projects recognize only the RAM user system. The RAM permission system is not recognized. After RAM users under your Alibaba Cloud account are added to a MaxCompute project, MaxCompute authenticates these RAM users but does not consider the permission definitions in RAM.


## Create an Alibaba Cloud account

If you do not have an Alibaba Cloud account, go to [Alibaba Cloud](https://www.alibabacloud.com/) to create an account.

**Note:** You must enter a valid email address during the creation. The reason is that this email address is used as the Alibaba Cloud account after the creation. For example, if Alice uses her email address alice@aliyun.com to create an Alibaba Cloud account, her account name is alice@aliyun.com

## Create an AccessKey pair

After you create an Alibaba Cloud account, you can create or manage your AccessKey pair list on the [Security Management page](http://i.aliyun.com/access_key).

An AccessKey pair consists of an AccessKey ID and an AccessKey secret. The AccessKey ID is used to retrieve the access key, whereas the AccessKey secret is used to calculate the signature of a request. You must keep your AccessKey pair confidential for further use. To update an AccessKey pair, you must create another pair and disable the existing one.

## Log on to MaxCompute with an Alibaba Cloud account

To log on to MaxCompute from the MaxCompute client \(odpscmd\), you must configure the AccessKey pair information in the conf/odps\_config.ini configuration file.

```
 project_name=myproject
 access_id=xxxx ## The AccessKey ID.
 access_key=xxxx  ## The AccessKey secret.
 end_point= http://service.odps.aliyun-inc.com/api ## The endpoint of the region where the MaxCompute service is deployed. For more information, see [Configure endpoints](/intl.en-US/Prepare/Configure endpoints.md).
```

**Note:** It requires about 15 minutes for you to enable or disable an AccessKey pair.

