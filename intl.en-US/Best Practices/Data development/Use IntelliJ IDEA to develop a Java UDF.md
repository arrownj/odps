---
keyword: [IntelliJ IDEA, Java UDF]
---

# Use IntelliJ IDEA to develop a Java UDF

IntelliJ IDEA is an integrated development environment \(IDE\) for developing Java programs. This topic describes how to use IntelliJ IDEA to develop a user-defined function \(UDF\) that converts uppercase letters to lowercase letters.

-   IntelliJ IDEA is installed. For more information, see [Install IntelliJ IDEA](/intl.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md).
-   A MaxCompute project is connected by using MaxCompute Studio in IntelliJ IDEA. For more information, see [Manage project connections](/intl.en-US/Tools and Downloads/MaxCompute Studio/Manage project connections.md).
-   A MaxCompute Java module is created. For more information, see [Create a MaxCompute Java module](/intl.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md).
-   A MaxCompute table is created. Strings of uppercase letters are inserted into the table. For more information, see [Create tables and import data](). You can use the following example statement to create a table:

    ```
    create table upperABC(upper string);
    ```


1.  Create a Java UDF project.

    1.  Open MaxCompute Studio in IntelliJ IDEA. Find the directory where your MaxCompute Java module is created. Right-click the **java** folder and choose **New** \> **MaxCompute Java**.

        ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/8317544061/p100670.png)

    2.  In the **Create new MaxCompute java class** dialog box, set the Name and Kind parameters and click **OK**.

        -   **Name**: the name of the MaxCompute Java class that you want to create, in the `Package name.Class name` format. If the specified package does not exist, the system automatically creates one.``
        -   **Kind**: the type of the MaxCompute Java class. Set this parameter to UDF in this example. Valid values for this parameter also include UDAF, UDTF, Driver, Mapper, Reducer, StorageHandler, and Extractor.
2.  Write code to develop a Java UDF.

    Write code in your Java UDF project, as shown in the following figure.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0511359951/p34458.png)

    In this example, use the following code:

    ```
    package <Package name>;
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

3.  Test the UDF. You can run the UDF in IntelliJ IDEA or perform a unit test. The following procedure shows how to test the UDF by running it in IntelliJ IDEA:

    1.  In the java folder, right-click your UDF class and select **Run 'Class name.main\(\)'**.

    2.  In the **Run/Debug Configurations** dialog box, set the parameters as required.

        ![debug](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/9040744061/p95989.png)

        -   MaxCompute project: the MaxCompute project that is used to run the UDF. To run the UDF in IntelliJ IDEA, select **local**.
        -   MaxCompute table: the name of the MaxCompute table that you want to use for testing the UDF.
        -   Table columns: the columns in the MaxCompute table on which you want the UDF to run.
    3.  Click **OK**. The following figure shows the result that the UDF returns.

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/1511359951/p34510.png)

4.  Package the UDF into a JAR file.

    1.  In the java folder, right-click the UDF and select **Deploy to server…**.

    2.  In the **Package a jar and submit resource** dialog box, set the parameters as required.

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12133/15447826972060_en-US.png)

        -   **MaxCompute project**: the name of the MaxCompute project to which you want to upload the JAR package.
        -   **Resource name**: the name of the JAR package.
        -   **Function name**: the name of the UDF.
        -   **Force update if already exists**: specifies whether to overwrite an existing JAR package or function in the destination MaxCompute project that has the same name as the JAR package or function to be uploaded.
    3.  Click **OK**.

5.  Upload the JAR package to MaxCompute.

    1.  In the top navigation bar, choose **MaxCompute** \> **Add Resource**.

    2.  In the **Add Resource** dialog box, set the parameters as required and click **OK**.

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12133/15447826972062_en-US.png)

        -   **MaxCompute project**: the name of the MaxCompute project to which you want to upload the JAR package.
        -   **Resource file**: the directory where the JAR package is stored in IntelliJ IDEA.
        -   **Resource name**: the name of the JAR package.
        -   **Force update if already exists**: specifies whether to overwrite an existing JAR package or function in the destination MaxCompute project that has the same name as the JAR package or function to be uploaded.
    3.  On the left-side navigation submenu, click **Project Explorer**.

    4.  In the **Project Explorer** section, click **Resources** and verify that the JAR package is uploaded.

6.  Register the UDF. You must register the UDF before you can call it.

    1.  In the top navigation bar, choose **MaxCompute** \> **Create Function**.

    2.  In the **Create Function** dialog box, set the parameters as required and click **OK**.

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12133/15447826972065_en-US.png)

        -   **MaxCompute project**: the name of the MaxCompute project to which you want to upload the UDF.
        -   **Function name**: the name of the UDF.
        -   **Using resources**: the JAR package of the UDF.
        -   **Main class**: the main class of the UDF.
        -   **Force update if already exists**: specifies whether to overwrite an existing JAR package or function in the destination MaxCompute project that has the same name as the JAR package or function to be uploaded.
7.  Call the UDF.

    On the MaxCompute client, execute the following statement to test the Java UDF:

    ```
    select Lower_test('ALIYUN');
    ```

    The following figure shows the result that the preceding statement returns. The result indicates that the Java UDF Lower\_test works properly.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/1511359951/p34582.png)


