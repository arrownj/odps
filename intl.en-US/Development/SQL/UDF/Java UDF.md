---
keyword: [UDF, UDAF, UDTF]
---

# Java UDF

MaxCompute supports user-defined functions \(UDFs\), user-defined aggregate functions \(UDAFs\), and user-defined table-valued functions \(UDTFs\). This topic describes how to implement the three types of functions by using Java.

## Data type

Java UDFs in MaxCompute V2.0 support more basic data types in addition to BIGINT, STRING, DOUBLE, and BOOLEAN. They also support complex data types such as ARRAY, MAP, and STRUCT, as well as Writable types.

-   To use a new basic data type, Java UDFs specify their function signatures in the following ways:
    -   A UDTF uses the `@Resolve` annotation, for example, `@Resolve("smallint->varchar(10)")`.
    -   A UDF uses the `evaluate` method. In this case, MaxCompute built-in function types and Java function types are one-to-one mapping.
    -   A UDAF uses the `@Resolve` annotation. MaxCompute V2.0 supports the use of new basic data types in annotations, for example, `@Resolve("smallint->varchar(10)")`.
-   To use a complex data type, Java UDFs specify their function signatures in the following ways:
    -   A UDTF uses the `@Resolve` annotation, for example, `@Resolve("array<string>,struct<a1:bigint,b1:string>,string->map<string,bigint>,struct<b1:bigint>")`.
    -   A UDF uses the signature of the `evaluate` method to match input and output types. For more information, see the mapping between MaxCompute and Java data types. The following examples show the mapping of data types supported by MaxCompute and Java:
        -   ARRAY in MaxCompute is equivalent to `java.util.List`.
        -   MAP in MaxCompute is equivalent to `java.util.Map`.
        -   STRUCT in MaxCompute is equivalent to `com.aliyun.odps.data.Struct`.
    -   A UDAF uses the `@Resolve` annotation. MaxCompute V2.0 supports the use of new data types in annotations, for example, `@Resolve("smallint->varchar(10)")`.

        **Note:**

        -   You can use `type,*` to pass any number of parameters, for example, `@resolve("string,*->array<string>")`. Note that you must add Subtype after ARRAY.
        -   The field name and field type of `com.aliyun.odps.data.Struct` cannot be reflected. Therefore, you must use `@Resolve` to obtain them. In other words, to use STRUCT in a UDF, you must add the `@Resolve annotation` to the UDF class. This annotation only affects the overloads of parameters or return values that contain `com.aliyun.odps.data.Struct`.
        -   Only one `@Resolve` annotation can be provided to the class. Therefore, only one overload with a STRUCT parameter or return value exists in a UDF.

Mapping between MaxCompute and Java data types

-   The following table describes the mapping between MaxCompute and Java data types.

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

    **Note:**

    -   Make sure that the input and output parameters in a UDF are of a Java type. Otherwise, error ODPS-0130071 is returned.
    -   Java data types and the data types of returned values are objects and must start with an uppercase letter.
    -   The NULL value in SQL statements is represented by a NULL reference in Java. Java Primitive Type cannot have NULL values and therefore must not be used.
    -   The ARRAY type in the preceding table maps the LIST data type in Java.
-   MaxCompute V2.0 allows you to use Writable types as parameters and return values when you define Java UDFs. The following table describes the mapping between MaxCompute data types and Java Writable types.

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

UDFs must inherit the class `com.aliyun.odps.udf.UDF` and use the `evaluate` method of the class. The `evaluate` method must be a non-static public method and its parameter and return value are used as the UDF signature in SQL statements. You can implement multiple `evaluate` methods in a UDF. To call a UDF, the framework matches the correct `evaluate` method based on the parameter type called by the UDF.

**Note:**

-   We recommend that you do not use classes that have the same class name but different function logic in different JAR packages. For example, in `UDF(UDAF/UDTF): udf1, udf2`, the JAR package of udf1 is `udf1.jar` and the JAR package of udf2 is `udf2.jar`. Assume that both JAR packages contain the `com.aliyun.UserFunction.class` class. If you use two UDFs in the same SQL statement at the same time, the system randomly loads either of them and may cause UDF execution inconsistency or even compilation failure.
-   If you execute an SQL statement that has two UDFs, the UDFs are not isolated. Both UDFs share the same classpath. If the resources referenced by the UDFs contain the same class, the class that classloader attempts to load is uncertain. To avoid this issue, make sure that the resources referenced by two UDFs do not contain the same class.

You can use `void setup(ExecutionContext ctx)` to initialize a UDF and `void close()` to terminate a UDF.

The method of using UDFs is the same as that of using built-in functions of MaxCompute SQL. For more information, see [Mathematical functions](/intl.en-US/Development/SQL/Builtin functions/Mathematical functions.md).

The following example shows how to implement a UDF:

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

