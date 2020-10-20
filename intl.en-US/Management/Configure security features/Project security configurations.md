---
keyword: project security configurations
---

# Project security configurations

MaxCompute is a platform that allows multiple tenants to simultaneously process data. When these tenants process data on MaxCompute, they may have different requirements for data security. To meet their requirements, MaxCompute allows you to configure security settings for projects. Project owners can specify an external account system that is supported by MaxCompute and an authentication model to suit their needs.

MaxCompute provides multiple methods of orthogonal authorization, including ACL-based authorization and implicit authorization. If implicit authorization is used, an object creator is automatically granted the permissions to access the newly created object. Not all users require this security mechanism. Users can properly configure the project authentication model based on their service security requirements and usage patterns. Commands for project security configurations:

-   View security configurations of a project.

    ```
    show SecurityConfiguration
    ```

-   Enable or disable ACL-based authorization. The default value of CheckPermissionUsingACL is true.

    ```
    set CheckPermissionUsingACL=true/false
    ```

-   Allow or disallow an object creator to be granted the permissions to access the newly created object. The default value of ObjectCreatorHasAccessPermission is true.

    ```
    set ObjectCreatorHasAccessPermission=true/false
    ```

-   Allow or disallow an object creator to be granted the authorization permission. The default value of ObjectCreatorHasGrantPermission is true.

    ```
    set ObjectCreatorHasGrantPermission=true/false
    ```

-   Enable or disable project data protection to allow or disallow data outflow to other projects.

    ```
    set ProjectProtection=true/false 
    ```


**Note:** You can also use DataWorks to configure security settings for a project in a visualized manner. For more information, see [Configure MaxCompute]().

