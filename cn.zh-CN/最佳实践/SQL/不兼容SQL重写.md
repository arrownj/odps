---
keyword: 不兼容SQL
---

# 不兼容SQL重写

本文为您介绍如何修改不兼容SQL。

## 背景信息

MaxCompute 2.0完全拥抱开源生态，支持更多的语言功能，拥有更快的运行速度。但是MaxCompute 2.0会执行更严格的语法检测，一些在旧版本编译器下正常执行的不严谨的语法在MaxCompute 2.0下执行会报错。

为了使MaxCompute 2.0灰度升级更加平滑，MaxCompute框架支持回退机制，如果MaxCompute 2.0作业失败，会回退到MaxCompute 1.0执行。回退本身会增加作业时延。建议您在提交作业之前，手动关闭回退开关`set odps.sql.planner.mode=lot;`，以避免MaxCompute框架回退策略修改造成的影响。

根据线上回退情况，MaxCompute会通过邮件、钉钉通知问题作业的责任人尽快完成SQL作业的修改，否则会导致作业失败。

## group.by.with.star

说明：即`select * …group by…`语句。

-   MaxCompute 2.0版本中，要求Group By列表是源表中所有的列，否则执行报错。
-   旧版MaxCompute中，即使Group By列表不覆盖源表中所有的列，也支持`select * from group by key`语法。

示例

-   场景1：Group By Key中不包含所有列。
    -   错误写法

        ```
        select * from t group by key;
        ```

    -   报错信息

        ```
        FAILED: ODPS-0130071:[1,8] Semantic analysis exception - column reference t.value should appear in GROUP BY key
        ```

    -   正确写法

        ```
        select distinct key from t;
        ```

-   场景2：group by key包含所有列。
    -   如下写法不推荐。

```
select * from t group by key, value; -- t has columns key and value
```

    -   虽然MaxCompute2.0不会报错，但推荐改为如下。

```
select distinct key, value from t;
```


## bad.escape

说明：错误的escape序列问题。

按照MaxCompute规定，在String literal中应该用反斜线加三位8进制数字表示从0到127的ASCII字符。例如：使用“\\001”、“\\002”表示0、1。但\\01、\\0001也被当作\\001处理了。

这种方式会给新用户带来困扰，比如需要用“\\0001”表示“\\000”+“1”，便没有办法实现。同时对于从其他系统迁移而来的用户而言，会导致正确性错误。

**说明：** `\000`后面再加数字，如`\0001 - \0009`或`\00001`的写法可能会返回错误。

MaxCompute 2.0会解决此问题，对脚本中错误的序列进行修改。

-   错误写法

    ```
    select split(key, "\01"), value like "\0001" from t;
    ```

-   报错信息

    ```
    FAILED: ODPS-0130161:[1,19] Parse exception - unexpected escape sequence: 01
    ODPS-0130161:[1,38] Parse exception - unexpected escape sequence: 0001
    ```

-   正确改法

    ```
    select split(key, "\001"), value like "\001" from t;
    ```


## column.repeated.in.creation

说明：如果创建表时列名重复，MaxCompute 2.0将会报错。

示例

-   错误写法

    ```
    create table t (a BIGINT, b BIGINT, a BIGINT);
    ```

-   报错信息

    ```
    FAILED: ODPS-0130071:[1,37] Semantic analysis exception - column repeated in creation: a
    ```

-   正确写法

    ```
    create table t (a BIGINT, b BIGINT);
    ```


## string.join.double

说明：写JOIN条件时，等号的左右两边分别是STRING和DOUBLE类型。

-   旧版MaxCompute会把两边都转成BIGINT类型，会导致严重的精度损失问题，例如：1.1=“1”在连接条件中会被认为是相等的。
-   MaxCompute 2.0会与Hive兼容转为DOUBLE类型。

示例

-   不推荐写法

    ```
    select * from t1 join t2 on t1.double_value = t2.string_value;
    ```

-   Warning信息

    ```
    WARNING:[1,48]  implicit conversion from STRING to DOUBLE, potential data loss, use CAST function to suppress
    ```

-   推荐改法

    ```
    select * from t1 join t2 on t.double_value = cast(t2.string_value as double);
    ```


## window.ref.prev.window.alias

说明：Window Function引用同级select List中的其他Window Function Alias的问题。

示例

-   如果rn在t1中不存在，错误写法如下。

    ```
    select row_number() over (partition by c1 order by c1) rn,
    row_number() over (partition by c1 order by rn) rn2
    from t1;
    ```

