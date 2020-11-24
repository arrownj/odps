---
keyword: [MaxCompute, MaxCompute, data warehousing solution]
---

# What is MaxCompute?

MaxCompute \(formerly known as ODPS\) is a fast and fully managed computing platform for large-scale data warehousing. It can process exabytes of data.

As data collection techniques are increasingly diverse, industries amass such large amounts \(terabytes, petabytes, or even exabytes\) of data that the traditional software industry cannot handle. Against this backdrop, MaxCompute is designed to store and compute large amounts of structured data. It provides various data warehousing solutions as well as analytics and modeling services.

Given the massive data amount, the limited processing capability of a single server has prompted data analysts to move towards distributed computing models. However, the distributed computing models are not easy to maintain and demand highly qualified data analysts. The data analysts must understand their business requirements and be familiar with the underlying distributed computing models. MaxCompute provides comprehensive data import solutions and a variety of typical distributed computing models. By using MaxCompute, you can complete big data analytics without knowledge about distributed computing and maintenance.

MaxCompute is available in 16 countries and regions around the world, with customers in the finance, Internet, biomedical, energy, transportation, and media industries. MaxCompute provides large-scale data storage and computing services for global users. Several MaxCompute customer cases won the 2017 Big Data Outstanding Product and Application Solution Cases Award that was issued by the Ministry of Industry and Information Technology of China. In addition, Alibaba Cloud, represented by MaxCompute, DataWorks, and AnalyticDB, was positioned as a contender in Cloud Data Warehouse \(CDW\) in the Forrester Wave™: Cloud Data Warehouse, Q4 2018 report.

**Note:** MaxCompute is widely used by Alibaba Group in scenarios such as data warehousing and business intelligence \(BI\) analysis for large Internet enterprises, website log analysis, transaction analysis for e-commerce websites, and customer behavior analysis.

MaxCompute seamlessly integrates with DataWorks, which provides a variety of features including data synchronization, workflow design, data development, data management, and O&M for MaxCompute. For more information, see [What is DataWorks?]()

## MaxCompute learning path

