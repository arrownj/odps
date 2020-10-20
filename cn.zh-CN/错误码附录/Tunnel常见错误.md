---
keyword: Tunnel常见错误
---

# Tunnel常见错误

本文为您介绍Tunnel异常信息格式以及常见问题。

## 错误信息格式

通常，Tunnel的错误信息如下。

```
requestId=request_id, ErrorCode=InvalidArgument, ErrorMessage=Invalid argument
```

涉及的信息如下：

-   requestId是标志用户请求的唯一标志。
-   ErrorCode是表明错误类型。
-   ErrorMessage是详细异常信息。

## 异常信息说明

|异常类型|异常信息|补充说明|
|:---|:---|:---|
|AccessDenied|Access Denied|权限不足。|
|CorruptedDataStream|The data stream was corrupted, please try again later|无。|
|DataUnderReplication|The specified table data is under replication and you cannot initiate upload or download at this time. Please try again later|数据处于跨级群复制状态，无法操作。|
|DataVersionConflict|The specified table has been modified since the upload or download initiated and table data is being replicated at this time. Please initiate another download or upload later|当前集群上的数据处于复制状态，无法进行数据操作，请稍后重试。|
|DataStoreError|DataStoreError|存储错误。建议联系管理员。|
|FlowExceeded|Your flow quota is exceeded|数据上传或下载超过流量限制。建议检查控制并发量。如果确实需要增加并发，请联系项目空间所有者或管理员评估流量压力。|
|InConsistentBlockList|The specified block list is not consistent with the uploaded block list on server side|无。|
|IncompleteBody|You did not provide the number of bytes specified by the Content-Length HTTP header|无。|
|InternalServerError|Service internal error, please try again later|服务内部错误，建议重试或联系管理员。|
|InvalidArgument|Invalid argument|参数不合法。|
|InvalidBlockID|The specified block id is not valid|非法BlockID。|
|InvalidColumnSpec|The specified columns is not valid|非法列名， 通常是下载指定列时列名错误。|
|InvalidRowRange|The specified row range is not valid|指定行数不合法，通常是超出了数据最大行数或者为0，建议检查相关参数。|
|InvalidStatusChange|You cannot change the specified upload or download status|无。|
|InvalidResourceSpec|InvalidResourceSpec|通常为项目、表和分区信息与Session不一致。建议检查相关信息并重试。|
|InvalidURI|Couldn’t parse the specified URI|无。|
|InvalidUriSpec|The specified uri spec is not valid|无。|
|InvalidProjectTable|InvalidProjectTable|项目或表名称不合法。建议检查相关名称。|
|InvalidPartitionSpec|InvalidPartitionSpec|分区信息不合法。请检查分区信息。|
|MalformedDataStream|The data stream you provided was not well-formed or did not validate against schema|数据格式错误，通常是网络断开导致。|
|MalformedHeaderValue|An HTTP header value was malformed|无。|
|MalformedXML|The XML you provided was not well-formed or did not validate against schema|无。|
|MaxMessageLengthExceeded|Your request was too big|无。|
|MethodNotAllowed|The specified method is not allowed against this resource|方法不支持。通常是尝试导出视图时发生此错误，目前不支持视图导出。|
|MissingContentLength|You must provide the Content-Length HTTP header|无。|
|MissingPartitionSpec|You need to specify a partitionspec along with the specified table|缺失分区信息。分区表操作必须携带分区信息，建议补充分区信息。|
|MissingRequestBodyError|The request body is missing|无。|
|MissingRequiredHeaderError|Your request was missing a required header|无。|
|NoPermission|You do not have enough privilege to complete the specified operation|无。|
|NoSuchData|The uploaded data within this uploaded no longer exists|无。|
|NoSuchPartition|The specified partition does not exist|指定的分区不存在。Tunnel命令不会创建分区，需要您自行创建分区后再上传下载。|
|NoSuchUpload|The specified upload id does not exist|指定的Upload ID不存在。|
|NoSuchDownload|The specified download id does not exist|指定的Download ID不存在。|
|NoSuchProject|The specified project name does not exist|项目不存在，建议检查相关名称。|
|NoSuchTable|The specified table name does not exist|表不存在，建议检查相关名称。|
|NoPermission|NoPermission|权限不足通常原因是无相关权限或者设置了IP白名单。建议检查权限是否正确。|
|NotImplemented|A header you provided implies functionality that is not implemented|无。|
|ObjectModified|The specified object has been modified since the specified timestamp|无。|
|RequestTimeOut|Your socket connection to the server was not read from or written to within the timeout period|无。|
|ServiceUnavailable|Service is temporarily unavailable, Please try again later|无。|
|StatusConflict|You cannot complete the specified operation under the current upload or download status|Session过期或者已经提交，重新创建Session上传。|
|TableModified|The specified table has been modified since the download initiated. Try initiate another download|表数据在上传下载期间被其他任务改动，建议重新创建Session重试。|
|Unauthorized|The request authorization header is invalid or missing|通常是AccessID、AccessKey有误，或本地机器时间与服务端时间差距15分钟以上。|
|UnexpectedContent|This request does not support content|无。|