-   报错信息

    ```
    FAILED: ODPS-0130071:[2,45] Semantic analysis exception - column rn cannot be resolved
    ```

-   正确改法

    ```
    select row_number() over (partition by c1 order by rn) rn2
    from
    (select c1, row_number() over (partition by c1 order by c1) rn
    from t1
    ) tmp;
    ```


## select.invalid.token.after.star

说明：select列表里面允许用户使用星号（\*）代表选择某张表的全部列，但星号（\*）后面不允许加`alias`，即使星号（\*）展开之后只有一列也不允许，新一代编译器将会对类似语法进行报错。

示例

-   错误写法

    ```
    select * as alias from table_test;
    ```

-   报错信息

    ```
    FAILED: ODPS-0130161:[1,10] Parse exception - invalid token 'as'
    ```

-   正确改法

    ```
    select * from table_test;
    ```


## agg.having.ref.prev.agg.alias

说明：有Having子句的情况下，select List可以出现前面Aggregate Function Alias的问题。

示例

-   错误写法

    ```
    select count(c1) cnt,
    sum(c1) / cnt avg
    from t1
    group by c2
    having cnt > 1;
    ```

-   报错信息

    ```
    FAILED: ODPS-0130071:[2,11] Semantic analysis exception - column cnt cannot be resolved
    ODPS-0130071:[2,11] Semantic analysis exception - column reference cnt should appear in GROUP BY key
    ```

    其中s、cnt在源表t1中都不存在，但因为有Having子句，旧版MaxCompute并未报错，MaxCompute 2.0则会提示`column cannot be resolve`，并报错。

-   正确改法

    ```
    select cnt, s, s/cnt avg
    from
    (
    select count(c1) cnt,
    sum(c1) s
    from t1
    group by c2
    having count(c1) > 1
    ) tmp;
    ```


## order.by.no.limit

说明：MaxCompute默认`order by`后需要增加`limit`限制数量，因为`order by`是全量排序，没有`limit`时执行性能较低。

示例

-   错误写法

    ```
    select * from (select *
    from (select cast(login_user_cnt as int) as uv, '3' as shuzi
    from test_login_cnt where type = 'device' and type_name = 'mobile') v
    order by v.uv desc) v
    order by v.shuzi limit 20;
    ```

-   报错信息

    ```
    FAILED: ODPS-0130071:[4,1] Semantic analysis exception - ORDER BY must be used with a LIMIT clause
    ```


在子查询`order by v.uv desc`中增加`limit`。

另外，MaxCompute 1.0对于view的检查不够严格。比如在一个不需要检查`limit`的Project（odps.sql.validate.orderby.limit=false）中，创建了一个View。

```
create view table_view as select id from table_view order by id;
```

若访问此View：

```
select * from table_view;
```

MaxCompute 1.0不会报错，而MaxCompute 2.0会报如下错误信息：

```
FAILED: ODPS-0130071:[1,15] Semantic analysis exception - while resolving view xdj.xdj_view_limit - ORDER BY must be used with a LIMIT clause
```

## generated.column.name.multi.window

说明：使用自动生成的`alias`的问题。

旧版MaxCompute会为select语句中的每个表达式自动生成一个alias，这个alias会最后显示在Console上。但是，它并不承诺这个alias的生成规则，也不承诺这个alias的生成规则会保持不变，所以不建议用户使用自动生成的alias。

MaxCompute 2.0会对使用自动生成alias的情况给予警告，由于牵涉面较广，暂时无法直接给予禁止。

对于某些情况，MaxCompute的不同版本间生成的alias规则存在已知的变动，但因为已有一些线上作业依赖于此类alias，这些查询在 MaxCompute版本升级或者回滚时可能会失败，存在此问题的用户，请修改您的查询，对于感兴趣的列，显式地指定列的别名。

示例

-   不推荐写法

    ```
    select _c0 from (select count(*) from table_name) t;
    ```

-   建议改法：

    ```
    select c from (select count(*) c from table_name) t;
    ```


## non.boolean.filter

**使用了非BOOLEAN过滤条件的问题。**

MaxCompute不允许布尔类型与其他类型之间的隐式转换，但旧版MaxCompute会允许用户在某些情况下使用BIGINT作为过滤条件。MaxCompute 2.0将不再允许，如果您的脚本中存在这样的过滤条件，请及时修改。示例如下：

错误写法：

```
select id, count(*) from table_name group by id having id;
```

报错信息：

