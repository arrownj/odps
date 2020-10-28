---
keyword: limits
---

# Limits

This topic describes the limits of MaxCompute MapReduce. Your business may be affected if these limits are violated.

The following table describes the limits of MaxCompute MapReduce.

|Item|Value range|Classification|Configuration item|Default value|Configurable|Description|
|:---|:----------|:-------------|:-----------------|:------------|:-----------|:----------|
|Memory occupied by an instance|\[256 MB,12 GB\]|Memory|`odps.stage.mapper(reducer).mem` and `odps.stage.mapper(reducer).jvm.mem`|2,048 MB and 1,024 MB|Yes|The memory occupied by a single map or reduce instance. The memory consists of two parts: the framework memory, which is 2,048 MB by default, and Java Virtual Machine \(JVM\) heap memory, which is 1,024 MB by default.|
|Number of resources|256|Quantity|-|N/A|No|Each job can reference up to 256 resources. Each table or archive is considered as one resource.|
|Numbers of inputs and outputs|1,024 and 256|Quantity|-|N/A|No|The number of the inputs of a job cannot exceed 1,024, and that of the outputs of a job cannot exceed 256. A partition of a table is regarded as one input. The number of tables cannot exceed 64.|
|Number of counters|64|Quantity|-|N/A|No|The number of custom counters in a job cannot exceed 64. The counter group name and counter name cannot contain number signs \(\#\). The total length of the two names cannot exceed 100 characters.|
|Number of map instances|\[1,100000\]|Quantity|odps.stage.mapper.num|N/A|Yes|The number of map instances in a job is calculated by the framework based on the split size. If no input table is specified, you can set the odps.stage.mapper.num parameter to specify the number of map instances. The value ranges from 1 to 100,000.|
|Number of reduce instances|\[0,2000\]|Quantity|odps.stage.reducer.num|N/A|Yes|By default, the number of reduce instances in a job is 25% of the number of map instances. You can set the number to a value that ranges from 0 to 2,000. Reduce instances process much more data than map instances, which may result in long processing time in the reduce stage. A job can have 2,000 reduce instances at most.|
|Number of retries|3|Quantity|-|N/A|No|The maximum number of retries that are allowed for a map or reduce instance is 3. Exceptions that do not allow retries may cause jobs to fail.|
|Local debug mode|A maximum of 100 instances|Quantity|-|N/A|No|In local debug mode:

-   The number of map instances is 2 by default and cannot exceed 100.
-   The number of reduce instances is 1 by default and cannot exceed 100.
-   The number of download records for one input is 100 by default and cannot exceed 10,000. |
|Number of times a resource is read repeatedly|64|Quantity|-|N/A|No|The number of times that a map or reduce instance repeatedly reads a resource cannot exceed 64.|
|Resource bytes|2 GB|Length|-|N/A|No|The total bytes of resources that are referenced by a job cannot exceed 2 GB.|
|Split size|Greater than or equal to 1|Length|odps.stage.mapper.split.size|256 MB|Yes|The framework determines the number of map instances based on the split size.|
|Length of a string in a column|8 MB|Length|-|N/A|No|A string in a column cannot exceed 8 MB in length.|
|Worker timeout period|\[1,3600\]|Time|odps.function.timeout|600|Yes|The timeout period of a map or reduce worker when the worker does not read or write data, or stops sending heartbeats by using `context.progress()`. The default value is 600 seconds.|
|Field types supported by tables that are referenced by MapReduce|BIGINT, DOUBLE, STRING, DATETIME, and BOOLEAN|Data type|-|N/A|No|When a MapReduce task references a table, an error is returned if the table has field types that are not supported.|
|Object Storage Service \(OSS\) data read|-|Feature|-|N/A|No|MapReduce cannot read OSS data.|
|New data types in MaxCompute V2.0|-|Feature|-|N/A|No|MapReduce does not support the new data types in MaxCompute V2.0.|

