---
keyword: [UDF, UDAF, UDTF]
---

# Java UDF

MaxCompute的UDF包括UDF、UDAF和UDTF三种函数。本文为您介绍如何通过Java实现这三种函数。

## 数据类型使用说明

MaxCompute 2.0版本升级后，Java UDF支持的数据类型从原来的BIGINT、STRING、DOUBLE、BOOLEAN扩展了更多的新数据类型，同时还扩展支持了ARRAY、MAP、STRUCT以及Writable等复杂类型。

-   Java UDF使用新数据类型的方法，如下所示：
    -   UDTF通过`@Resolve`注解获取Signature，例如`@Resolve("smallint->varchar(10)")`。
    -   UDF通过反射分析`evaluate`获取Signature，此时MaxCompute内置类型与Java类型符合一一映射关系。
    -   UDAF通过`@Resolve`注解获取Signature，MaxCompute2.0支持在注解中使用新类型，例如`@Resolve("smallint->varchar(10)")`。
-   Java UDF使用复杂类型的方法，如下所示：
    -   UDTF通过`@Resolve`注解指定Signature，例如`@Resolve("array<string>,struct<a1:bigint,b1:string>,string->map<string,bigint>,struct<b1:bigint>")`。
    -   UDF通过`evaluate`方法的Signature映射UDF的输入输出类型，此时参考MaxCompute类型与Java类型的映射关系。其中：
        -   ARRAY对应`java.util.List`；
        -   MAP对应`java.util.Map`；
        -   STRUCT对应`com.aliyun.odps.data.Struct`。
    -   UDAF通过`@Resolve`注解来获取Signature。MaxCompute 2.0版本支持在注解中使用新类型，例如`@Resolve("smallint->varchar(10)")`。

        **说明：**

        -   您可以使用`type,*`实现任意个数的传参，例如`@resolve("string,*->array<string>")`，请注意此处ARRAY后需要加Subtype。
        -   `com.aliyun.odps.data.Struct`从看不出Field Name和Field Type，所以需要用`@Resolve`注解来辅助。即如果需要在UDF中使用STRUCT，要求在UDF Class上也标注上`@Resolve`注解，这个注解只会影响参数或返回值中包含`com.aliyun.odps.data.Struct`的重载。
        -   目前Class上只能提供一个`@Resolve`注解，因此一个UDF中带有STRUCT参数或返回值的重载只能有一个。

数据类型的对应关系如下。

-   MaxCompute数据类型与Java数据类型的对应关系，如下所示。

    |MaxCompute Type|Java Type|
    |:--------------|:--------|
    |TINYINT|java.lang.Byte|
    |SMALLINT|java.lang.Short|
    |INT|java.lang.Integer|
    |BIGINT|java.lang.Long|
    |FLOAT|java.lang.Float|
    |DOUBLE|java.lang.Double|
    |DECIMAL|java.math.BigDecimal|
    |BOOLEAN|java.lang.Boolean|
    |STRING|java.lang.String|
    |VARCHAR|com.aliyun.odps.data.Varchar|
    |BINARY|com.aliyun.odps.data.Binary|
    |DATETIME|java.util.Date|
    |TIMESTAMP|java.sql.Timestamp|
    |ARRAY|java.util.List|
    |MAP|java.util.Map|
    |STRUCT|com.aliyun.odps.data.Struct|

    **说明：**

    -   在UDF中输入或输出参数的类型请务必使用Java Type，否则会报错ODPS-0130071。
    -   Java中对应的数据类型以及返回值数据类型是对象，数据类型首字母需大写。
    -   SQL中的NULL值通过Java中的NULL引用表示，因此不允许使用Java Primitive Type，因为无法表示SQL中的NULL值。
    -   此处ARRAY类型对应的Java类型是List，而不是数组。
