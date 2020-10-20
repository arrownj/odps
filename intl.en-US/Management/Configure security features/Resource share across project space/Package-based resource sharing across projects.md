---
keyword: resource sharing
---

# Package-based resource sharing across projects

This topic describes how to implement resource sharing across projects.

Assume that you are the project owner or administrator, and multiple projects are created under an Alibaba Cloud account. Objects in the prj1 project need to be shared with users of other projects. The objects include tables, resources, and custom functions. You can use the following methods for resource sharing:

-   Add users of other projects to the prj1 project and grant the users permissions one by one. This method is not recommended for resource sharing across projects because the operations are complex. We recommend that you use this method if fine-grained control of resource sharing specific to users is required and you are a member of the business project team. For more information, see [Authorize users](/intl.en-US/Management/Configure security features/Manage users and permissions/Authorize users.md).
-   Use packages for resource sharing across projects. Packages are used to share data and resources across projects and grant permissions to users of different projects.

    If package-based resource sharing across projects is used, administrators of other projects can create a package that contains the required resources in the prj1 project, and then allow administrators of other projects to install this package. After this package is installed, administrators of other projects determine whether to grant the permissions on the package to users of their projects.


