---
keyword: [view query jobs, cancel a query job, stv\_recents]
---

# View or cancel query jobs

MaxCompute Lightning provides the system view stv\_recents. You can use stv\_recents to view all the query jobs that are running.

## View all running query jobs

You can run a command to query the information of all running query jobs. The information includes the job ID, username, SQL statement, start time, duration, and resource waiting status. t indicates that the job is waiting for resources and is not executed. f indicates that the resources have been obtained and the job is being executed.

Run the following command:

```
select * from stv_recents;
```

The following figure shows an example of the query result.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/9092659951/p11169.jpg)

## Cancel a running query job

You can run the following command to cancel a running query job:

```
select cancel('query_id');
```

Replace query\_id in the command with the value of query\_id that is obtained by using stv\_recents.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/9092659951/p11170.jpg)

