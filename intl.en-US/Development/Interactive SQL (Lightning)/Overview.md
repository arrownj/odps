---
keyword: MaxCompute Lightning
---

# Overview

MaxCompute Lightning is an interactive query service that is provided by MaxCompute. MaxCompute Lightning complies with the PostgreSQL standards and syntax and allows you to use familiar tools and standard SQL statements to query and analyze data in MaxCompute projects.

You can use mainstream business intelligence \(BI\) tools, such as Tableau and FineReport, or SQL clients to access data in your MaxCompute projects for BI analysis or ad hoc queries. You can also use the quick query feature of MaxCompute Lightning to expose your table data as APIs for external services to call. This way, data can be applied in a variety of scenarios without migration.

MaxCompute Lightning provides serverless computing services. You do not need to manage any infrastructure and you pay only for running queries.

## Key features

-   Compatibility with PostgreSQL

    MaxCompute Lightning provides Java Database Connectivity \(JDBC\) and Open Database Connectivity \(ODBC\) APIs that are compatible with PostgreSQL. These APIs allow tools or applications that support PostgreSQL databases to connect to MaxCompute projects by using the PostgreSQL driver. You can also use PostgreSQL tools to analyze MaxCompute data.

-   High query performance

    The quick query feature of MaxCompute Lightning allows you to query data in MaxCompute tables more efficiently, especially in scenarios in which small datasets need to be scanned concurrently. You can use the quick query feature for various business purposes, for example, to generate fixed reports and expose data in MaxCompute tables as APIs.

-   Unified permission management

    As a service provided by MaxCompute, MaxCompute Lightning provides access to MaxCompute projects and shares the same permission system with MaxCompute projects. This ensures that users can query data that they are authorized to access.

-   Out-of-the-box feature and payment by query

    MaxCompute Lightning provides serverless computing services based on existing MaxCompute computing resources. You can use MaxCompute Lightning to connect to MaxCompute projects and query data without the need to configure, manage, or maintain MaxCompute Lightning resources.

    When you use MaxCompute Lightning, you are charged based only on the amount of data that is processed in each query.


## Architecture

MaxCompute Lightning provides an endpoint, which allows clients and applications to call the JDBC or ODBC API of MaxCompute Lightning by using the PostgreSQL driver. This way, clients and applications can connect to MaxCompute projects and access data in the projects under the unified permission system of MaxCompute projects.

The serverless computing services of MaxCompute Lightning are used to run the query tasks that are initiated and submitted by using the JDBC or ODBC API. This ensures the service quality.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5092659951/p11152.jpg)

## Scenarios

-   Ad hoc queries

    MaxCompute Lightning provides high performance for querying small datasets, that is, datasets that are smaller than 100 GB. You can use MaxCompute Lightning to directly query data in MaxCompute tables at low latency without the need to import data to other systems such as Analytic Database Service \(ADS\) and Relational Database Service \(RDS\). This helps you save resources and reduce costs.

    This scenario has the following characteristics: The data to query can be flexibly specified. The query logic is complex. Users want to obtain the query result as soon as possible and adjust the query logic. The expected query latency is within dozens of seconds. Users are data analysts who master SQL skills and want to use familiar client tools to query and analyze data.

-   Report analysis

    MaxCompute Lightning generates analysis reports based on MaxCompute project data on which extract, transform, load \(ETL\) is performed. Administrators and business staff regularly view the reports.

    This scenario has the following characteristics: The data to query is a small amount of aggregate data. The query logic is fixed and simple. Queries are latency-sensitive, and results must be returned in seconds. For example, the latency for most queries must be within 5 seconds. The time that is required for each query varies based on the size of the queried data and complexity of the query logic.

-   Online application

    Data in MaxCompute projects is exposed as RESTful APIs for online applications to call.

    This scenario has the following characteristics: MaxCompute Lightning, as an engine to accelerate queries, is used together with [DataService Studio]() of Alibaba Cloud DataWorks to expose data in MaxCompute tables as APIs. Manual development or O&M is not required in this process.


## Limits

For more information about the limits on DDL and DML statements, queries, and user-defined functions \(UDFs\), see [Constraints and limitations](/intl.en-US/Development/Interactive SQL (Lightning)/Constraints and limitations.md).