For more information about the concepts, basic operations, and advanced operations of MaxCompute, see [MaxCompute learning path](https://www.alibabacloud.com/getting-started/learningpath/maxcompute).

## Benefits

-   Large-scale computing and storage

    MaxCompute can store and compute up to exabytes of data. MaxCompute is suitable if you have more than 100 GB of data to store and compute.

-   Multiple computing models

    MaxCompute supports multiple computing models and Message Passing Interface \(MPI\) iterative algorithms. The computing models supported include SQL, MapReduce, user-defined functions \(UDFs\) in Java or Python, Graph, directed acyclic graph \(DAG\) based processing, interactive analytics, in-memory computing, and machine learning. MaxCompute simplifies the application architecture of the big data platform for enterprises.

-   Strong data security
    -   MaxCompute has steadily supported all data warehouse business of Alibaba for more than nine years, providing multi-layer sandboxing, fine-grained permission management, and monitoring.
    -   MaxCompute has passed an independent third-party audit on compliance with the trust services criteria for security, availability, and confidentiality established by American Institute of Certified Public Accountants \(AICPA\). For more information about the audit report, see [SOC 3 Report](https://www.alibabacloud.com/zh/trust-center/soc).
-   Cost-efficiency

    Compared with an on-premises private cloud, MaxCompute is more efficient in computing and storage. MaxCompute can reduce your procurement costs by 30% to 50%.

-   O&M-free

    MaxCompute is designed based on the serverless concept. MaxCompute allows you to focus on jobs and data rather than the underlying distributed architecture and O&M.

-   Elastic scalability

    MaxCompute provides job-level resource management based on the pay-as-you-go billing method. MaxCompute automatically expands computing, storage, and network resources based on your requirements, which greatly reduces costs.


## System architecture

MaxCompute is a big data computing service that provides multiple computing models and APIs to meet a wide range of data analytics requirements. You can use all services of MaxCompute immediately after you activate MaxCompute.

![](../images/p45239.jpg)

## Features

-   Data tunnels
    -   [Tunnel service for transmitting batch or historical data](/intl.en-US/Development/Data upload and download/Tunnel SDK/MaxCompute Tunnel overview.md#)

        [Tunnel](/intl.en-US/Development/Data upload and download/Tunnel SDK/MaxCompute Tunnel overview.md) is a data transmission service that MaxCompute provides for you to upload and download offline data in high concurrency. Tunnel supports daily import and export of terabytes or petabytes of data. Tunnel is particularly useful for batch import of full or historical data. Tunnel supports the Java API. You can use commands on the MaxCompute client to exchange files and data with the cloud.

    -   DataHub service for transmitting real-time incremental data

        MaxCompute provides DataHub for you to upload real-time data. DataHub features low latency and is easy to use. DataHub is particularly useful for incremental data imports. DataHub supports a variety of data transmission plug-ins, such as Logstash, Flume, Fluentd, and Sqoop. DataHub can also deliver logs to MaxCompute by using Log Service. Then, you can use DataWorks to analyze log data.

-   Computing and analysis tasks

    MaxCompute supports the following computing models:

    -   [SQL](/intl.en-US/Development/SQL/MaxCompute SQL overview.md): MaxCompute stores data in tables, supports multiple [data type editions](/intl.en-US/Development/Data types/Data type editions.md), and provides SQL query capabilities. You can use MaxCompute similarly to traditional database software but with the ability to process terabytes or petabytes of data.

        **Note:**

        -   MaxCompute SQL does not support transactions, indexing, or UPDATE and DELETE operations.
        -   The SQL syntax of MaxCompute is different from that of Oracle or MySQL. You cannot seamlessly migrate SQL statements from other databases to MaxCompute.
        -   MaxCompute is suitable to compute more than 100 GB of data. MaxCompute SQL can return query results in minutes or seconds, but not in milliseconds.
        -   MaxCompute SQL is easy to use. You do not need to understand distributed computing. If you have experience in database operations, you can be familiar with MaxCompute SQL.
    -   [UDF](/intl.en-US/Development/SQL/UDF/Overview.md): a user-defined function.

        MaxCompute provides a variety of [built-in functions](/intl.en-US/Development/SQL/Builtin functions/Date functions.md) to meet your computing requirements. You can also create UDFs.

    -   [MapReduce](/intl.en-US/Development/MapReduce/Summary/Overview.md): a Java MapReduce programming model that is provided by MaxCompute. MapReduce simplifies the development process and is more efficient. To use MapReduce in MaxCompute, you must have a basic understanding of distributed computing and relevant programming experience. MapReduce provides the Java API.
    -   [Graph](/intl.en-US/Development/Graph/Overview.md): an iterative graph computing framework that is provided by MaxCompute. Graph computing jobs use graphs to build models. A graph consists of vertices and edges that have values. You can edit and evolve a graph through iteration to obtain the final result. Typical applications include [PageRank](/intl.en-US/Development/Graph/Examples/PageRank.md), [single source shortest path \(SSSP\) algorithm](/intl.en-US/Development/Graph/Examples/SSSP.md), and [K-means clustering algorithm](/intl.en-US/Development/Graph/Examples/K-means clustering.md).
    -   Spark on MaxCompute: a big data analytics engine that is designed by Alibaba Cloud to provide big data processing capabilities. For more information, see [Spark on MaxCompute overview](/intl.en-US/Development/Spark/Spark on MaxCompute overview.md).
-   SDK

    MaxCompute SDK is a toolkit provided for developers. Both the [SDK for Java](/intl.en-US/SDK Reference/Java SDK/Java SDK.md) and [SDK for Python](/intl.en-US/SDK Reference/Python SDK/SDK for Python.md) are provided.

-   Security

    MaxCompute offers powerful security services to protect your data. For more information, see the [security guide](/intl.en-US/Management/Configure security features/Target users.md).