-   MaxCompute 2.0版本支持定义Java UDF时，使用Writable类型作为参数和返回值。下面为MaxCompute数据类型和Java Writable类型的映射关系。

    |MaxCompute Type|Java Writable Type|
    |---------------|------------------|
    |TINYINT|ByteWritable|
    |SMALLINT|ShortWritable|
    |INT|IntWritable|
    |BIGINT|LongWritable|
    |FLOAT|FloatWritable|
    |DOUBLE|DoubleWritable|
    |DECIMAL|BigDecimalWritable|
    |BOOLEAN|BooleanWritable|
    |STRING|Text|
    |VARCHAR|VarcharWritable|
    |BINARY|BytesWritable|
    |DATETIME|DatetimeWritable|
    |TIMESTAMP|TimestampWritable|
    |INTERVAL\_YEAR\_MONTH|IntervalYearMonthWritable|
    |INTERVAL\_DAY\_TIME|IntervalDayTimeWritable|
    |ARRAY|N/A|
    |MAP|N/A|
    |STRUCT|N/A|


## UDF

实现UDF需要继承`com.aliyun.odps.udf.UDF`类，并实现`evaluate`方法。`evaluate`方法必须是非静态的Public方法。`evaluate`方法的参数和返回值数据类型将作为SQL中UDF的函数签名。您可以在UDF中实现多个`evaluate`方法，在调用UDF时，框架会依据UDF调用的参数类型匹配正确的`evaluate`方法 。

**说明：**

-   不同的JAR包中最好不要有类名相同但实现功能逻辑不一样的类。例如`UDF(UDAF/UDTF)： udf1、udf2`分别对应资源`udf1.jar、``udf2.jar`。如果两个JAR包里都包含一个`com.aliyun.UserFunction.class`类，当同一个SQL中同时使用到这两个UDF时，系统会随机加载其中一个类，导致UDF执行行为不一致，甚至编译失败。
-   如果1个SQL中同时使用2个UDF，执行时2个UDF会共享一个Classpath，不会隔离。如果2个UDF引用的资源都包含同一个Class，则Classloader会引用哪个Class是不确定的。因此，请尽量避免在不同的资源中使用同一个Class。

可以通过实现`void setup(ExecutionContext ctx)`和`void close()`来分别实现UDF的初始化和结束代码。

UDF的使用方式与MaxCompute SQL中普通的内建函数相同，详情请参见[内建函数](/intl.zh-CN/开发/SQL及函数/内建函数/数学函数.md)。

UDF的示例如下。

```
package org.alidata.odps.udf.examples; 
  import com.aliyun.odps.udf.UDF; 

public final class Lower extends UDF { 
  public String evaluate(String s) { 
    if (s == null) { 
        return null; 
    } 
        return s.toLowerCase(); 
  } 
}
```

使用Writable类型实现Concat的示例如下。

```
package com.aliyun.odps.udf.example;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.udf.UDF;
public class MyConcat extends UDF {
  private Text ret = new Text();
  public Text evaluate(Text a, Text b) {
    if (a == null || b == null) {
      return null;
    }
    ret.clear();
    ret.append(a.getBytes(), 0, a.getLength());
    ret.append(b.getBytes(), 0, b.getLength());
    return ret;
  }
}
```

**说明：**

