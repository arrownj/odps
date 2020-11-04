---
keyword: SQL优化
---

# SQL调优

本文为您介绍常见的SQL问题以及优化示例。

## 数据倾斜优化

数据倾斜产生的根本原因是少数Worker处理的数据量远远超过其他Worker处理的数据量，因此少数Worker的运行时长远远超过其他Worker的平均运行时长，导致整个任务运行时间超长，造成任务延迟。

-   **Join操作导致的数据倾斜**

    Join操作导致数据倾斜的原因是Join on的Key分布不均匀。假设大表A和小表B执行Join操作，运行如下语句。

    ```
    SELECT * FROM A JOIN B ON A.value = B.value;
    ```

    复制该语句的Logview链接并打开，双击执行Join操作的作业，可以看到此时在\[Long-tails\]区域存在长尾现象，表示数据已经倾斜了。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3004359951/p37045.png)

    您可通过如下方法进行优化：

    -   由于表B是个小表并且没有超过512 MB，您可将上述语句优化为MapJoin语句后执行，语句如下。

        ```
        SELECT /* + MAPJOIN(B) */ * FROM A JOIN B ON A.value = B.value;
        ```

    -   将倾斜的Key用单独的逻辑来处理。假设两边的Key中有大量NULL数据导致了倾斜，则需要在Join前先过滤掉NULL数据或者补上随机数，然后再进行Join，示例如下。

        ```
        SELECT * FROM A JOIN B ON CASE WHEN A.value IS NULL THEN CONCAT('value',RAND() ) ELSE A.value END = B.value;
        ```

        在实际场景中，如果您发现已经数据倾斜，但无法获取导致数据倾斜的Key信息，可以使用如下方法查看数据倾斜。

        ```
        --执行如下语句产生数据倾斜。
        SELECT * FROM a JOIN b ON a.key=b.key;  
        --您可以执行如下SQL，查看Key的分布，判断执行Join操作时是否会有数据倾斜。
        SELECT left.key, left.cnt * right.cnt FROM 
        (select key, count(*) AS cnt FROM a GROUP BY key) LEFT 
        JOIN
        (SELECT key, COUNT(*) AS cnt FROM b GROUP BY key) RIGHT
        ON left.key=right.key;
        ```

-   **Group By倾斜**

    造成Group By倾斜的原因是Group By的Key分布不均匀。

    假设表A内有两个字段（Key, Value），表内的数据量较大且Key值分布不均匀，运行语句如下所示。

    ```
    SELECT key, COUNT(value) FROM A GROUP BY key;
    ```

    当表中的数据足够大时，您会在Logview中发现长尾现象。解决此问题，您需要在执行SQL前设置防倾斜的参数，设置语句为`set odps.sql.groupby.skewindata=true`。

-   **错误使用动态分区造成的数据倾斜**

    动态分区SQL时，在MaxCompute中会默认增加一个Reduce，用来将相同分区的数据合并在一起。此操作可以：

    -   减少MaxCompute系统产生的小文件，使后续处理更快速。
    -   避免一个Worker输出文件很多时占用内存过大。
    如果引入的Reduce导致分区数据倾斜，则会发生长尾。因为相同的数据最多只会有10个Worker处理，所以数据量大，则会发生长尾，示例如下。

    ```
    INSERT OVERWRITE TABLE A2 PARTITION(dt) SELECT SPLIT_PART(value,'\t',1) AS field1, SPLIT_PART(value,'\t',2) AS field2, dt FROM A WHERE dt='20151010';
    ```

    这种情况下，不建议使用动态分区，优化语句如下。

    ```
    INSERT OVERWRITE TABLE A2 PARTITION (dt='20151010') 
    SELECT
    SPLIT_PART(value,'\t',1) as field1,
    SPLIT_PART(value,'\t',2) as field2
    FROM A 
    WHERE dt='20151010';
    ```


数据倾斜优化的详情请参见[其它计算长尾调优](/cn.zh-CN/最佳实践/计算优化/其它计算长尾调优.md)。

## 窗口函数优化

如果SQL语句中使用了窗口函数，通常每个窗口函数会形成一个Reduce作业。如果窗口函数较多，会消耗过多的资源。您可以对符合下述条件的窗口函数进行优化：

-   窗口函数在OVER关键字后面要完全相同，要有相同的分组和排序条件。
-   多个窗口函数在同一层SQL中执行。

符合上述2个条件的窗口函数会合并为一个Reduce执行。SQL示例如下所示。

```
SELECT
RANK()OVER(PARTITION BY A ORDER BY B desc) AS RANK,
ROW_NUMBER()OVER(PARTITION BY A ORDER BY B desc) AS row_num
FROM MyTable;
```

## 子查询优化

子查询如下所示。

```
SELECT * FROM table_a a WHERE a.col1 IN (SELECT col1 FROM table_b b WHERE xxx);
```

当此语句中的table\_b子查询返回的col1的个数超过1000个时，系统会报错为`records returned from subquery exceeded limit of 1000`。此时您可以使用Join语句来代替，如下所示。

```
SELECT a.* FROM table_a a JOIN (SELECT DISTINCT col1 FROM table_b b WHERE xxx) c ON (a.col1 = c.col1)
```

**说明：**

-   如果没有使用DISTINCT关键字，而子查询表c返回的结果中有相同的col1的值，可能会导致a表的结果数变多。
-   DISTINCT关键字会导致查询在同一个Worker中执行。如果子查询数据量较大，会导致查询比较慢。
-   如果业务上已经确保子查询中col1列值无重复，您可以删除DISTINCT关键字，以提高性能。

## Join语句优化

当两个表进行Join操作时，建议在如下位置使用WHERE子句：

-   主表的分区限制条件可以写在WHERE子句中（最好先用子查询过滤）。
-   主表的WHERE子句建议写在SQL语句最后。
-   从表分区限制条件不要写在WHERE子句中，建议写在ON条件或者子查询中。

示例如下。

```
SELECT * FROM A JOIN (SELECT * FROM B WHERE dt=20150301)B ON B.id=A.id WHERE A.dt=20150301；
SELECT * FROM A JOIN B ON B.id=A.id WHERE B.dt=20150301；--不建议使用。此语句会先执行Join操作后进行分区裁剪，导致数据量变大，性能下降。
SELECT * FROM (SELECT * FROM A WHERE dt=20150301)A JOIN (SELECT * FROM B WHERE dt=20150301)B ON B.id=A.id；
```

