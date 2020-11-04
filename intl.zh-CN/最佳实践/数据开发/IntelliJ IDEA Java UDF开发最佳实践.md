---
keyword: [IntelliJ IDEA, Java UDF]
---

# IntelliJ IDEA Java UDF开发最佳实践

IntelliJ IDEA是Java语言的集成开发环境，可以帮助您快速的开发Java程序。本文为您详细介绍如何使用IntelliJ IDEA进行Java UDF开发，实现大写字母转换为小写字母。

-   准备IntelliJ IDEA开发工具，请参见[安装Studio](/intl.zh-CN/工具及下载/MaxCompute Studio/工具安装与版本信息/安装IntelliJ IDEA.md)。
-   通过IntelliJ IDEA连接MaxCompute Studio[创建MaxCompute项目连接](/intl.zh-CN/工具及下载/MaxCompute Studio/管理项目连接.md)。
-   连接MaxCompute项目成功后，您需要[创建MaxCompute Java Module](/intl.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/创建MaxCompute Java Module.md)。
-   在MaxCompute上创建表并上传大写字母作为数据。详情请参见[建表并上传数据]()。建表语句如下。

    ```
    create table upperABC(upper string);
    ```


1.  创建Java UDF Project。

    1.  在IntelliJ IDEA中已创建的MaxCompute Java Module目录上右键单击**java**，选择**New** \> **MaxCompute Java**。

        ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6993359951/p100670.png)

    2.  在**Create new MaxCompute java calss**对话框中填写配置参数，单击**OK**完成创建类。

        -   **Name**：填写创建的MaxCompute Java Class名称，输入`package名称.文件名`。如果还没创建Package，可以在此处填写`packagename.classname`，会自动生成Package。
        -   **Kind**：选择创建类型，选择UDF。目前支持的类型有自定义函数（包括UDF、UDAF和UDTF）、MapReduce（包括Driver、Mapper和Reducer）、非结构化开发（包括StorageHandler和Extractor）等。
2.  编辑Java UDF代码。

    在新建的Java UDF项目中编辑代码。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6993359951/p34458.png)

    示例代码如下。

    ```
    package <package名称>;
    import com.aliyun.odps.udf.UDF;
    public class Lower extends UDF {
       public String evaluate(String s) {
          if (s == null) { 
              return null; 
          }
          return s.toLowerCase();
       }
    }
    ```

3.  测试UDF，查看UDF运行是否符合预期。测试UDF有单元测试和本地运行两种方式，本文以本地运行举例。

    1.  在Java目录下，右键单击UDF类，选择**Run '类名.main\(\)'**。

    2.  在**Run/Debug Configurations**页面上配置运行参数。

        ![debug](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6993359951/p95989.png)

        -   MaxCompute project：UDF运行使用的MaxCompute空间。本地运行时选择**local**。
        -   MaxCompute table：UDF运行时需要使用的MaxCompute表的名称。
        -   Table columns：UDF运行时需要使用的MaxCompute表的列信息。
    3.  单击**OK**，运行结果如下图。

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6993359951/p34510.png)

4.  发布UDF。测试通过后将UDF打包为JAR资源。

    1.  右键单击已经编译成功的Java代码，选择**Deploy to server…**。

    2.  在**Package a jar and submit resource**对话框中，配置相关参数。

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6993359951/p2060.png)

        -   **MaxCompute project**：指定目标MaxCompute项目的名称。
        -   **Resource name**：指定打包的资源名。
        -   **Function name**：指定打包的函数名称。
        -   **Force update if already exists**：选择当资源或函数已存在时是否强制更新。
    3.  单击**OK**，完成打包。

5.  上传JAR包。打包成功后，需要将该JAR包上传到MaxComptute服务端。

    1.  在顶部菜单栏，单击**MaxCompute** \> **添加资源**。

    2.  在**Add Resource**对话框中配置相关信息，单击**OK**。

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6993359951/p2062.png)

        -   **MaxCompute project**：指定目标MaxCompute项目的名称。
        -   **Resource file**：指定JAR包路径。
        -   **Resource name**：输入上传的资源名。
        -   **Force update if already exists**：选择当资源或函数已存在时是否强制更新。
    3.  在左侧导航栏，单击**Project Explorer**。

    4.  在**Project Explorer**区域的**Resources**节点下可以看到该资源。

6.  注册UDF。JAR包上传完成后，需要注册UDF函数后您才可以调用该函数。

    1.  单击顶部菜单栏上的**MaxCompute**，选择**创建UDF**。

    2.  在**Create Function**页面配置如下参数，然后单击**OK**。

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6993359951/p2065.png)

        -   **MaxCompute project**：选择要上传的Project名称。
        -   **Function name**：函数名称。
        -   **Using resources**：函数依赖的JAR包名称。
        -   **Main class**：JAR的主类。
        -   **Force update if already exists**：当资源或函数已存在时是否强制更新。
7.  使用UDF。

    在MaxCompute客户端上执行如下命令测试Java UDF函数。

    ```
    select Lower_test('ALIYUN');
    ```

    返回结果如下。表明使用IntelliJ IDEA上开发的Java UDF函数Lower\_test已经可用了。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7993359951/p34582.png)