所有的Writable类型所在的Package为`com.aliyun.odps.io`。如果您要使用该类型，可在[API文档地址](https://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-commons/0.30.9-public)下载`odps-sdk-commons`包。

MaxCompute提供的SDK包整体信息，如下表所示。

|包名|描述|
|--|--|
|odps-sdk-core|MaxCompute的基础功能，例如对表、Project的操作，以及Tunnel均在此包中。|
|odps-sdk-commons|Util封装。|
|odps-sdk-udf|UDF功能的主体接口。|
|odps-sdk-mapred|MapReduce功能。|
|odps-sdk-graph|Graph Java SDK，搜索关键词odps-sdk-graph。|

## UDAF

实现Java UDAF类需要继承`com.aliyun.odps.udf.Aggregator`类，并实现以下几个方法。

```
import com.aliyun.odps.udf.ContextFunction;
import com.aliyun.odps.udf.ExecutionContext;
import com.aliyun.odps.udf.UDFException;
public abstract class Aggregator implements ContextFunction {
    @Override
    public void setup(ExecutionContext ctx) throws UDFException {
    }
    @Override
    public void close() throws UDFException {
    }
    /**
     * 创建聚合Buffer * @return Writable 聚合buffer。
     */
    abstract public Writable newBuffer();
    /**
     * @param buffer 聚合buffer * @param args SQL中调用UDAF时指定的参数，不能为null，但是args里面的元素可以为null，代表对应的输入数据是null * @throws UDFException。
     */
    abstract public void iterate(Writable buffer, Writable[] args) throws UDFException;

    /**
     * 生成最终结果 * @param buffer * @return Object UDAF的最终结果 * @throws UDFException。
     */
    abstract public Writable terminate(Writable buffer) throws UDFException;
    abstract public void merge(Writable buffer, Writable partial) throws UDFException;
}
```

其中最重要的是`iterate`、`merge`和`terminate`三个方法，UDAF的主要逻辑依赖于这三个方法的实现。此外，还需要您实现自定义的Writable Buffer。

以实现求平均值`avg`为例，下图简要说明了在MaxCompute UDAF中这一函数的实现逻辑及计算流程。

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7882659951/p1855.jpg)

在上图中，输入数据被按照一定的大小进行分片（详情请参见[MapReduce](/intl.zh-CN/开发/MapReduce/概要/MapReduce概述.md)），每片的大小适合一个Worker在适当的时间内完成。分片大小的设置需要您手动配置完成。

UDAF的计算过程分为两个阶段：

-   第一阶段：每个Worker统计分片内数据的个数及汇总值。您可以将每个分片内的数据个数及汇总值视为一个中间结果。
-   第二阶段：Worker汇总上一个阶段中每个分片内的信息。在最终输出时，`r.sum/r.count`即是所有输入数据的平均值。

计算平均值的UDAF的代码示例，如下所示。

```
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import com.aliyun.odps.io.DoubleWritable;
import com.aliyun.odps.io.Writable;
import com.aliyun.odps.udf.Aggregator;
import com.aliyun.odps.udf.UDFException;
import com.aliyun.odps.udf.annotation.Resolve;
@Resolve("double->double")
public class AggrAvg extends Aggregator {
  private static class AvgBuffer implements Writable {
    private double sum = 0;
    private long count = 0;
    @Override
    public void write(DataOutput out) throws IOException {
      out.writeDouble(sum);
      out.writeLong(count);
    }
    @Override
    public void readFields(DataInput in) throws IOException {
      sum = in.readDouble();
      count = in.readLong();
    }
  }
  private DoubleWritable ret = new DoubleWritable();
  @Override
  public Writable newBuffer() {
    return new AvgBuffer();
  }
  @Override
  public void iterate(Writable buffer, Writable[] args) throws UDFException {
    DoubleWritable arg = (DoubleWritable) args[0];
    AvgBuffer buf = (AvgBuffer) buffer;
    if (arg != null) {
      buf.count += 1;
      buf.sum += arg.get();
    }
  }
  @Override
  public Writable terminate(Writable buffer) throws UDFException {
    AvgBuffer buf = (AvgBuffer) buffer;
    if (buf.count == 0) {
      ret.set(0);
    } else {
      ret.set(buf.sum / buf.count);
    }
    return ret;
  }
  @Override
  public void merge(Writable buffer, Writable partial) throws UDFException {
    AvgBuffer buf = (AvgBuffer) buffer;
    AvgBuffer p = (AvgBuffer) partial;
    buf.sum += p.sum;
    buf.count += p.count;
  }
}
```

使用Writable类型实现Concat，示例如下。

