---
keyword: [package, upload, register]
---

# Package, upload, and register a Java program

After a Java program is developed, you must package and publish it to MaxCompute. This topic describes how to package, upload, and register a Java program.

To publish a Java program such as a UDF, MapReduce program, or Graph program to MaxCompute for production use, you must package, upload, and register the Java program in sequence. You can use the one-click publish function to complete these procedures. MaxCompute Studio runs the mvn clean package command, uploads a JAR package, and registers a UDF in one stop.

## Package the code

1.  Right-click the compiled Java code and select **Deploy to server...**.

2.  In the **Package a jar and submit resource** dialog box, set the required parameters.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12133/15447826972060_en-US.png)

    -   **MaxCompute project**: specifies the name of the MaxCompute project.
    -   **Resource name**: specifies the name of the resource that you want to package.
    -   **Function name**: specifies the name of the function that you want to package.
    -   **Force update if already exists**: specifies whether to force the update when the resource or function already exists.
3.  Click **OK** to complete the packaging.

    **Note:** You can modify relevant settings in the pom.xml file based on your packaging requirements.


## Upload the JAR package

After the JAR package is prepared, you must upload it to MaxCompute.

1.  In the top navigation bar, choose **MaxCompute** \> **Add Resource**.

2.  In the **Add Resource** dialog box, set the required parameters and click **OK**.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12133/15447826972062_en-US.png)

    -   **MaxCompute project**: specifies the name of the MaxCompute project.
    -   **Resource file**: specifies the path of the JAR package.
    -   **Resource name**: specifies the name of the resource that you want to upload.
    -   **Force update if already exists**: specifies whether to force the update when the resource or function already exists.
3.  In the left-side navigation pane, click **Project Explorer**.

4.  After the Java package is uploaded, you can view the resource under the **Resources** node of the **Project Explorer** window.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0429644061/p2063.png)


## Register the UDF

You must register the UDF before you can call it.

1.  In the top navigation bar, click **MaxCompute** and select **Create Function**.

2.  On the **Create Function** page, set the following parameters and click **OK**.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12133/15447826972065_en-US.png)

    -   **MaxCompute project**: specifies the name of the project to which you want to upload the UDF.
    -   **Function name**: specifies the name of the UDF.
    -   **Using resources**: specifies the name of the JAR package on which the UDF depends.
    -   **Main class**: specifies the main class of the JAR package.
    -   **Force update if already exists**: specifies whether to force the update when the resource or function already exists.
3.  In the left-side navigation pane, click **Project Explorer**.

4.  After the UDF is registered, you can view the UDF under the **Functions** node of the **Project Explorer** window.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0429644061/p2066.png)


