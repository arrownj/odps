---
keyword: optimize the calculation for long-period metrics
---

# Optimize the calculation for long-period metrics

This topic describes how to optimize the calculation for long-period metrics.

## Background information

When e-commerce companies build data warehouses or analyze their business, they often need to calculate metrics such as the numbers of visitors, buyers, and regular buyers in a period of time. These metrics are calculated based on the data that is accumulated over the period of time.

In general, these metrics are calculated based on the data in log tables. For example, you can execute the following statement to calculate the number of visitors for each item in the last 30 days:

```
select item_id -- The field that indicates the item ID. 
  ,count(distinct visitor_id) as ipv_uv_1d_001 
from vistor_item_detail_log 
where ds <= ${bdp.system.bizdate} 
and ds >=to_char(dateadd(to_date(${bdp.system.bizdate},'yyyymmdd'),-29,'dd'),'yyyymmdd') 
 group by item_id;
```

**Note:** All the variables in the code samples in this topic are scheduling variables in DataWorks. Therefore, the code samples in this topic are applicable only to scheduling nodes in DataWorks.

If a large amount of log data is generated every day, the preceding SELECT statement requires a large number of map tasks. If more than 99,999 map tasks are required, the map tasks fail.

## Objective

Calculate long-period metrics with minimal impact on the query performance.

The amount of the data accumulated over a long period of time is huge. If the system calculates metrics based on the data, the query performance is deteriorated. We recommend that you create an intermediate table that is used to summarize the data generated every day. This can remove duplicate data records and reduce the amount of data to be queried.

## Solution

1.  Create an intermediate table to summarize the data generated every day.

    In this example, you can create an intermediate table based on the data in the item\_id and visitor\_id fields. The following code provides an example:

    ```
    insert overwrite table mds_itm_vsr_xx(ds='${bdp.system.bizdate} ')
    select item_id,visitor_id,count(1) as pv
      from
      (
      select item_id,visitor_id
      from vistor_item_detail_log 
      where ds =${bdp.system.bizdate} 
      group by item_id,visitor_id
      ) a;
    ```

2.  Summarize the data that is accumulated over a long period of time from the intermediate table.

    The following code calculates the number of visitors for each item in the last 30 days:

    ```
    select item_id
            ,count(distinct visitor_id) as uv
            ,sum(pv) as pv
      from mds_itm_vsr_xx
      where ds <= '${bdp.system.bizdate} '
      and ds >= to_char(dateadd(to_date('${bdp.system.bizdate} ','yyyymmdd'),-29,'dd'),'yyyymmdd')
      group by item_id;
    ```


## Impact and consideration

In the preceding solution, the log data is deduplicated on a daily basis. This can reduce the amount of data for calculation and improve the query performance. However, the system needs to read data from multiple partitions for every calculation on the data that is accumulated over a long period of time.

To resolve this issue, you can merge data from multiple partitions into one partition, which contains all historical data. This way, you can accumulate data in an incremental manner and calculate long-period metrics based on data in one partition.

## Scenarios

Calculate the number of the regular buyers in the last day. A regular buyer is a buyer who made a purchase in a specific period of time, for example, in the last 30 days.

The following code calculates the number of the regular buyers in a period of time:

```
select item_id -- The field that indicates the item ID. 
        ,buyer_id as old_buyer_id
from buyer_item_detail_log 
where ds < ${bdp.system.bizdate} 
and ds >=to_char(dateadd(to_date(${bdp.system.bizdate},'yyyymmdd'),-29,'dd'),'yyyymmdd') 
group by item_id
        ,buyer_id;
```

Improvement:

-   Create and maintain a dimension table. This table is used to record the relationship between buyers and purchased items, such as the first purchase time, the last purchase time, the total number of purchased items, and the total amount of the purchases.
-   Update the data in the dimension table every day with the data in the billing logs of the last day.
-   To determine whether a buyer is a regular buyer, check whether the last purchase time of the buyer is within the last 30 days. This deduplicates data mappings and reduces the amount of data for calculation.

