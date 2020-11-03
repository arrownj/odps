---
keyword: [resource, resource, MapReduce, Archive]
---

# Resource

This topic introduces the concept of MaxCompute resources on which specific operations in MaxCompute depend.

## Concept

Resources are a unique concept of MaxCompute. To accomplish tasks by using [user-defined functions \(UDFs\)](/intl.en-US/Development/SQL/UDF/Overview.md) or [MapReduce](/intl.en-US/Development/MapReduce/Summary/MapReduce.md), you must upload files as MaxCompute resources.

-   SQL UDF: After you write a UDF, you must compile it as a JAR package and upload the package to MaxCompute as a resource. Then, when you run the UDF, MaxCompute automatically downloads the JAR package and obtains the code to run the UDF. JAR packages are a type of MaxCompute resources. When you upload a JAR package, a resource is created in MaxCompute.
-   MapReduce: After you write a MapReduce program, you must compile it as a JAR package and upload the package to MaxCompute as a resource. Then, when you run a MapReduce job, the MapReduce framework automatically downloads the JAR package and obtains the code to run the MapReduce job. You can upload text files and MaxCompute tables to MaxCompute as different types of resources. Then, you can read or use these resources when you run UDFs or MapReduce jobs.

MaxCompute provides APIs for you to read and use resources. For more information, see [Resource samples](/intl.en-US/Development/MapReduce/Program Example/Resources.md) and [Java UDF](/intl.en-US/Development/SQL/UDF/Java UDF.md).

**Note:** For more information about limits on how MaxCompute [UDFs](/intl.en-US/Development/SQL/UDF/Overview.md) and [MapReduce](/intl.en-US/Development/MapReduce/Summary/MapReduce.md) access resources, see [Limits](/intl.en-US/Development/MapReduce/Limits.md).

## Resource type

The maximum size that MaxCompute supports for a single resource is 500 MB. MaxCompute supports the following types of resources:

-   File
-   Table: tables in MaxCompute.

    **Note:** Only BIGINT, DOUBLE, STRING, DATETIME, and BOOLEAN fields are supported in tables referenced by MapReduce.

-   JAR: compiled Java JAR packages.
-   Archive: compressed files identified by the resource name extension. Supported file types include .zip, .tgz, .tar.gz, .tar, and .jar.

For more information about resource operations, see [Add a resource](/intl.en-US/Development/Common commands/Resource operations.md), [Remove a resource](/intl.en-US/Development/Common commands/Resource operations.md), [View a resource list](/intl.en-US/Development/Common commands/Resource operations.md), and [View resource information](/intl.en-US/Development/Common commands/Resource operations.md).

