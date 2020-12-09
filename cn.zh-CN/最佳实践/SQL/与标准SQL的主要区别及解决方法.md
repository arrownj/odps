---
keyword: [MaxCompute SQL, 标准SQL, 与标准SQL的区别]
---

# 与标准SQL的主要区别及解决方法

本文为您列举MaxCompute SQL与标准SQL的区别及常见问题解决方法。

## MaxCompute SQL与标准SQL的基本区别

|主要区别|问题现象|解决方法|
|----|----|----|
|应用场景|不支持事务（不支持Commit和Rollback，不推荐使用INSERT INTO）。|建议代码具备幂等性，支持重新执行。推荐您使用INSERT OVERWRITE写数据。|
|不支持索引和主键约束。|无。|
|部分字段不支持默认值或默认函数。|如果字段有默认值，您可以在数据写入时自行赋值。MaxCompute支持在创建表时，对BIGINT、DOUBLE、BOOLEAN和STRING类型的字段添加默认值。详情请参见[MaxCompute SQL新功能](/cn.zh-CN/新功能发布记录/公告.md)。|
|不支持自增字段。|无。|
|表分区|单表最多支持6万个分区。超过6万个分区会报错。|选择合适的分区列，减少分区数。|
|一次查询输入的分区不能超过1万个，否则会报错。如果是2级分区且查询时只根据2级分区进行过滤，总的分区数大于1万也可能导致报错。|解决方法请参见[在执行MaxCompute SQL过程中，报错输出表的分区过多，如何处理？](/cn.zh-CN/常见问题/SQL/SQL语句.md)。|
|精度|DOUBLE类型存在精度问题。|不建议直接使用等于号（=）关联两个DOUBLE字段。建议将两个数相减，如果差距小于一个预设的值，则认为两个数是相同的。例如`ABS(a1-a2)<0.000000001`。|
|虽然MaxCompute支持高精度类型DECIMAL，但是有更高精度的要求。|如果有更高的精度要求，您可以先把数据存储为STRING类型，然后使用UDF实现对应的计算。|
|数据类型转换|出现各种预期外的错误，代码维护问题。|如果有2个不同的字段类型需要执行JOIN操作，建议您先转换字段类型再执行JOIN操作。|
|日期类型和字符串的隐式转换。|如果在需要传入日期类型的函数中传入一个字符串，字符串和日期类型根据`yyyy-mm-dd hh:mi:ss`格式进行转换。|

## DDL与DML的区别及解决方法

|主要区别|问题现象|解决办法|
|----|----|----|
|表结构|不能修改分区列列名，只能修改分区列对应的值。|解决方案请参见[分区和分区列的区别是什么？](/cn.zh-CN/常见问题/SQL/SQL语句.md)。|
|支持增加列，但是不支持删除列及修改列的数据类型。|解决方案请参见[SQL常见问题](/cn.zh-CN/常见问题/SQL/SQL语句.md)。|
|INSERT|MaxCompute SQL需要在INSET INTO或INSERT OVERWRITE后加关键字TABLE。|无。|
|数据插入表的字段映射不是根据SELECT的别名执行，而是根据SELECT字段的顺序和表中字段的顺序执行映射。|无。|
|UPDATE和DELETE|不支持UPDATE和DELETE语句。|解决方案请参见[如何删除MaxCompute表或分区中的数据？](/cn.zh-CN/常见问题/SQL/SQL语句.md)和[如何更新MaxCompute表或分区中的数据？](/cn.zh-CN/常见问题/SQL/SQL语句.md)。|
|SELECT|MaxCompute SQL最多支持6张小表的MAPJOIN，并且连续JOIN的表不能超过16张。|解决方案请参见[在执行MaxCompute SQL过程中，报错输入表过多，如何处理？](/cn.zh-CN/常见问题/SQL/SQL语句.mdsection_may_waj_495)。|
|IN和NOT IN|IN、NOT IN、EXIST和NOT EXIST，后面的子查询返回的分区数据量不能超过1000条。|解决方案请参见[如何使用NOT IN？](/cn.zh-CN/常见问题/SQL/SQL语句.mdsection_03v_q8c_l4j)。如果业务上已经保证子查询返回结果的唯一性，可以考虑去掉DISTINCT，从而提升查询性能。|
|SQL返回10000条|MaxCompute限制了单独执行SELECT语句时返回的数据条数。|解决方案请参见[其他操作](http://kb.aliyun-inc.com/kb/27834)。|
|需要查询的结果数据条数很多。|解决方案请参见[使用SQLTask执行SQL查询时，如果查询结果条数大于限制的1000条，该如何获取所有数据？](/cn.zh-CN/常见问题/SQL/SQL语句.mdsection_3ti_u89_t81)。|
|MAPJOIN|JOIN不支持笛卡尔积。|JOIN必须要用ON关键字设置关联条件。如果有一些小表要作为广播表，需要使用MAPJOIN HINT。

解决方案请参见[如何解决JOIN报错？](/cn.zh-CN/常见问题/SQL/SQL语句.md)。 |
|ORDER BY|ORDER BY需要配合LIMIT N使用。|如果希望执行大数据量的排序任务，甚至是全表排序任务，可以增大N值。解决方案请参见[MaxCompute查询得到的数据是根据什么排序的？](/cn.zh-CN/常见问题/SQL/SQL语句.md)。|
|UNION ALL|参与UNION ALL运算的所有表必须列数一致，否则会报错。|参与UNION ALL运算的所有列的数据类型、列个数和列名称必须完全一致。|
|UNION ALL需要再嵌套一层子查询。|无。|

