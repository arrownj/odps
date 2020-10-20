---
keyword: security model
---

# Security models

This topic describes the security models of MaxCompute and DataWorks. The security model of MaxCompute can be used by MaxCompute project owners and security administrators for security purposes when they perform routine O&M and data operations. Before you configure security functions, we recommend that you read this topic to familiarize yourself with the security models.

A security model can be configured for MaxCompute and [DataWorks](). If you use MaxCompute in the DataWorks console but the security model of DataWorks does not meet the security requirements of your business, you must use the security models of MaxCompute and DataWorks.

## MaxCompute security model

-   Security system

    MaxCompute supports multi-tenant data security, which has the following benefits:

    -   [User authentication](/intl.en-US/Management/Configure security features/Manage users and permissions/User authentication.md)

        MaxCompute supports two account systems: the Alibaba Cloud account system and RAM user system. Note that MaxCompute recognizes RAM users but does not recognize RAM permissions. You can add RAM users under your Alibaba Cloud account to a MaxCompute project. However, MaxCompute does not consider the RAM permission definitions when it verifies the permissions of RAM users.

    -   [User and authorization management](/intl.en-US/Management/Configure security features/Manage users and permissions/Manage users.md)

        User management operations such as adding and removing users and granting permissions to users are supported for MaxCompute projects. You can manage permissions by using roles. An admin role is automatically provided for each project. MaxCompute supports both ACL-based authorization and policy-based authorization. This topic describes only ACL-based authorization.

        ACL-based authorization uses a syntax that is similar to the syntax of the GRANT and REVOKE statements defined in SQL-92. MaxCompute uses simple authorization statements to grant or revoke permissions on objects in your project. Syntax used by ACL-based authorization:

        ```
        grant actions on object to subject;
        revoke actions on object from subjec;
        ```

    -   [LabelSecurity](/intl.en-US/Management/Configure security features/Column-level access control.md)

        LabelSecurity is a mandatory access control \(MAC\) policy used for projects. It allows project administrators to flexibly control user access to sensitive data in columns.

    -   [Package-based resource sharing across projects](/intl.en-US/Management/Configure security features/Resource share across project space/Package-based resource sharing across projects.md)

        You can share data and objects among projects by using packages. The objects include tables, resources, and functions in a project. To implement this type of resource sharing, you only need to manage users in your project.

    -   [Project data protection](/intl.en-US/Management/Configure security features/Project data protection.md)

        Project data protection is used to prevent data outflow to other projects and resolve other similar issues.

-   Permissions, roles, and labels

    The security system provided by MaxCompute includes a variety of policies. These policies are used to grant permissions to users. Permissions that are granted to users are gradually extended. The following procedure describes how to grant permissions on an L4 table to a user by using policies:

    1.  If no permissions are granted to the user and the user does not belong to the project, add the user to the project first. The user does not have any permissions before the user is added to the project.
    2.  Grant operation permissions to the user by using the following methods. For more information, see [Authorize users](/intl.en-US/Management/Configure security features/Manage users and permissions/Authorize users.md).
        -   Grant a specific operation permission to the user.
        -   Grant an ACL to a role and then grant the role to the user. If a resource does not have a label, the user has obtained the permissions on the resource.
    3.  If the user manages resources that have [labels](/intl.en-US/Management/Configure security features/Column-level access control.md), such as datasheets and packages with datasheets, grant label permissions to the user. Four types of label permissions are provided:
        -   Permissions on fields in a datasheet
        -   Permissions on a datasheet \(This type of permission is not supported.\)
        -   Permissions on a package
        -   Permissions on a user \(Label permissions cannot be granted to a role.\)
    The following figure shows how permissions are granted by using fine-grained authorization and access control.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5143948951/p37987.png)

-   Relationship between project data protection and packages

    Project data protection is a security function. It is used to prevent batch data outflow to other projects in MaxCompute. After project data protection is enabled, if a trusted project group is not established in your project, data of other projects must be accessed by using packages. After a package is installed for a user, the user can grant permissions on the package to other users in the group.

    You can group some resources, such as commonly used tables and user-defined functions \(UDFs\), into a package, and then grant the permissions on this package to another project.

    In some scenarios, data outflow is allowed after you configure exception policies that are specific to application IP addresses and Alibaba Cloud accounts.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5143948951/p37993.png)


## DataWorks security model

DataWorks is a platform that allows multiple users to work together on a project for data analytics. The security model of DataWorks must meet the following requirements:

1.  Security isolation of data from different organizations
    -   Authentication and interworking with [RAM]() are supported. You can use your Alibaba Cloud account to create and activate a DataWorks project, and then grant RAM users under your Alibaba Cloud account the permissions to access resources in the created project.
    -   Projects created by an Alibaba Cloud account belong to an organization. You can configure task dependencies for projects. This way, the data of various tasks from the projects that are created by using different accounts is isolated.
2.  Security in the extract, transform, and load \(ETL\) processes. To achieve this objective, you must ensure that no changes occur in production tasks, and different members can perform only operations within their permission scope, such as code editing and debugging, and production task publishing.

    DataWorks distinguishes between [development and production projects]() by service. This ensures that task development and debugging are isolated from stable production. You can use roles to specify which members develop and debug tasks and which members are responsible for the O&M of production tasks.

3.  Permissions to access objects in a MaxCompute project

    MaxCompute has a security model. Project members require the permissions to access objects in a MaxCompute project during ETL. The objects include tables, resources, functions, and instances. When a MaxCompute project is created, MaxCompute roles that correspond to DataWorks roles are also created and granted the related permissions.