```
package com.aliyun.odps.udf.example;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.udf.UDF;
public class MyConcat extends UDF {
  private Text ret = new Text();
  public Text evaluate(Text a, Text b) {
    if (a == null || b == null) {
      return null;
    }
    ret.clear();
    ret.append(a.getBytes(), 0, a.getLength());
    ret.append(b.getBytes(), 0, b.getLength());
    return ret;
  }
}
```

-   `Writable[] writables`：表示一行数据。代码中是指传入的列，例如`writables[0]`表示第一列，`writables[1]`表示第二列。
-   `iterate(Writable writable,Writable[] writables)`方法：`writable`是指一个阶段性的汇总数据，在不同的Map任务中，`group by`后得出的数据（可理解为一个集合），每行执行一次。
-   `merge()`方法：将不同的Map直接结算的结果进行汇总。
-   `terminate()`方法：返回数据。
-   `newBuffer()`方法：创建初始返回结果的值。
-   Writable的`readFields`方法，由于partial的`writable`对象是重用的，同一个对象的`readFields`方法会被调用多次。该方法每次调用的时候重置整个对象，如果对象中包含`Collection`，则需要清空。
-   UDAF在SQL中的使用语法与普通的内建聚合函数相同，详情请参见[聚合函数](/intl.zh-CN/开发/SQL及函数/内建函数/聚合函数.md)。
-   运行UDTF的方法与UDF类似，详情请参见[运行 UDF](/intl.zh-CN/快速入门/开发Java UDF（可选）.md)。
-   STRING对应的Writable类型为Text。

## UDTF

Java UDTF需要继承`com.aliyun.odps.udf.UDTF`类。这个类需要实现4个方法，如下表所示。

|接口定义|描述|
|:---|:-|
|public void setup\(ExecutionContext ctx\) throws UDFException|初始化方法，在UDTF处理输入数据前，调用用户自定义的初始化行为。在每个Worker内`setup`会被先调用一次。|
|public void process\(Object\[\] args\) throws UDFException|这个方法由框架调用，SQL中每一条记录都会对应调用一次`process`，`process`的参数为SQL语句中指定的UDTF输入参数。输入参数以`Object[]`的形式传入，输出结果通过`forward`函数输出。您需要在`process`函数内自行调用`forward`，以决定输出数据。|
|public void close\(\) throws UDFException|UDTF的结束方法。此方法由框架调用，并且只会被调用一次，即在处理完最后一条记录之后被调用。|
|public void forward\(Object …o\) throws UDFException|调用`forward`方法输出数据，每调用一次`forward`代表输出一条记录。SQL语句UDTF的`as`子句指定的列需为该记录中的列。|

示例

1.  UDTF的程序示例，如下所示。

    ```
    package org.alidata.odps.udtf.examples;
    import com.aliyun.odps.udf.UDTF;
    import com.aliyun.odps.udf.UDTFCollector;
    import com.aliyun.odps.udf.annotation.Resolve;
    import com.aliyun.odps.udf.UDFException;
    // TODO define input and output types, e.g., "string,string->string,bigint".
       @Resolve("string,bigint->string,bigint")
       public class MyUDTF extends UDTF {
         @Override
         public void process(Object[] args) throws UDFException {
           String a = (String) args[0];
           Long b = (Long) args[1];
           for (String t: a.split("\\s+")) {
             forward(t, b);
           }
         }
       }
    ```

    **说明：** 以上只是程序示例，关于如何在MaxCompute中运行UDTF的方法与UDF类似，详情请参见[运行UDF](/intl.zh-CN/快速入门/开发Java UDF（可选）.md)。

2.  执行如下语句使用UDF。假设在MaxCompute上创建UDTF时，注册函数名为`user_udtf`。

    ```
    select user_udtf(col0, col1) as (c0, c1) from my_table;
    ```

    假设`my_table`的`col0`、`col1`的值如下所示。

    ```
    +------+------+
    | col0 | col1 |
    +------+------+
    | A B | 1 |
    | C D | 2 |
    +------+------+
    ```

    则`select`的结果，如下所示。

    ```
    +----+----+
    | c0 | c1 |
    +----+----+
    | A  | 1  |
    | B  | 1  |
    | C  | 2  |
    | D  | 2  |
    +----+----+
    ```


