---
keyword: JSON字符串增加键值
---

# JSON字符串增加键值示例

本文为您介绍如何通过Java UDF实现向基础JSON字符串中增加（覆盖）一个或多个键值对（支持嵌套JSON）。

## JSON字符串增加键值UDF说明

```
PutJsonValues(String jsonBase,String Key1,String Value1,String Key2,String Value2...)
```

-   函数功能：向基础JSON串中增加（覆盖）一个或多个键值对支持嵌套JSON，接受奇数个参数。可以和[不同类型数据转换JSON STRING类型示例](/cn.zh-CN/开发/SQL及函数/UDF示例程序/不同类型数据转换JSON STRING类型示例.md)中的`ToJsonObeject`配合使用（请先完成`ToJsonObeject`函数的注册）。
-   参数说明：
    -   jsonBase：基础JSON串。
    -   Key1：增加的Key1值，STRING类型。
    -   Value1：增加的Key1对应的Value1值，STRING类型。
    -   Key2：增加的Key2值，STRING类型。
    -   Value2：增加的Key2对应的Value2值，STRING类型。

## UDF使用示例

-   资源上传

    实际在MaxCompute公共云运行时，由于沙箱等限制，需要把fastjson.JSON包和UDF JAR包分别作为资源上传至MaxCompute，并且在DataWorks创建函数时同时引用两个资源。您可以在[MVN](https://mvnrepository.com/artifact/com.alibaba/fastjson)手动下载fastjson.JSON包。

-   注册函数

    PutJsonValues.java测试通过后，将其注册函数。

    本例中PutJsonValues.jar是UDF打包后生成的JAR包，ODPSUDF-1.0-SNAPSHOT2.jar是上例`ToJsonString`打包后生成的JAR包，而fastjson-1.2.28.odps.jar是fastjson.JSON包。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3982659951/p37650.png)

    **说明：**

    -   本例的操作步骤详情请参见[创建MaxCompute资源]()。
    -   一个UDF从发布到服务端供生产使用，需要经过打包、上传、注册三个步骤。您可以使用一键发布功能一次性完成这些步骤（MaxCompute Studio会依次执行`mvn clean package`、上传JAR包和注册UDF这三个步骤）。
    -   通过客户端使用常用命令进行资源上传操作，请参见[资源操作](/cn.zh-CN/开发/常用命令/资源操作.md)。
-   使用示例

    成功注册UDF后，执行以下命令。

    ```
    SELECT tojson(NULL)
    ,  tojson(concat('123', chr(3)))
    ,  tojson(123)
    ,  tojson(12.3)
    ,  tojson(true)
    ,  tojson(CAST(123  AS  DECIMAL))
    ,  putJsonValuesTest('{}', 'a', '123', 'b', tojson('abc'), 'c', '{"c1":567}')
    ;
    ```

    运行结果：

    ```
    null
    "123\u0003"
    123
    12.3
    true
    123
    {"a":123,"b":"abc","c":{"c1":567}}
    ```

    **说明：**

    -   `null`代表NULL。`a`的值是数值没有引号。`b`的值是字符串，需要使用`tojson`转义为JSON字符串。`{"c1":567}`本身已经是一个合法的JSON元素，因此可以直接传入。
    -   `putJsonValues`、`tojson`是您使用[DataWorks]()或[Intellij IDEA](/cn.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/打包、上传和注册.md)注册的函数名称。

## UDF代码示例

```
import com.alibaba.fastjson.JSON; 
import com.alibaba.fastjson.JSONObject; //引入fastjson.JSON类。
import com.aliyun.odps.udf.UDF;//阿里云UDF。

public class PutJsonValues extends UDF {
//MaxCompute中String类型数据与JAVA UDF中类型一致，String... keyValues为可变形参，表示参数个数不定。
    public  PutJsonValues(){
    }

    public  String evaluate(String jsonBase, String... keyValues) {
        if (jsonBase == null) {
            return null;
        }
        JSONObject outputObj = JSON.parseObject(jsonBase);
        for (int i = 1; i < keyValues.length; i = i + 2) {
            String key = keyValues[i - 1];
            String value = keyValues[i];
            if (key == null || value == null) {
                continue;
            }
            Object valueObj = JSON.parse(value);
            outputObj.put(key, valueObj);
        }
        return JSON.toJSONString(outputObj);
    }
}
```

## 单元测试

```
//下列代码为测试代码，用于测试数据转换是否生效，您在实际使用过程中可注释掉。
public static void main(String[] args) {
    PutJsonValues putJsonValues=new PutJsonValues();
    System.out.println(putJsonValues.hashCode());

    String js=putJsonValues.evaluate("{}", "k", "null");
    System.out.println(js);
    System.out.println(new PutJsonValues().evaluate("{}", "a", "123", "b", "234", "c", "345"));
    System.out.println(new PutJsonValues().evaluate("{}", "k", "\"abc\""));
    System.out.println(new PutJsonValues().evaluate("{}", "k", "{\"kk\":123}"));
}
```