```
FAILED: ODPS-0130071:[1,50] Semantic analysis exception - expect a BOOLEAN expression
```

正确改法：

```
select id, count(*) from table_name group by id having id <> 0;
```

## post.select.ambiguous

**在order by、cluster by、distribute by、sort by等语句中，引用了名字冲突的列的问题。**

旧版MaxCompute中，系统会默认选取Select列表中的后一列作为操作对象，MaxCompute 2.0将会进行报错，请及时修改。示例如下：

错误写法：

```
select a, b as a from t order by a limit 10;
```

报错信息：

```
FAILED: ODPS-0130071:[1,34] Semantic analysis exception - a is ambiguous, can be both t.a or null.a
```

正确改法：

```
select a as c, b as a from t order by a limit 10;
```

本次推送修改会包括名字冲突但语义一样的情况，虽然不会出现歧义，但是考虑到这种情况容易导致错误，作为一个警告，希望用户进行修改。

## duplicated.partition.column

**在query中指定了同名的partition的问题。**

旧版MaxCompute在用户指定同名partition key时并未报错，而是后一个的值直接覆盖了前一个，容易产生混乱。MaxCompute2.0将会对此情况进行报错，示例如下：

错误写法一：

```
insert overwrite table partition (ds = '1', ds = '2'）select ... ;
```

实际上，在运行时`ds = ‘1’`被忽略。

正确改法：

```
insert overwrite table partition (ds = '2'）select ... ;
```

错误写法二：

```
create table t (a bigint, ds string) partitioned by (ds string);
```

正确改法：

```
create table t (a bigint) partitioned by (ds string);
```

## order.by.col.ambiguous

**select list中alias重复，之后的order by子句引用到重复的alias的问题。**

错误写法：

```
select id, id
from table_test 
order by id;
```

正确改法：

```
select id, id id2
from table_name 
order by id;
```

需要去掉重复的alias，order by子句再进行引用。

## in.subquery.without.result

**colx in subquery没有返回任何结果，则colx在源表中不存在的问题。**

错误写法：

```
select * from table_name
where not_exist_col in (select id from table_name limit 0);
```

报错信息：

```
FAILED: ODPS-0130071:[2,7] Semantic analysis exception - column not_exist_col cannot be resolved
```

## ctas.if.not.exists

**目标表语法错误问题。**

如果目标表已经存在，旧版MaxCompute不会做任何语法检查，MaxCompute 2.0则会做正常的语法检查，这种情况会出现很多错误信息，示例如下：

错误写法：

```
create table if not exists table_name
as
select * from not_exist_table;
```

报错信息：

```
FAILED: ODPS-0130131:[1,50] Table not found - table meta_dev.not_exist_table cannot be resolved
```

## worker.restart.instance.timeout

旧版MaxCompute UDF每输出一条记录，便会触发一次对分布式文件系统的写操作，同时会向Fuxi发送心跳，如果UDF 10分钟没有输出任何结果，会得到如下错误提示：

```
FAILED: ODPS-0123144: Fuxi job failed - WorkerRestart errCode:252,errMsg:kInstanceMonitorTimeout, usually caused by bad udf performance.
```

MaxCompute 2.0的Runtime框架支持向量化，一次会处理某一列的多行来提升执行效率。但向量化可能导致原来不会报错的语句（2条记录的输出时间间隔不超过10分钟），因为一次处理多行，没有及时向Fuxi发送心跳而导致超时。

遇到这个错误，建议首先检查UDF是否有性能问题，每条记录需要数秒的处理时间。如果无法优化UDF性能，可以尝试手动设置batch row大小来绕开（默认为1024）：

```
set odps.sql.executionengine.batch.rowcount=16;
```

## divide.nan.or.overflow

**旧版MaxCompute不会做除法常量折叠的问题。**

比如如下语句，旧版MaxCompute对应的物理执行计划如下：

```
explain
select if(false, 0/0, 1.0)
from table_name;
in task M1_Stg1:
    Data source: meta_dev.table_name
    TS: alias: table_name
      SEL: If(False, Divide(UDFToDouble(0), UDFToDouble(0)), 1.0)
        FS: output: None
```

由此可以看出，IF和Divide函数仍然被保留，运行时因为IF第一个参数为false，第二个参数Divide的表达式不需要求值，所以不会出现除零异常。

而MaxCompute 2.0支持除法常量折叠，所以会报错。如下所示：

错误写法：

```
select IF(FALSE, 0/0, 1.0)
from table_name;
```

报错信息：

