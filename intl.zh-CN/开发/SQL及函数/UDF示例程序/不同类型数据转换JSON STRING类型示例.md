---
keyword: JSON STRING类型
---

# 不同类型数据转换JSON STRING类型示例

本文为您介绍如何通过Java UDF实现不同类型数据转换为JSON STRING类型。

在您的数据使用过程中，常常需要将不同类型的数据转换为JSON STRING类型。例如，您的数据是ARRAY类型，由于Tunnel上传下载目前不支持复杂数据类型，可能需要先将ARRAY类型数据转换为JSON STRING类型，再进行下载。

## 转换JSONStringUDF

```
toJSONString()
```

-   函数功能：将MaxCompute对象（字段）转换为JSON字符串，包括必要的字符串转义。您可以与`PutJsonValues`配合使用。
-   参数说明：输入为任意类型的参数，但不支持DATETIME类型。

## UDF使用示例

-   资源上传

    在MaxCompute公共云运行时，由于沙箱等限制，您需要把fastjson.JSON包和UDF JAR包分别作为资源上传至MaxCompute，并且在DataWorks创建函数时同时引用两个资源。您可以在[MVN](https://mvnrepository.com/artifact/com.alibaba/fastjson)手动下载fastjson.JSON包。

-   注册函数

    ToJsonStrin.java通过测试后，将其注册为函数。

    本例中ODPSUDF-1.0-SNAPSHOT2.jar是UDF打包后生成的Jar，而fastjson-1.2.28.odps.jar是fastjson.JSON包。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3982659951/p37575.png)

-   使用示例

    ```
    SELECT tojson(array(2.0,3.0,4.0));
    ```

    **说明：** `tojson`是您使用[DataWorks]()或[Intellij IDEA](/intl.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/打包、上传和注册.md)注册的函数名称。

    运行结果如下。

    ```
    [2.0,3.0,4.0]
    ```


## UDF代码示例

```
import com.aliyun.odps.udf.UDF; //阿里云UDF。
import java.math.BigDecimal;
import com.alibaba.fastjson.JSON;  //引入fastjson.JSON类。
import java.util.Arrays;
import java.util.List;

public class ToJsonString extends UDF {
//下列代码调用fastjson.JSON的toJSONString方法，将各种类型的数据转换为JSON string，您可以根据您的需要增加更多的数据类型。
    public String evaluate(String obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(Long obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(Double obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(Boolean obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(BigDecimal obj) {
        return JSON.toJSONString(obj);
    }
//MaxCompute中Array类型数据在JAVA UDF中类型为List，例如Array<double>类型可以在UDF中通过List<Double>进行匹配。
    public String evaluate(List<Double> obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(double[] obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(int[] obj) {
        return JSON.toJSONString(obj);
    }
}
```

## 单元测试

```
//下列代码为测试代码，用于测试数据转换是否生效，您在实际使用过程中可注释掉。
    public static void main(String[] args) {
        //System.out.println(new ToJsonString().evaluate(null));//java不能调用，但MaxCompute可以。
        System.out.println(new ToJsonString().evaluate("中文\t"));
        System.out.println(new ToJsonString().evaluate(123L));
        System.out.println(new ToJsonString().evaluate(1.123));
        System.out.println(new ToJsonString().evaluate(true));
        //System.out.println(new ToJsonString().evaluate(new Date()));//udf/udaf不支持datetime类型。
        System.out.println(new ToJsonString().evaluate(BigDecimal.TEN));
        System.out.println(new ToJsonString().evaluate(Arrays.asList(1.0,2.0,3.0,4.0)));
        double[] args1 = {1,2,3,4};
        System.out.println(new ToJsonString().evaluate(args1));
       }
    }
```

