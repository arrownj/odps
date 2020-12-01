---
keyword: [MCQA, query acceleration]
---

# Overview

This topic describes the system architecture, benefits, scenarios, and limits of the MaxCompute Query Acceleration \(MCQA\) feature.

## Description

The MCQA feature of MaxCompute accelerates the execution of small- and medium-sized query jobs and reduces the execution time from minutes to seconds. This feature is compatible with other query features of MaxCompute.

You can use MCQA to connect mainstream business intelligence \(BI\) tools or SQL clients to MaxCompute projects to perform ad hoc queries or BI analysis.

MCQA uses an independent resource pool that does not use quota groups. MCQA automatically identifies query jobs and shortens the job queue length, which improves user experience.

The billing method of MCQA query jobs is pay-as-you-go, which is the same as SQL query jobs. For more information, see [Pay-as-you-go billing for MCQA jobs](/intl.en-US/Pricing/Computing pricing.md).

**Note:** If an MCQA query job is rolled back to an SQL query job, it is charged based on the billing method of the SQL query job. After the public preview ends, you are charged for MCQA query jobs based on the billing method of the SQL query jobs. For more information about the commercial availability of the MCQA feature, see [Announcements](/intl.en-US/Release Note/Announcements.md).

## Benefits

MCQA has the following benefits:

-   Low latency

    Uses an efficient and low-latency resource scheduling policy and an independent resource pool.

-   Automatic identification

    Automatically identifies the size of query jobs. The system can use MCQA to accelerate the queries or process multiple query jobs at the same time and then return the query results. This allows you to analyze query jobs of different sizes or complexities.

-   Syntax compatibility

    Uses the same SQL syntax as MaxCompute.

-   Automatic selection of query methods

    Allows clients to use MCQA or batch processing to run query jobs. You can also configure clients to use MCQA in latency-sensitive scenarios.


## Scenarios

The following table describes the scenarios of MCQA.

|Use scenario|Description|Applicable scope|
|------------|-----------|----------------|
|Ad hoc query|You can use MCQA to optimize the query performance of small- and medium-sized datasets \(less than 100 GB\) and perform low-latency queries on MaxCompute tables. This helps develop and analyze data.|You can specify conditions based on your requirements, obtain query results, and adjust the query logic. In this scenario, the query latency must be within dozens of seconds. Users are data developers or data analysts who have mastered SQL skills and prefer to use familiar client tools to analyze queries.|
|BI analysis|If you use MaxCompute to build an enterprise-class data warehouse, MaxCompute performs an extract, transform, load \(ETL\) operation to process data into business-oriented and consumable aggregate data. MCQA features low latency and supports elastic concurrency and data caching. You can use MCQA with MaxCompute partition tables and buckets to execute concurrent jobs, generate reports, analyze statistics, and analyze fixed reports at a low cost.|The query object is usually aggregate data. This scenario is suitable for multi-dimensional queries, fixed queries, or high-frequency queries that contain small amounts of data. In this scenario, queries are latency-sensitive, and the results are returned in seconds. For example, the latency for most queries is less than five seconds. The time elapsed for each query varies based on the data size and query complexity.|
|Detailed queries and analysis of large amounts of data|MCQA automatically identifies the size of query jobs. MCQA can respond to and process small-sized jobs in a timely manner, and can allocate the resources required for large-sized jobs. This helps analysts run query jobs of different sizes and complexities.|In this scenario, large amounts of historical data are queried. The size of valid data that is queried is small, and the requirement for latency is not high. Users are business analysts who need to gain business insights from data, discover business opportunities, and validate business assumptions.|

## Limits

MCQA supports only SELECT statements and does not support user-defined functions \(UDFs\). If you submit a statement or feature that MCQA does not support from the MaxCompute client of V0.35.1 or later, the client automatically rolls back to the common offline mode to execute the statement or feature. Other tools cannot roll back to the common offline mode to execute the submitted statement or feature that MCQA does not support.

The following table describes the limits of MCQA.

|Item|Limit|
|----|-----|
|Feature|-   The MCQA feature is available only in MaxCompute Standard Edition that uses the pay-as-you-go billing method.
-   The MCQA feature does not support the subscription billing method. To use MCQA for a project, purchase a pay-as-you-go resource package. If the project uses a subscription resource package, make sure that you are granted the cross-project access permissions. You can also create a project that uses the pay-as-you-go billing method.
-   The MCQA feature is unavailable in MaxCompute Developer Edition. You must upgrade MaxCompute to the Standard Edition. |
|Query|-   If more than 1,000 MCQA query jobs are executed at the same time, the system automatically rolls back these MCQA query jobs to SQL query jobs.
-   If you submit an MCQA query job from a client, the default timeout period is 30s. If you submit an MCQA query job from the ad hoc query module of DataWorks, the default timeout period is 20s. If the MCQA query job times out, the system automatically rolls back the MCQA query job to an SQL query job.
-   MCQA can cache only data that is stored in ALIORC tables into memory to accelerate queries.
-   You cannot use MCQA to query data from external tables. |
|Query concurrency|You can run a maximum of 120 concurrent MCQA query jobs in each MaxCompute project.|

