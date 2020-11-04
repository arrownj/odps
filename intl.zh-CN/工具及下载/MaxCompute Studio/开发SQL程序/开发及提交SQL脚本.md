---
keyword: [编写SQL脚本, 提交SQL脚本]
---

# 开发及提交SQL脚本

本文为您介绍如何在MaxCompute Studio上开发SQL脚本。包括编写和运行SQL脚本。

-   已连接MaxCompute项目，详情请参见[管理项目连接](/intl.zh-CN/工具及下载/MaxCompute Studio/管理项目连接.md)。
-   已创建MaxCompute Script Module，详情请参见[创建MaxCompute Script Module](/intl.zh-CN/工具及下载/MaxCompute Studio/开发SQL程序/创建MaxCompute Script Module.md)。

## 编写SQL脚本

1.  在**Project**区域下，右键单击**scripts**，选择**New** \> **MaxCompute SQL脚本**。

    ![创建](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7070380061/p1845.png)

2.  在**New MaxCompute SQL Script**对话框，配置参数信息，单击**OK**。

    ![脚本名称](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/1445819951/p1846.png)

    -   **Script Name**：脚本名称。
    -   **MaxCompute Project**：目标MaxCompute项目。单击**+**即可新建一个MaxCompute项目连接，配置详情请参见[管理项目连接](/intl.zh-CN/工具及下载/MaxCompute Studio/管理项目连接.md)。
3.  在脚本编辑界面中编写SQL。SQL语法详情请参见[SQL概述](/intl.zh-CN/开发/SQL及函数/SQL概述.md)。

    **说明：**

    -   支持跨项目空间资源依赖。例如，脚本绑定了项目A的同时，允许访问项目B下的table1（ProjectB.table1）。
    -   MaxCompute Studio支持设置SQL脚本编辑器，详情请参见概述。

## 提交SQL脚本

在提交SQL脚本前您需要根据自身需求进行相关设置。MaxCompute Studio提供了丰富的设置功能，您可以在编辑器页面上方的工具栏中快速设置。设置主要分为以下三种：

-   编辑器模式：
    -   **单句模式**：会将提交的脚本文件按`;`分隔，逐条提交到MaxCompute服务端执行。
    -   **脚本模式**：为最新开发模式，可将整条脚本一次提交到MaxCompute服务端，由MaxCompute服务端提供整体优化，效率更高，推荐使用此模式。
-   类型系统：类型系统主要解决SQL语句的兼容性问题。分为以下三种类型：
    -   **旧有类型系统**：MaxCompute旧类型的系统。
    -   **MaxCompute类型系统**：MaxCompute 2.0引入的新类型系统。
    -   **Hive类型系统**：MaxCompute 2.0引入的Hive兼容模式下的类型系统。
-   执行模式：
    -   **默认**：稳定版本。
    -   **查询加速**：包含查询加速（MCQA）新特性。
    -   **加速失败重跑**：支持作业在查询加速失败时，重新执行作业。

1.  完成SQL脚本编写后，单击工具栏或侧边栏上的![运行](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/1445819951/p95128.png)图标，即可将SQL脚本提交到MaxCompute服务端运行。

    ![运行](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7070380061/p1916.png)

    **说明：** 当SQL中存在变量时（如上图中的$\{bizdate\}），会弹出对话框，提示您输入变量值。

2.  在SQL任务运行前，IntelliJ IDEA会向您提示预估的SQL费用。确认费用后，在**Confirmation**对话框，单击**OK**。

    ![确认费用](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/1445819951/p38172.png)

    **说明：**

    -   在工具栏上，单击![刷新](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/1445819951/p95126.png)图标，可以更新SQL脚本中使用的元数据，例如表、UDF。如果MaxCompute服务端存在表或函数，但MaxCompute Studio提示表和函数不存在时，请尝试使用该功能更新元数据。
    -   SQL依赖于您在**Project Explore**窗口中添加的项目元数据，系统先在本地进行编译，无编译错误后会提交到服务端执行。
    -   SQL执行过程中会显示运行日志。当SQL开始在MaxCompute服务端运行时，会自动打开任务详情页签，显示运行作业的基本信息。
3.  在控制台**结果**页签查看SQL运行结果。

    单句模式下存在多条语句时，系统会显示每条语句的运行结果。

    ![运行结果](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4401380061/p1920.png)