The following example shows how to implement Concat by using the Writable type:

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

**Note:**

The package containing all Writable types is `com.aliyun.odps.io`. To use Writable types, you can download the `odps-sdk-commons` package from the [API document website](https://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-commons/0.30.9-public).

The following table describes the SDK packages provided by MaxCompute.

|Package name|Description|
|------------|-----------|
|odps-sdk-core|Provides basic functions of MaxCompute, including Tunnel and operations on tables and projects.|
|odps-sdk-commons|Provides util encapsulation.|
|odps-sdk-udf|Provides main interfaces of UDFs.|
|odps-sdk-mapred|Provides the MapReduce function.|
|odps-sdk-graph|Provides Graph Java SDK. You can search by the keyword odps-sdk-graph.|

## UDAF

Java UDAFs must inherit `com.aliyun.odps.udf.Aggregator` and implement the following interfaces:

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
     * Create an aggregation buffer * @return Writable aggregation buffer
     */
    abstract public Writable newBuffer();
    /**
     * @param buffer: aggregation buffer * @param args: a parameter specified to call a UDAF in SQL. It cannot be NULL, but the values in args can be NULL, which indicates that the input data is NULL * @throws UDFException.
     */
    abstract public void iterate(Writable buffer, Writable[] args) throws UDFException;

    /**
     * Generate the final result * @param buffer * @return Final result of Object UDAF * @throws UDFException.
     */
    abstract public Writable terminate(Writable buffer) throws UDFException;
    abstract public void merge(Writable buffer, Writable partial) throws UDFException;
}
```

The most important interfaces are `iterate`, `merge`, and `terminate` because the main logic of UDAFs relies on these interfaces. In addition, you must implement the user-defined Writable buffer.

The following figure illustrates the implementation logic and calculation procedure of the `avg` function in MaxCompute UDAFs.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/9782659951/p1855.jpg)

In the preceding figure, the input data is sliced according to the specified size. \(For more information, see [MapReduce](/intl.en-US/Development/MapReduce/Summary/Overview.md)\). The size of each slice is suitable for a worker to complete in a specified time. The slice size must be manually configured.

The calculation procedure of a UDAF involves two steps:

-   Step 1: Each worker counts the data quantity and total sum in a slice. You can consider the data quantity and total sum in each slice as an intermediate result.
-   Step 2: Each worker gathers the information about each slice generated in step 1. In the final output, `r.sum/r.count` is the average value of all input data.

The following code sample implements a UDAF that calculates the average of numbers:

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
    if (arg ! = null) {
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

The following code sample implements Concat by using a Writable type:

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

-   `Writable[] writables`: indicates a row of data. In the code, it indicates the passed column. For example, `writables[0`\] indicates the first column, and `writables[1]` indicates the second column.
-   `iterate(Writable writable,Writable[] writables)` method: `writable` indicates the aggregated data in a phase. In different Map tasks, each row of the data \(also regarded as a set\) obtained by using `group by` is executed once.
-   `merge()` method: aggregates the results obtained from different Map tasks.
-   `terminate()` method: returns data.
-   `newBuffer()` method: creates the initial value to be returned.
-   In the `readFields` method of Writable, the `readFields` method of the same object is called multiple times because the `writable` objects of partial are reused. This method resets the entire object each time it is called. If an object contains `Collection`, it is cleared.
-   UDAFs are similar to aggregate functions in MaxCompute SQL. For more information, see [Aggregate functions](/intl.en-US/Development/SQL/Builtin functions/Aggregate functions.md).
-   The method for running UDTFs is similar to that for running UDFs. For more information, see [Develop Java UDFs \(Optional\)](/intl.en-US/Quick Start/(Optional) Develop Java UDFs.md).
-   The Writable type that corresponds to STRING is Text.

## UDTF

Java UDTFs must inherit the `com.aliyun.odps.udf.UDTF` class and implement four interfaces. The following table describes the interfaces.

|Interface definition|Description|
|:-------------------|:----------|
|public void setup\(ExecutionContext ctx\) throws UDFException|The initialization method to call the user-defined initialization behavior before a UDTF processes the input data. `setup` is called once first for each worker.|
|public void process\(Object\[\] args\) throws UDFException|This method is called by the framework. Each SQL record calls `process` once. The parameters of `process` are the input parameters of the UDTF specified in the SQL statement. The input parameters are passed in as `Object[]`, and the results are returned by using the `forward` function. You must call `forward` in the `process` function to determine the output data.|
|public void close\(\) throws UDFException|The method for terminating a UDTF. The framework calls this method only once. The method is called after the last record is processed.|
|public void forward\(Object ...o\) throws UDFException|You can call the `forward` method to return data. One record is returned each time `forward` is called. The record corresponds to the column specified by the `as` clause in the SQL statement of the UDTF.|

Example

1.  The following code sample shows a UDTF.

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

    **Note:** The method for running UDTFs in MaxCompute is similar to that for running UDFs. For more information, see [Develop Java UDFs \(Optional\)](/intl.en-US/Quick Start/(Optional) Develop Java UDFs.md).

2.  Assume that you create a UDTF in MaxCompute with the registered function name of `user_udtf`. Execute the following statement to use the UDTF:

    ```
    select user_udtf(col0, col1) as (c0, c1) from my_table;
    ```

    The following example shows the values of `col0` and `col1` in `my_table`:

    ```
    +------+------+
    | col0 | col1 |
    +------+------+
    | A B | 1 |
    | C D | 2 |
    +------+------+
    ```

    The following example shows the result of `select`:

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


## Example of using UDTFs to read resources in MaxCompute

You can use UDTFs to read resources in MaxCompute. For more information, see [Resource](/intl.en-US/Product Introduction/Definitions/Resource.md). The following example describes how to use a UDTF to read resources in MaxCompute:

1.  Compile a UDTF program. After the compilation is successful, export the JAR package. In this example, the JAR package is named udtfexample1.jar.

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
       while ((line = br.readLine()) ! = null) {
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

2.  Add resources to MaxCompute.

    ```
    Add file file_resource.txt;
    Add jar udtfexample1.jar;
    Add table table_resource1 as table_resource1;
    Add table table_resource2 as table_resource2;
    ```

3.  Create a UDTF in MaxCompute. In this example, the UDTF is named `mp_udtf`.

    ```
    create function mp_udtf as com.aliyun.odps.examples.udf.UDTFResource using 
    'udtfexample1.jar, file_resource.txt, table_resource1, table_resource2';
    ```

4.  Run this UDTF.

    ```
    select mp_udtf("10","20") as (a, b, fileResourceLineCount) from tmp1;  
    -- Returned result:
    +-------+------------+-------+
    | a | b      | fileResourceLineCount |
    +-------+------------+-------+
    | 10    | 2          | fileResourceLineCount=3|tableResource1RecordCount=0|tableResource2RecordCount=0 |
    | 10    | 2          | fileResourceLineCount=3|tableResource1RecordCount=0|tableResource2RecordCount=0 |
    +-------+------------+-------+
    ```


## Examples of using complex data types in a UDF

The following example defines a UDF with three overloads. The first overload uses the ARRAY type, the second uses the MAP type, and the third uses the STRUCT type. The third overload uses STRUCT as the data type of parameters or returned values. Therefore, the `@Resolve` annotation must be added to the UDF class to specify the STRUCT type.

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

You can import complex data types into a UDF.

```
create function my_index as 'UdfArray' using 'myjar.jar'; 
select id, my_index(array('red', 'yellow', 'green'), colorOrdinal) as color_name from co
```

## Hive UDF compatibility example

MaxCompute V2.0 supports Hive UDFs. Some Hive UDFs can be directly used in MaxCompute.

**Note:** MaxCompute V2.0 is compatible with Hive 2.1.0, and the corresponding Hadoop version is 2.7.2. If you develop a UDF by using other versions of Hive or Hadoop, you may need to recompile the UDF on a compatible version.

The following example shows how to use a Hive UDF in MaxCompute:

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
      if (objectInspectors[i] ! = objectInspectors[0]) {
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

**Note:**

-   The UDF in the example supports all data types, such as ARRAY, MAP, and STRUCT.
-   For more information about how to use Hive UDFs, see the following references:
    -   [HivePlugins](https://cwiki.apache.org/confluence/display/Hive/HivePlugins)
    -   [DeveloperGuide UDTF](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide+UDTF)
    -   [GenericUDAFCaseStudy](https://cwiki.apache.org/confluence/display/Hive/GenericUDAFCaseStudy)

The UDF in the example packages any type and number of parameters into ARRAY. Assume that the output jar package is named `test.jar`.

```
-- Add a resource.
Add jar test.jar;
-- Create a function.
CREATE FUNCTION hive_collect as 'com.aliyun.odps.compiler.hive.Collect' using 'test.jar';
-- Use the function.
set odps.sql.hive.compatible=true;
select hive_collect(4y,5y,6y);
+------+
| _c0  |
+------+
| [4, 5, 6] |
+------+
```

Consider the following notes when you use Hive-compatible UDFs in MaxCompute:

-   Specify the JAR package when you create a UDF. MaxCompute does not automatically add all JAR packages to the classpath.

    **Note:** The `add jar` command in MaxCompute creates a permanent resource in the project.

-   Insert `set odps.sql.hive.compatible=true;` before an SQL statement, and commit and execute the `set` statement with the SQL statement.
-   Pay attention to the limits imposed by the [Java sandbox](/intl.en-US/Development/MapReduce/Java SDK/Java Sandbox.md) on MaxCompute.

