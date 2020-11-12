---
keyword: add key-value pairs to JSON strings
---

# Add key-value pairs to a JSON string

This topic describes how to use a Java UDF to add one or more key-value pairs to a basic JSON string or overwrite the pairs. Nested JSON strings support this function.

## UDF used to add key-value pairs to a JSON string

```
PutJsonValues(String jsonBase,String Key1,String Value1,String Key2,String Value2...)
```

-   Description: This function is used to add one or more key-value pairs to a basic JSON string or overwrite the pairs. Nested JSON strings support this function. The number of parameters can be an odd value. This UDF can work with the `ToJsonObeject` function in [Convert data types to JSON STRING](/intl.en-US/Development/SQL/UDF examples/Convert data types to JSON STRING.md). You must first register the `ToJsonObeject` function.
-   Parameters:
    -   jsonBase: the basic JSON string to which you want to add key-value pairs.
    -   Key1: Key1 that you want to add. It is of the STRING type.
    -   Value1: Value1 that corresponds to Key1. It is of the STRING type.
    -   Key2: Key2 that you want to add. It is of the STRING type.
    -   Value2: Value2 that corresponds to Key2. It is of the STRING type.

## UDF examples

-   Resource upload

    Due to sandbox limits, to run this function in Alibaba Cloud MaxCompute, you must upload the fastjson.JSON and UDF JAR packages to MaxCompute as resources. When you create a function in DataWorks, you must reference the two packages. You can visit [MVN](https://mvnrepository.com/artifact/com.alibaba/fastjson) to download the fastjson.JSON package.

-   Function registration

    After PutJsonValues.java passes the test, register it as a function.

    In this example, PutJsonValues.jar is the JAR package that is generated by packaging the sample code of the UDF used to add key-value pairs to a JSON string. ODPSUDF-1.0-SNAPSHOT2.jar is the JAR package generated by packaging `ToJsonString` in the previous topic. fastjson-1.2.28.odps.jar is a fastjson.JSON package.

    ![](../images/p37650.png)

    **Note:**

    -   For more information about the procedure in this example, see [Create, reference, and download MaxCompute resources]().
    -   To publish a UDF to a server for production use, you must package, upload, and then register the UDF. You can use the one-click publish feature to complete these steps. MaxCompute Studio allows you to run the `mvn clean package` command, upload a JAR package, and register the UDF in sequence.
    -   You can run common commands on the client to upload resources. For more information, see [Resource operations](/intl.en-US/Development/Common commands/Resource operations.md).
-   Examples

    After the UDF is registered, execute the following statement:

    ```
    SELECT tojson(NULL)
    ,  tojson(concat('123', chr(3)))
    ,  tojson(123)
    ,  tojson(12.3)
    ,  tojson(true)
    ,  tojson(CAST(123  AS  DECIMAL))
    ,  putJsonValuesTest('{}', 'a', '123', 'b', tojson('abc'), 'c', '{"c1":567}')
    FROM dual;
    ```

    The following result is returned:

    ```
    null
    "123\u0003"
    123
    12.3
    true
    123
    {"a":123,"b":"abc","c":{"c1":567}}
    ```

    **Note:**

    -   `null` indicates the NULL value. `a` is a numeric value without quotation marks. `b` is a string which needs to be converted to a JSON string by using `tojson`. `{"c1":567}` is a valid JSON element and can be directly passed.
    -   `putJsonValues` and `tojson` are functions that you registered by using [DataWorks]() or [IntelliJ IDEA](/intl.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Package, upload, and register a Java program.md).

## UDF code example

```
import com.alibaba.fastjson.JSON; 
import com.alibaba.fastjson.JSONObject; // The fastjson.JSON class is introduced.
import com.aliyun.odps.udf.UDF;// Alibaba Cloud UDF.

public class PutJsonValues extends UDF {
// Data of the STRING type in MaxCompute is consistent with Java UDFs. The String... keyValues field is a formal parameter that is changeable, which indicates that the number of parameters is uncertain.
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

## UDF unit testing

```
// The following code is used to check whether the data type conversion takes effect. You can comment out the code when you use this function.
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
