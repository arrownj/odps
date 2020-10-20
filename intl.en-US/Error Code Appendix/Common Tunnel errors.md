---
keyword: common Tunnel errors
---

# Common Tunnel errors

This topic describes the format of Tunnel error messages and provides details of these error messages.

## Format

Format of a common Tunnel error message:

```
requestId=request_id, ErrorCode=InvalidArgument, ErrorMessage=Invalid argument
```

This message contains the following parameters:

-   requestId: the unique ID of the request.
-   ErrorCode: the error type.
-   ErrorMessage: the detailed error information.

## Description

|Error type|Error message|Remarks|
|:---------|:------------|:------|
|AccessDenied|Access Denied|The error message returned because you are not authorized to perform this operation.|
|CorruptedDataStream|The data stream was corrupted, please try again later|N/A|
|DataUnderReplication|The specified table data is under replication and you cannot initiate upload or download at this time. Please try again later|The error message returned because the data is being copied across clusters and the operation cannot be performed.|
|DataVersionConflict|The specified table has been modified since the upload or download initiated and table data is being replicated at this time. Please initiate another download or upload later|The error message returned because the data is being copied across clusters and the operation cannot be performed. Try again later.|
|DataStoreError|DataStoreError|The error message returned because a storage error occurs. We recommend that you contact the administrator.|
|FlowExceeded|Your flow quota is exceeded|The error message returned because the data upload or download traffic exceeds the upper limit. We recommend that you check and control concurrent traffic. If you want to increase concurrent traffic, contact the project owner or administrator to evaluate the traffic pressure.|
|InConsistentBlockList|The specified block list is not consistent with the uploaded block list on server side|N/A|
|IncompleteBody|You did not provide the number of bytes specified by the Content-Length HTTP header|N/A|
|InternalServerError|Service internal error, please try again later|The error message returned because an internal server error occurs. We recommend that you try again later or contact the administrator.|
|InvalidArgument|Invalid argument|The error message returned because the parameter is invalid.|
|InvalidBlockID|The specified block id is not valid|The error message returned because the block ID is invalid.|
|InvalidColumnSpec|The specified columns is not valid|The error message returned because the specified column name is invalid. Typically, this error occurs because the name of the column from which you want to download data is incorrect.|
|InvalidRowRange|The specified row range is not valid|The error message returned because the specified number of rows is invalid. Typically, the value is zero or exceeds the maximum number of rows. We recommend that you check related parameters.|
|InvalidStatusChange|You cannot change the specified upload or download status|N/A|
|InvalidResourceSpec|InvalidResourceSpec|The error message returned because the project, table, or partition information is not the same as that contained in the session. We recommend that you check related information and try again later.|
|InvalidURI|Couldn’t parse the specified URI|N/A|
|InvalidUriSpec|The specified uri spec is not valid|N/A|
|InvalidProjectTable|InvalidProjectTable|The error message returned because the project name or table name is invalid. We recommend that you check the project name and table name.|
|InvalidPartitionSpec|InvalidPartitionSpec|The error message returned because the partition information is invalid. We recommend that you check the partition information again.|
|MalformedDataStream|The data stream you provided was not well-formed or did not validate against schema|The error message returned because the data format is incorrect. Typically, this error is caused by network disconnections.|
|MalformedHeaderValue|An HTTP header value was malformed|N/A|
|MalformedXML|The XML you provided was not well-formed or did not validate against schema|N/A|
|MaxMessageLengthExceeded|Your request was too big|N/A|
|MethodNotAllowed|The specified method is not allowed against this resource|The error message returned because the specified method is not allowed. Typically, this error occurs when you attempt to export views. This happens because views cannot be exported.|
|MissingContentLength|You must provide the Content-Length HTTP header|N/A|
|MissingPartitionSpec|You need to specify a partitionspec along with the specified table|The error message returned because the partition information is missing. If you perform operations on partitioned tables, the partition information must be specified. We recommend that you specify the partition information.|
|MissingRequestBodyError|The request body is missing|N/A|
|MissingRequiredHeaderError|Your request was missing a required header|N/A|
|NoPermission|You do not have enough privilege to complete the specified operation|N/A|
|NoSuchData|The uploaded data within this uploaded no longer exists|N/A|
|NoSuchPartition|The specified partition does not exist|The error message returned because the specified partition does not exist. Tunnel commands cannot be used to create partitions. You must create partitions before you upload or download data.|
|NoSuchUpload|The specified upload id does not exist|The error message returned because the specified upload ID does not exist.|
|NoSuchDownload|The specified download id does not exist|The error message returned because the specified download ID does not exist.|
|NoSuchProject|The specified project name does not exist|The error message returned because the specified project name does not exist. We recommend that you check the project name again.|
|NoSuchTable|The specified table name does not exist|The error message returned because the specified table name does not exist. We recommend that you check the table name again.|
|NoPermission|NoPermission|The error message returned because you are not authorized to perform this operation or you have configured an IP address whitelist. We recommend that you check whether the permissions are correct.|
|NotImplemented|A header you provided implies functionality that is not implemented|N/A|
|ObjectModified|The specified object has been modified since the specified timestamp|N/A|
|RequestTimeOut|Your socket connection to the server was not read from or written to within the timeout period|N/A|
|ServiceUnavailable|Service is temporarily unavailable, Please try again later|N/A|
|StatusConflict|You cannot complete the specified operation under the current upload or download status|The error message returned because the session has expired or has been committed to the creation of an upload task. We recommend that you create another session to upload data.|
|TableModified|The specified table has been modified since the download initiated. Try initiate another download|The error message returned because the table data is changed by other tasks during an upload or download. We recommend that you create another session and try again later.|
|Unauthorized|The request authorization header is invalid or missing|The error message returned because the AccessKey ID or AccessKey secret is incorrect, or the local device has a 15-minute time gap with the server.|
|UnexpectedContent|This request does not support content|N/A|