```
FAILED: ODPS-0130071:[1,19] Semantic analysis exception - encounter runtime exception while evaluating function /, detailed message: DIVIDE func result NaN, two params are 0.000000 and 0.000000
```

除了上述的错误，还可能遇到overflow错误，比如：

错误写法：

```
select if(false, 1/0, 1.0)
from table_name;
```

报错信息：

```
FAILED: ODPS-0130071:[1,19] Semantic analysis exception - encounter runtime exception while evaluating function /, detailed message: DIVIDE func result overflow, two params are 1.000000 and 0.000000
```

正确改法：

建议去掉/0的用法，换成合法常量。

CASE WHEN常量折叠也有类似问题，比如：CASE WHEN TRUE THEN 0 ELSE 0/0，MaxCompute 2.0常量折叠时所有子表达式都会求值，导致除0错误。

CASE WHEN可能涉及更复杂的优化场景，比如：

```
select case when key = 0 then 0 else 1/key end
from (
select 0 as key from src
union all
select key from src) r;
```

优化器会将除法下推到子查询中，转换类似于：

```
M (
select case when 0 = 0 then 0 else 1/0 end c1 from src
UNION ALL
select case when key = 0 then 0 else 1/key end c1 from src) r;
```

报错信息：

```
FAILED: ODPS-0130071:[0,0] Semantic analysis exception - physical plan generation failed: java.lang.ArithmeticException: DIVIDE func result overflow, two params are 1.000000 and 0.000000
```

其中UNION ALL第一个子句常量折叠报错，建议将SQL中的 CASE WHEN挪到子查询中，并去掉无用的CASE WHEN和去掉/0用法：

```
select c1 end
from (
select 0 c1 end from src
union all
select case when key = 0 then 0 else 1/key end) r;
```

## small.table.exceeds.mem.limit

旧版MaxCompute支持Multi-way Join优化，多个Join如果有相同Join Key，会合并到一个Fuxi Task中执行，比如下面例子中的J4\_1\_2\_3\_Stg1：

```
explain
select t1.*
from t1 join t2 on t1.c1 = t2.c1
join t3 on t1.c1 = t3.c1;
```

旧版MaxCompute物理执行计划：

```
In Job job0:
root Tasks: M1_Stg1, M2_Stg1, M3_Stg1
J4_1_2_3_Stg1 depends on: M1_Stg1, M2_Stg1, M3_Stg1

In Task M1_Stg1:
    Data source: meta_dev.t1

In Task M2_Stg1:
    Data source: meta_dev.t2

In Task M3_Stg1:
    Data source: meta_dev.t3

In Task J4_1_2_3_Stg1:
    JOIN: t1 INNER JOIN unknown INNER JOIN unknown
        SEL: t1._col0, t1._col1, t1._col2
            FS: output: None
```

如果增加MapJoin hint，旧版MaxCompute物理执行计划不会改变。也就是说对于旧版MaxCompute优先应用Multi-way Join优化，并且可以忽略用户指定MapJoin hint。

```
explain
select /* +mapjoin(t1) */ t1.*
from t1 join t2 on t1.c1 = t2.c1
join t3 on t1.c1 = t3.c1;
```

旧版MaxCompute物理执行计划同上。

MaxCompute 2.0 Optimizer会优先使用用户指定的MapJoin hint，对于上述例子，如果t1比较大的话，会遇到类似错误：

```
FAILED: ODPS-0010000:System internal **error** - SQL Runtime Internal Error: Hash Join Cursor HashJoin_REL… small **table** exceeds, **memory** limit(MB) 640, fixed **memory** used …, variable **memory** used …
```

对于这种情况，如果MapJoin不是期望行为，建议去掉MapJoin hint。

## sigkill.oom

同small.table.exceeds.mem.limit，如果用户指定了MapJoin hint，并且用户本身所指定的小表比较大。在旧版MaxCompute下有可能被优化成Multi-way Join从而成功。但在MaxCompute 2.0下，用户可能通过设定`odps.sql.mapjoin.memory.max`来避免小表超限的错误，但每个MaxCompute worker有固定的内存限制，如果小表本身过大，则MaxCompute worker会由于内存超限而被杀掉，错误类似于：

```
Fuxi job failed - WorkerRestart errCode:9,errMsg:SigKill(OOM), usually caused by OOM(**out****of** memory).
```

这里建议您去掉MapJoin hint，使用Multi-way Join。

## wm\_concat.first.argument.const

