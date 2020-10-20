---
keyword: MapReduce error codes
---

# Common MapReduce errors

This topic describes the common MapReduce error codes and their trigger conditions.

|Error code|Trigger condition|
|:---------|:----------------|
|ODPS-0710005: Exception occurred when connecting to meta store|A metastore connection error occurs.|
|ODPS-0710015 MR task encountered meta store exception|A metastore error occurs when a MapReduce task is running.|
|ODPS-0710025: Get partitions from meta store error|An error occurs when you obtain the partition information from metastore.|
|ODPS-0720001: Invalid table name format|The table name format specified by the user is invalid.|
|ODPS-0720015: PartKeys size do not match partVals size|The number of partKeys is inconsistent with that of partVals in the table schema.|
|ODPS-0720021: Column does not exist|The specified column does not exist.|
|ODPS-0720031 PARSER Cache resource between comma must not be empty|There are empty values between commas in a specified resource list.|
|ODPS-0720041: Resource not found|The specified resource does not exist.|
|ODPS-0720051: Resource table should not be a view|The specified resource table is a view.|
|ODPS-0720061: Table or partition not found for resource|The table or table partition that is specified as the resource does not exist.|
|ODPS-0720071: Total size of cache resources is too big|The total number or size of the specified resources exceeds the upper limit. The default maximum number of the specified resources is 256, and the default maximum size of the specified resources is 512 MB.|
|ODPS-0720081: Job has not specified mapper class|The Mapper class is not specified for the job.|
|ODPS-0720091: Column duplicate|Duplicate columns are specified in the job input.|
|ODPS-0720101: Input table should not be a view|The input table is a view.|
|ODPS-0720111: Output table should not be a view|The output table is a view.|
|ODPS-0720121: Invalid table partSpec|The partSpec of the specified table is invalid.|
|ODPS-0720131: Invalid multiple output|The multiple-output \(including duplicate labels, same output tables, and same partitions\) is invalid.|
|ODPS-0720141: Memory value out of bound|The specified memory value is out of the range \[256 MB, 12288 MB\].|
|ODPS-0720151: Cpu value out of bound|The specified CPU value is out of the range \[50, 800\].|
|ODPS-0720161: Invalid max attempts value|The specified value of odps.mapred.map/reduce.max.attempts is out of the range \(0, 6\).|
|ODPS-0720171: Invalid IO sort buffer|The specified I/O sort buffer is out of the range \(64 MB, odps.mapred.map/reduce.memory\).|
|ODPS-0720181: Classpath resource between comma must not be empty|Content between commas in classpath is empty.|
|ODPS-0720191: Invalid input split mode|The specified split mode is invalid.|
|ODPS-0720201: Invalid map split size|The specified odps.mapred.map.min/max.split.size is invalid.|
|ODPS-0720211: Invalid number of map tasks|The specified odps.mapred.map.tasks is out of the range \(1, 100000\).|
|ODPS-0720221: Invalid max splits number|The specified maximum sample split is invalid.|
|ODPS-0720231: Job input not set|No input is specified for the job.|
|ODPS-0720241: Num of map instance is too big|The number of map instances is greater than the value of odps.mapred.max.map.tasks.|
|ODPS-0720251: Num of reduce instance is invalid|The number of reduce instances is out of the range \[0, odps.mapred.reduce.tasks\].|
|ODPS-0720261: Invalid partition value|The specified partition value is invalid.|
|ODPS-0720271: Allow no input is conflict to split mode|When you set setallownoinput to true, you set a split mode that is different from that of ALLOW\_NO\_INPUT.|
|ODPS-0720281: Invalid partitition format|The partition format is invalid.|
|ODPS-0720291:Invalid description json|The JSON format of the input description is invalid.|
|ODPS-0720301:Too many job input|The number of inputs specified exceeds the value of odps.mapred.map.max.input.num. The default value of odps.mapred.map.max.input.num is 1024.|
|ODPS-0720311:Invalid output label|The format of the specified output label is invalid.|
|ODPS-0720321: Too many job output|The number of specified outputs exceeds the value of odps.mapred.max.output.num. The default value of odps.mapred.max.output.num is 256.|
|ODPS-0720331: Too many cache resources|The specified number of resources exceeds the value of odps.mapred.job.max.cache.resource.num.|
|ODPS-0730005:MR job internal error|An internal error that is out of user control occurs in a MapReduce task.|