## UDTF读取MaxCompute资源示例

在UDTF中，您可以读取MaxCompute的[资源](/intl.zh-CN/产品简介/基本概念/资源.md)。利用UDTF读取MaxCompute资源的示例如下所示：

1.  编写UDTF程序，编译成功后导出JAR包（udtfexample1.jar）。

    ```
    package com.aliyun.odps.examples.udf;
    import java.io.BufferedReader;
    import java.io.IOException;
    import java.io.InputStream;
    import java.io.InputStreamReader;
    import java.util.Iterator;
    import com.aliyun.odps.udf.ExecutionContext;
    import com.aliyun.odps.udf.UDFException;
    import com.aliyun.odps.udf.UDTF;
    import com.aliyun.odps.udf.annotation.Resolve;
    /**
     * project: example_project 
     * table: wc_in2 
     * partitions: p2=1,p1=2 
     * columns: colc,colb
     */
    @Resolve("string,string->string,bigint,string")
    public class UDTFResource extends UDTF {
      ExecutionContext ctx;
      long fileResourceLineCount;
      long tableResource1RecordCount;
      long tableResource2RecordCount;
      @Override
      public void setup(ExecutionContext ctx) throws UDFException {
      this.ctx = ctx;
      try {
       InputStream in = ctx.readResourceFileAsStream("file_resource.txt");
       BufferedReader br = new BufferedReader(new InputStreamReader(in));
       String line;
       fileResourceLineCount = 0;
       while ((line = br.readLine()) != null) {
         fileResourceLineCount++;
       }
       br.close();
       Iterator<Object[]> iterator = ctx.readResourceTable("table_resource1").iterator();
       tableResource1RecordCount = 0;
       while (iterator.hasNext()) {
         tableResource1RecordCount++;
         iterator.next();
       }
       iterator = ctx.readResourceTable("table_resource2").iterator();
       tableResource2RecordCount = 0;
       while (iterator.hasNext()) {
         tableResource2RecordCount++;
         iterator.next();
       }
     } catch (IOException e) {
       throw new UDFException(e);
     }
    }
       @Override
       public void process(Object[] args) throws UDFException {
         String a = (String) args[0];
         long b = args[1] == null ? 0 : ((String) args[1]).length();
         forward(a, b, "fileResourceLineCount=" + fileResourceLineCount + "|tableResource1RecordCount="
         + tableResource1RecordCount + "|tableResource2RecordCount=" + tableResource2RecordCount);
        }
    }
    ```

2.  添加资源到MaxCompute。

    ```
    Add file file_resource.txt;
    Add jar udtfexample1.jar;
    Add table table_resource1 as table_resource1;
    Add table table_resource2 as table_resource2;
    ```

3.  在MaxCompute中创建UDTF函数（`my_udtf`）。

    ```
    create function mp_udtf as com.aliyun.odps.examples.udf.UDTFResource using 
    'udtfexample1.jar, file_resource.txt, table_resource1, table_resource2';
    ```

4.  运行该UDTF。

    ```
    select mp_udtf("10","20") as (a, b, fileResourceLineCount) from tmp1;  
    --返回结果如下。
    +-------+------------+-------+
    | a | b      | fileResourceLineCount |
    +-------+------------+-------+
    | 10    | 2          | fileResourceLineCount=3|tableResource1RecordCount=0|tableResource2RecordCount=0 |
    | 10    | 2          | fileResourceLineCount=3|tableResource1RecordCount=0|tableResource2RecordCount=0 |
    +-------+------------+-------+
    ```


## 复杂数据类型示例