[聚合函数](/cn.zh-CN/开发/SQL及函数/内建函数/聚合函数.md)中关于WM\_CONCAT的说明，一直要求WM\_CONCAT第一个参数为常量，旧版MaxCompute检查不严格，比如源表没有数据，就算WM\_CONCAT第一个参数为ColumnReference，也不会报错。

```
函数声明：
string wm_concat(string separator, string str)
参数说明：
separator：String类型常量，分隔符。其他类型或非常量将引发异常。
```

MaxCompute 2.0会在plan阶段便检查参数的合法性，假如WM\_CONCAT的第一个参数不是常量，会立即报错。示例如下：

错误写法：

```
select wm_concat(value, ',') FROM src group by value;
```

报错信息：

```
FAILED: ODPS-0130071:[0,0] Semantic analysis exception - physical plan generation failed: com.aliyun.odps.lot.cbo.validator.AggregateCallValidator$AggregateCallValidationException: Invalid argument** type **- The first argument of WM_CONCAT must be constant string.
```

## pt.implicit.convertion.failed

srcpt是一个分区表，并有两个分区：

```
create table srcpt(key STRING, value STRING) partitioned by (pt STRING);
alter table srcpt add partition (pt='pt1');
alter table srcpt add partition (pt='pt2');
```

对于以上SQL，String类型pt列，INT类型常量，都会转为DOUBLE进行比较。即使Project设置了`odps.sql.udf.strict.mode=true`，旧版MaxCompute不会报错，所有pt都会过滤掉，而MaxCompute 2.0会直接报错。示例如下：

错误写法：

```
select key from srcpt where pt in (1, 2);
```

报错信息：

```
FAILED: ODPS-0130071:[0,0] Semantic analysis exception - physical plan generation failed: java.lang.NumberFormatException: ODPS-0123091:Illegal type cast - In function cast, value 'pt1' cannot be casted from String to Double.
```

建议避免STRING分区列和INT类型常量比较，将INT类型常量改成STRING类型。

## having.use.select.alias

SQL规范定义Group by + Having子句是Select子句之前阶段，所以Having中不应该使用Select子句生成的Column alias。

示例

-   错误写法：

    ```
    select id id2 from table_name group by id having id2 > 0;
    ```

-   报错信息：

    ```
    FAILED: ODPS-0130071:[1,44] Semantic analysis exception - column id2 cannot be resolvedODPS-0130071:[1,44] Semantic analysis exception - column reference id2 should appear in GROUP BY key
    ```

    其中id2为Select子句中新生成的Column alias，不应该在Having子句中使用。


## dynamic.pt.to.static

说明：MaxCompute2.0动态分区某些情况会被优化器转换成静态分区处理。

示例

```
insert overwrite table srcpt partition(pt) select id, 'pt1' from table_name;
```

会被转化成

```
insert overwrite table srcpt partition(pt='pt1') select id from table_name;
```

如果用户指定的分区值不合法，比如错误地使用了’$\{bizdate\}’，MaxCompute 2.0语法检查阶段便会报错。详情请参见[分区](/cn.zh-CN/产品简介/基本概念/分区.md)。

错误写法：

```
insert overwrite table srcpt partition(pt) select id, '${bizdate}' from table_name limit 0;
```

报错信息：

```
FAILED: ODPS-0130071:[1,24] Semantic analysis exception - wrong columns count 2 in data source, requires 3 columns (includes dynamic partitions if any)
```

旧版MaxCompute因为LIMIT 0，SQL最终没有输出任何数据，动态分区不会创建，所以最终不报错。

## lot.not.in.subquery

说明：In subquery中NULL值的处理问题。

在标准SQL的IN 运算中，如果后面的值列表中出现NULL，则返回值不会出现false，只可能是NULL或者true。如1 in \(null, 1, 2, 3\) 为true，而1 in \(null, 2, 3\) 为NULL，null in \(null, 1, 2, 3\) 为NULL。同理not in操作在列表中有NULL的情况下，只会返回false或者NULL，不会出现true。

MaxCompute 2.0会用标准的行为进行处理，收到此提醒的用户请注意检查您的查询，IN操作中的子查询中是否会出现空值，出现空值时行为是否与您预期相符，如果不符合预期请做相应的修改。

示例

-   ```
select * from t where c not in (select accepted from c_list);
```

    若accepted中不会出现NULL值，则此问题可忽略。若出现空值，则`c not in (select accepted from c_list)`原先返回true，则新版本返回NULL。

-   正确写法

    ```
    select * from t where c not in (select accepted from c_list where accepted is not null)
    ```


