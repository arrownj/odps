---
keyword: MapReduce错误码
---

# MapReduce常见错误

本文为您介绍MapReduce常见错误码以及对应的触发条件。

|错误信息|触发条件|
|:---|:---|
|ODPS-0710005：Exception occurred when connecting to meta store|连接Meta Store时发生错误。|
|ODPS-0710015 MR task encountered meta store exception|运行MapReduce任务时发生Meta Store类错误。|
|ODPS-0710025：Get partitions from meta store error|从Meta Store获取分区信息时发生错误。|
|ODPS-0720001： Invalid table name format|用户指定非法的表名格式。|
|ODPS-0720015：PartKeys size do not match partVals size|表的schema partVal不一致。|
|ODPS-0720021： Column does not exist|指定不存在的列。|
|ODPS-0720031 PARSER Cache resource between comma must not be empty|指定的资源列表在逗号之间为空。|
|ODPS-0720041：Resource not found|指定了不存在的资源。|
|ODPS-0720051：Resource table should not be a view|指定的资源表是视图。|
|ODPS-0720061： Table or partition not found for resource|指定了不存在的表或者表分区为资源。|
|ODPS-0720071： Total size of cache resources is too big|指定的资源的总个数或者总大小超过限制。默认最大个数为256个、最大大小为512MB。|
|ODPS-0720081： Job has not specified mapper class|Job没有指定mapper类。|
|ODPS-0720091：Column duplicate|Job输入指定了重复的列。|
|ODPS-0720101： Input table should not be a view|输入表是视图。|
|ODPS-0720111：Output table should not be a view|输出表是视图。|
|ODPS-0720121：Invalid table partSpec|指定表的partSpec非法。|
|ODPS-0720131：Invalid multiple output|非法输出（包括duplicate label、same output tables or partitions）。|
|ODPS-0720141： Memory value out of bound|指定了不在\[256MB, 12288MB\]范围内的内存值。|
|ODPS-0720151： Cpu value out of bound|指定了不在\[50, 800\]范围内的CPU值。|
|ODPS-0720161： Invalid max attempts value|指定了不在（0，6）范围内的odps.mapred.map/reduce.max.attempts。|
|ODPS-0720171 ： Invalid IO sort buffer|指定的io sort buffer不在（64MB，odps.mapred.map/reduce.memory）范围内。|
|ODPS-0720181：Classpath resource between comma must not be empty|ClassPath的逗号之间为空。|
|ODPS-0720191：Invalid input split mode|指定了非法的切片模式。|
|ODPS-0720201：Invalid map split size|指定的odps.mapred.map.min/max.split.size不合法。|
|ODPS-0720211：Invalid number of map tasks|指定的odps.mapred.map.tasks不在（1,100000）内。|
|ODPS-0720221：Invalid max splits number|指定了非法的最大样本切片数。|
|ODPS-0720231：Job input not set|Job没有指定输入。|
|ODPS-0720241：Num of map instance is too big|Map实例数大于odps.mapred.max.map.tasks。|
|ODPS-0720251： Num of reduce instance is invalid|Reduce实例数不在\[0,odps.mapred.reduce.tasks\]范围。|
|ODPS-0720261：Invalid partition value|指定了非法的分区值。|
|ODPS-0720271： Allow no input is conflict to split mode|设置setallowNoInput\(true\)的同时，设置了不同于ALLOW\_NO\_INPUT的分隔模式。|
|ODPS-0720281: Invalid partitition format|分区格式非法。|
|ODPS-0720291:Invalid description json|输入说明的JSON格式非法。|
|ODPS-0720301:Too many job input|指定的输入数超过odps.mapred.map.max.input.num（默认为1024）。|
|ODPS-0720311:Invalid output label|指定的输出标签格式非法。|
|ODPS-0720321: Too many job output|指定的输出数超过odps.mapred.max.output.num（默认为256）。|
|ODPS-0720331: Too many cache resources|指定了超过odps.mapred.job.max.cache.resource.num参数设置的资源数。|
|ODPS-0730005:MR job internal error|MapReduce任务中发生不受用户控制的内部错误。|

