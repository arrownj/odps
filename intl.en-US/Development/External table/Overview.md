---
keyword: external table
---

# Overview

MaxCompute is the core computing component of the Alibaba Cloud big data platform and provides powerful computing capabilities. MaxCompute schedules a large number of nodes to run concurrent computing jobs. It also provides systematic processing and management mechanisms for distributed computing features, such as failovers and retries.

## Background information

MaxCompute SQL provides an entry for distributed data processing. This allows you to quickly process and store exabytes of offline data. The computing framework of MaxCompute continues to evolve to meet the requirements of expanded big data business and newly emerged use scenarios. In early versions, MaxCompute provides powerful computing capabilities to process internal data in special formats. It is now available to process external data.

MaxCompute SQL now focuses on the structured data that is stored by using the **CFlie** column store format in MaxCompute internal tables. You must use different tools to import external user data to MaxCompute tables for computations. The user data includes texts and unstructured data. For example, to process Object Storage Service \(OSS\) data in MaxCompute, you can use one of the following methods:

-   Use OSS SDK or other tools to download data from OSS. Then, use MaxCompute Tunnel to import the downloaded data to a table.
-   Write a user-defined function \(UDF\) to call OSS SDK and access OSS data.

However, both methods have deficiencies.

-   The first method requires data transfer operations outside the MaxCompute system. If large amounts of OSS data need to be processed, concurrent operations are required to accelerate the process. As a result, you cannot fully utilize the large-scale computing capabilities of MaxCompute.
-   The second method requires UDF-based access permissions. It also requires that developers control the number of concurrent jobs and handle issues related to data partitioning.

MaxCompute provides external tables to handle these issues. External tables are used to process data that is not stored in MaxCompute internal tables. You can execute a simple Data Definition Language \(DDL\) statement to create an external table in MaxCompute. Then, you can use this table to associate MaxCompute with external data sources. This allows access to and output of data in various formats. In most cases, external tables can be accessed like standard MaxCompute tables. You can fully utilize the computing capabilities of MaxCompute SQL to process external data.

**Note:**

-   If you use an external table, the data in this table is not stored in MaxCompute, and no storage costs are generated.
-   Full search is supported for external tables.
-   Tunnel commands and Tunnel SDK are unavailable for external tables. You can use Tunnel to upload data to MaxCompute internal tables. You can also use OSS SDK for Python to upload data to OSS and map the data to external tables in MaxCompute.
-   You can create, search, configure, and process external tables in the DataWorks console. You can also query and analyze data in external tables. For more information, see [External table]().

## Examples

This section describes how to use MaxCompute external tables to process unstructured data:

-   To access unstructured data in OSS and Tablestore, see [Access OSS data](/intl.en-US/Development/External table/Access OSS data by using the built-in extractor.md) and [Access Tablestore data](/intl.en-US/Development/External table/Access Table Store data.md).
-   To use external tables to access OSS data, you must be granted the permissions to access OSS from MaxCompute. The authorization is performed in the Resource Access Management \(RAM\) console. For more information, see [STS authorization](/intl.en-US/Development/External table/STS authorization.md).
-   The unstructured data processing framework of MaxCompute allows you to export MaxCompute data to OSS by using the INSERT statement. For more information, see [Write unstructured data to OSS](/intl.en-US/Development/External table/Export unstructured data to OSS.md).
-   For more information about how to process data in various open source formats, see [Process OSS data stored in open source formats](/intl.en-US/Development/External table/Process OSS data stored in open source formats.md).

