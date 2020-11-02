---
keyword: [business data migration, log data migration, MaxCompute data migration]
---

# Overview

This topic describes the best practices for data migration, including migrating business data or log data from other business platforms to MaxCompute or migrating data from MaxCompute to other business platforms.

## Background information

Traditional relational databases are not suitable for processing a large amount of data. If you have a large amount of data stored in a traditional relational database, you can migrate the data to [MaxCompute](https://www.aliyun.com/product/odps?spm=a2c4g.11186623.2.7.48701099j4Wth9).

MaxCompute provides a comprehensive set of data migration solutions and a variety of classic distributed computing models, allowing you to store a large amount of data and compute data fast. By using MaxCompute, you can efficiently save costs for your enterprise.

[DataWorks](https://www.alibabacloud.com/product/ide) provides comprehensive features for MaxCompute, such as data integration, data analytics, data management, and data administration. Among these features, [data integration]() enables stable, efficient, and scalable data synchronization.

## Best practices

-   Migrate business data from other business platforms to MaxCompute
    -   Migrate data across DataWorks workspaces. For more information, see [Migrate data across DataWorks workspaces](/intl.en-US/Best Practices/Data migration/Migrate data across DataWorks workspaces.md).
    -   Migrate data from Hadoop to MaxCompute. For more information, see [Best practices of migrating data from Hadoop to MaxCompute](/intl.en-US/Best Practices/Data migration/Best practices of migrating data from Hadoop to MaxCompute.md). For more information about the issues that you may encounter during data and script migration and the solutions, see [Practices of migrating data from a user-created Hadoop cluster to MaxCompute](https://yq.aliyun.com/articles/630231).
    -   Migrate data from Oracle to MaxCompute. For more information, see [Migrate data from Oracle to MaxCompute]().
    -   Migrate data from a Kafka cluster to MaxCompute. For more information, see [Migrate data from a Kafka cluster to MaxCompute](/intl.en-US/Best Practices/Data migration/Migrate data from a Kafka cluster to MaxCompute.md).
    -   Migrate data from an Elasticsearch cluster to MaxCompute. For more information, see [Migrate data from an Elasticsearch cluster to MaxCompute](/intl.en-US/Best Practices/Data migration/Migrate data from an Elasticsearch cluster to MaxCompute.md).
    -   Migrate data from RDS to MaxCompute. For more information, see [Migrate data from RDS to MaxCompute to implement dynamic partitioning](/intl.en-US/Best Practices/Data migration/Migrate data from RDS to MaxCompute to implement dynamic partitioning.md).
    -   Migrate JSON data from Object Storage Service \(OSS\) to MaxCompute. For more information, see [Migrate JSON data from OSS to MaxCompute](/intl.en-US/Best Practices/Data migration/Migrate JSON data from OSS to MaxCompute.md).
    -   Migrate JSON data from MongoDB to MaxCompute. For more information, see [Migrate JSON data from MongoDB to MaxCompute](/intl.en-US/Best Practices/Data migration/Migrate JSON data from MongoDB to MaxCompute.md).
    -   Migrate data from a user-created MySQL database on an Elastic Compute Service \(ECS\) instance to MaxCompute. For more information, see [Migrate data from a user-created MySQL database on an ECS instance to MaxCompute]().
-   Migrate log data from other business platforms to MaxCompute
    -   Use Tunnel to migrate log data to MaxCompute. For more information, see [Use Tunnel to upload log data to MaxCompute](/intl.en-US/Best Practices/Data migration/Migrate log data to MaxCompute/Use Tunnel to upload log data to MaxCompute.md).
    -   Use DataHub to migrate log data to MaxCompute. For more information, see [Use DataHub to migrate log data to MaxCompute](/intl.en-US/Best Practices/Data migration/Migrate log data to MaxCompute/Use DataHub to migrate log data to MaxCompute.md).
    -   Use DataWorks to migrate log data to MaxCompute. For more information, see [Use DataWorks Data Integration to migrate log data to MaxCompute](/intl.en-US/Best Practices/Data migration/Migrate log data to MaxCompute/Use DataWorks Data Integration to migrate log data to MaxCompute.md).
-   Migrate data from MaxCompute to other business platforms
    -   Migrate data from MaxCompute to OSS. For more information, see [Migrate data from MaxCompute to OSS](/intl.en-US/Best Practices/Data migration/Migrate data from MaxCompute to OSS.md).
    -   Migrate data from MaxCompute to Tablestore. For more information, see [Migrate data from MaxCompute to Tablestore](/intl.en-US/Best Practices/Data migration/Migrate data from MaxCompute to Tablestore.md).

After the business data and log data are processed by MaxCompute, you can use Quick BI to present the data processing results in a visualized manner. For more information, see [Best practices of using MaxCompute to process data and Quick BI to present the data processing results](https://bp.aliyun.com/detail/106).

