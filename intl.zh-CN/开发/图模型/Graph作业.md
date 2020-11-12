---
keyword: 运行MaxCompute Graph作业
---

# Graph作业

本文为您介绍如何运行MaxCompute Graph作业。

## 运行作业

MaxCompute客户端提供一个Jar命令用于运行MaxCompute Graph作业，其使用方式与[MapReduce](/intl.zh-CN/开发/MapReduce/概要/MapReduce概述.md)中的[Jar 命令](/intl.zh-CN/开发/MapReduce/功能介绍/MapReduce作业提交.md)相同。

使用语法如下。

```
Usage: jar [<GENERIC_OPTIONS>] <MAIN_CLASS> [ARGS]
    -conf <configuration_file>         Specify an application configuration file
    -classpath <local_file_list>       classpaths used to run mainClass
    -D <name>=<value>                  Property value pair, which will be used to run mainClass
    -local                             Run job in local mode
    -resources <resource_name_list>    file/table resources used in graph, seperate by comma
```

其中<GENERIC\_OPTIONS\>包括以下参数（均为可选）：

-   -conf <configuration file\>：指定JobConf配置文件。
-   -classpath <local\_file\_list\>：本地执行时的classpath，主要用于指定main函数所在的Jar包。

    通常，用户更习惯于将main函数与Graph作业编写在一个包中，例如[单源最短距离](/intl.zh-CN/开发/图模型/示例程序/单源最短距离.md)。因此，在执行示例程序时，-resources及-classpath的参数中都出现了Jar包，但二者意义不同：

    -   -resources引用的是Graph作业，运行于分布式环境中。
    -   -classpath引用的是main函数，运行于本地，指定的Jar包路径也是本地文件路径。包名之间使用系统默认的文件分割符进行分割（通常，Windows系统是分号，Linux系统是冒号）。
-   -D <prop\_name\>=<prop\_value\>：本地执行时，<mainClass\>的Java属性可以定义多个。
-   -local：以本地模式执行Graph作业，主要用于程序调试。
-   -resources <resource\_name\_list\>：Graph作业运行时使用的资源声明。通常，resource\_name\_list中需要指定运行Graph作业所使用的资源名称。如果您在Graph作业中读取了其他MaxCompute资源，则这些资源名称也需要被添加到<resource\_name\_list\>中。资源之间使用逗号分隔。如果使用跨项目空间资源时，需要在前面加上PROJECT\_NAME/resources/。示例：`-resources otherproject/resources/resfile`。

同时，您也可以直接运行Graph作业的main函数，直接将作业提交到MaxCompute，而不是通过MaxCompute客户端提交作业。以[PageRank算法](/intl.zh-CN/开发/图模型/示例程序/PageRank.md)为例，如下所示。

```
public static void main(String[] args) throws Exception {
  if (args.length < 2)
    printUsage();
  Account account = new AliyunAccount(accessId, accessKey);
  Odps odps = new Odps(account);
  odps.setEndpoint(endPoint);
  odps.setDefaultProject(project);
  SessionState ss = SessionState.get();
  ss.setOdps(odps);
  ss.setLocalRun(false);
  String resource = "mapreduce-examples.jar";
  GraphJob job = new GraphJob();
  // 将使用的jar及其他文件添加到class cache resource，对应于jar命令中-libjars中指定的资源。
  job.addCacheResourcesToClassPath(resource);
  job.setGraphLoaderClass(PageRankVertexReader.class);
  job.setVertexClass(PageRankVertex.class);
  job.addInput(TableInfo.builder().tableName(args[0]).build());
  job.addOutput(TableInfo.builder().tableName(args[1]).build());
  // default max iteration is 30
  job.setMaxIteration(30);
  if (args.length >= 3)
    job.setMaxIteration(Integer.parseInt(args[2]));
  long startTime = System.currentTimeMillis();
  job.run();
  System.out.println("Job Finished in "
      + (System.currentTimeMillis() - startTime) / 1000.0
      + " seconds");
}
```

## 输入及输出

MaxCompute Graph作业的输入输出限制为表，不允许您自定义输入输出格式。

-   作业输入定义如下。

    ```
    GraphJob job = new GraphJob();
    job.addInput(TableInfo.builder().tableName(“tblname”).build());  //表作为输入。
    job.addInput(TableInfo.builder().tableName(“tblname”).partSpec("pt1=a/pt2=b").build()); //分区作为输入。
    //只读取输入表的col2和col0列，在GraphLoader的load方法中，record.get(0)得到的是col2列，顺序一致。
    job.addInput(TableInfo.builder().tableName(“tblname”).partSpec("pt1=a/pt2=b").build(), new String[]{"col2", "col0"});
    ```

    **说明：**

    -   支持多路输入。
    -   addInput框架读取输入表的记录传给用户自定义的GraphLoader，载入图数据。
    -   暂时不支持分区过滤条件，更多应用限制请参见 [应用限制](/intl.zh-CN/开发/图模型/使用限制.md)。
-   作业输出定义如下所示。

    ```
    GraphJob job = new GraphJob();
    //输出表为分区表时需要给到最末一级分区
    job.addOutput(TableInfo.builder().tableName("table_name").partSpec("pt1=a/pt2=b").build());
    //下面的参数中，true表示覆盖tableinfo指定的分区，即INSERT OVERWRITE语义，false表示INSERT INTO语义。
    job.addOutput(TableInfo.builder().tableName("table_name").partSpec("pt1=a/pt2=b").lable("output1").build(), true);
    ```

    **说明：**

    -   支持多路输出，通过Label标识每路输出。
    -   Graph作业在运行时可以通过WorkerContext的write方法写出记录到输出表，多路输出需要指定标识。

## 读取资源

-   通过GraphJob读取资源。

    除了通过Jar命令指定Graph读取的资源外，还可以通过GraphJob的以下两种方法指定。

    ```
    void addCacheResources(String resourceNames)
    void addCacheResourcesToClassPath(String resourceNames)
    ```

-   通过WorkerContext读取资源。

    您可以通过WorkerContext的相应上下文对象读取资源。

    ```
    public byte[] readCacheFile(String resourceName) throws IOException;
    public Iterable<byte[]> readCacheArchive(String resourceName) throws IOException;
    public Iterable<byte[]> readCacheArchive(String resourceName, String relativePath)throws IOException;
    public Iterable<WritableRecord> readResourceTable(String resourceName);
    public BufferedInputStream readCacheFileAsStream(String resourceName) throws IOException;
    public Iterable<BufferedInputStream> readCacheArchiveAsStream(String resourceName) throws IOException;
    public Iterable<BufferedInputStream> readCacheArchiveAsStream(String resourceName, String relativePath) throws IOException;
    						
    ```

    **说明：**

    -   Graph也支持在WorkerComputer的setup方法中读取资源，保存在Worker Value中，之后通过getWorkerValue方法获取。
    -   如果您需要在读取资源的同时并行进行资源处理，可以使用上述流接口。该读取方式可以减少内存的消耗。
    -   更多应用限制请参见[应用限制](/intl.zh-CN/开发/图模型/使用限制.md)。

