# Use errors and alerts in the MaxCompute compiler for troubleshooting

The MaxCompute compiler is based on the new-generation SQL engine of MaxCompute V2.0. This simplifies the SQL compilation and improves language expressions. This topic describes how to use errors and alerts in the MaxCompute compiler for troubleshooting.

We recommend that you use MaxCompute Studio in combination with the compiler. This improves the usability of the MaxCompute compiler.

[Install MaxCompute Studio](/intl.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Installation procedure.md), add a MaxCompute project, create a MaxCompute Studio project, and then create a MaxCompute script file.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7192659951/p2373.png)

In the preceding figure, the following errors exist:

-   In the first INSERT statement, an error occurs on the use of the wm\_concat function.
-   In the second INSERT statement, a column name error occurs and a data type conversion alert is generated. In MaxCompute, if a BIGINT value is compared with a DOUBLE value, an implicit conversion to the DOUBLE type is performed. A conversion error may occur. Therefore, the MaxCompute compiler generates an alert for you to check whether the conversion meets your expectation.

Move the pointer over an error or alert. The details of the error or alert are displayed. If you ignore the error or alert and submit your script, the script is blocked by MaxCompute Studio.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7192659951/p2378.png)

Therefore, you must handle the error or alert based on the message.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7192659951/p2374.png)

After the modification, submit the script again.

You can also use MaxCompute Studio to specify all alerts as errors.

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7192659951/p2375.png)

This way, no errors are missed.

Before you submit scripts, we recommend that you use MaxCompute Studio to perform static compilation checks for the scripts. We also recommend that you use MaxCompute Studio to specify alerts as errors and handle all alerts before you submit the scripts. This saves time and resources. If you submit a script with errors, your health score is reduced. This affects the priorities and order that you use to submit tasks. In later versions, unhandled alerts will be considered as a factor of the health score system. Therefore, you can fully utilize the details of the errors and alerts in the MaxCompute compiler to avoid priority downgrade.

Some alerts indicate the implicit type conversions that are not secure. If you are sure of a conversion, use cast \(xxx as\) to clear the alert. The MaxCompute compiler also allows you to use \(xxx\) to clear alerts.