以下示例中，定义了一个有三个Overloads的UDF，其中第一个用了ARRAY作为参数，第二个用了MAP作为参数，第三个用了STRUCT作为参数。由于第三个Overloads将STRUCT作为参数或者返回值的类型，因此要求必须要对UDF Class打上`@Resolve`注解，指定STRUCT的具体类型。

```
import com.aliyun.odps.udf.UDF;
import com.aliyun.odps.udf.annotation.Resolve;
@Resolve("struct,string->string")
public class UdfArray extends UDF {
    public String evaluate(List vals, Long len) {
        return vals.get(len.intValue());
    }

    public String evaluate(Map map, String key) {
        return map.get(key);
    }

    public String evaluate(Struct struct, String key) {
        return struct.getFieldValue("a") + key;
    }
}
```

您可以直接将复杂类型传入UDF中。

```
create function my_index as 'UdfArray' using 'myjar.jar'; 
select id, my_index(array('red', 'yellow', 'green'), colorOrdinal) as color_name from co
```

## Hive UDF兼容示例

MaxCompute 2.0支持Hive风格的UDF，部分Hive UDF、UDTF可以直接在MaxCompute上使用。

**说明：** 目前支持兼容的Hive版本为2.1.0，对应Hadoop版本为2.7.2。如果UDF是在其他版本的Hive/Hadoop开发的，则可能需要使用此Hive/Hadoop版本重新编译。

示例如下。

```
package com.aliyun.odps.compiler.hive;
import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
public class Collect extends GenericUDF {
  @Override
  public ObjectInspector initialize(ObjectInspector[] objectInspectors) throws UDFArgumentException {
    if (objectInspectors.length == 0) {
      throw new UDFArgumentException("Collect: input args should >= 1");
    }
    for (int i = 1; i < objectInspectors.length; i++) {
      if (objectInspectors[i] != objectInspectors[0]) {
        throw new UDFArgumentException("Collect: input oi should be the same for all args");
      }
    }
    return ObjectInspectorFactory.getStandardListObjectInspector(objectInspectors[0]);
  }
  @Override
  public Object evaluate(DeferredObject[] deferredObjects) throws HiveException {
    List<Object> objectList = new ArrayList<>(deferredObjects.length);
    for (DeferredObject deferredObject : deferredObjects) {
      objectList.add(deferredObject.get());
    }
    return objectList;
  }
  @Override
  public String getDisplayString(String[] strings) {
    return "Collect";
  }
}
```

**说明：**

-   该UDF可以支持所有的类型，包括ARRAY、MAP、STRUCT等复杂类型。
-   对应Hive UDF的使用请参考以下链接：
    -   [HivePlugins](https://cwiki.apache.org/confluence/display/Hive/HivePlugins)
    -   [DeveloperGuide UDTF](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide+UDTF)
    -   [GenericUDAFCaseStudy](https://cwiki.apache.org/confluence/display/Hive/GenericUDAFCaseStudy)

上述UDF可以将任意类型、数量的参数打包成ARRAY输出。假设输出JAR包名为`test.jar`。

```
--添加资源。
Add jar test.jar;
--创建function。
CREATE FUNCTION hive_collect as 'com.aliyun.odps.compiler.hive.Collect' using 'test.jar';
--使用function。
set odps.sql.hive.compatible=true;
select hive_collect(4y,5y,6y);
+------+
| _c0  |
+------+
| [4, 5, 6] |
+------+
```

使用兼容Hive的UDF需要注意以下几点：

-   创建UDF时需要指定JAR包，无法自动将所有JAR包加入Classpath。

    **说明：** MaxCompute的`add jar`命令会永久地在Project中创建一个资源。

-   执行SQL语句时，需要在SQL前加`set odps.sql.hive.compatible=true;`语句，`set`语句和SQL语句一起提交执行。
-   请注意MaxCompute的[Java沙箱](/intl.zh-CN/开发/MapReduce/Java SDK/Java沙箱.md)限制。

