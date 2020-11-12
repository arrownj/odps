---
keyword: run a MaxCompute Graph job
---

# Graph jobs

This topic describes how to run a MaxCompute Graph job.

## Run a job

The MaxCompute client provides a JAR command for you to run MaxCompute Graph jobs. This command is used in the same way as the [JAR command](/intl.en-US/Development/MapReduce/Function Introduction/Submit a MapReduce job.md) in [MapReduce](/intl.en-US/Development/MapReduce/Summary/Overview.md).

Syntax:

```
Usage: jar [<GENERIC_OPTIONS>] <MAIN_CLASS> [ARGS]
    -conf <configuration_file>         Specify an application configuration file
    -classpath <local_file_list>       classpaths used to run mainClass
    -D <name>=<value>                  Property value pair, which will be used to run mainClass
    -local                             Run job in local mode
    -resources <resource_name_list>    file/table resources used in graph, seperate by comma
```

<GENERIC\_OPTIONS\> includes the following optional parameters:

-   -conf <configuration file\>: the JobConf configuration file.
-   -classpath <local\_file\_list\>: the classpath used to run a job in local mode. This parameter specifies the JAR package where the main function is located.

    Users prefer to write the main function and Graph job in one package, such as [SSSP](/intl.en-US/Development/Graph/Examples/SSSP.md). Therefore, the JAR package is used in both -resources and -classpath parameters of the sample program. However, the packages used in these two parameters have different meanings:

    -   -resources references Graph jobs for distributed execution.
    -   -classpath references the main function for local execution. The specified JAR package must also be saved in a local directory. Package names are separated by default file delimiters. Windows operating systems use semicolons \(;\) as the default file delimiter while Linux operating systems use colons \(:\).
-   -D <prop\_name\>=<prop\_value\>: the Java property of <mainClass\> for local execution. You can specify multiple properties.
-   -local: specifies that the Graph jobs run in local mode. It is used for program debugging.
-   -resources <resource\_name\_list\>: the resource used to run a Graph job. You must specify the names of the resources that the Graph job uses in resource\_name\_list. If you read other MaxCompute resources while you run the Graph job, you must add the names of those resources to <resource\_name\_list\>. Separate resource names with commas \(,\). If you use cross-project resources, you must add PROJECT\_NAME/resources/ before resource\_name\_list. Example: `-resources otherproject/resources/resfile`.

You can directly run the main function in the Graph job to submit the job to MaxCompute, instead of submitting the job by using the MaxCompute client. The [PageRank algorithm](/intl.en-US/Development/Graph/Examples/PageRank.md) is used in the following example:

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
  // Add the used JAR file and other files to the class cache resource. These files correspond to those specified by -libjars in the JAR command.
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

## Input and output

The input and output of MaxCompute Graph jobs must be tables. You cannot customize the input or output format.

-   Job input definition

    ```
    GraphJob job = new GraphJob();
    job.addInput(TableInfo.builder().tableName("tblname").build());  // Use tables as the input.
    job.addInput(TableInfo.builder().tableName("tblname").partSpec("pt1=a/pt2=b").build()); // Use partitions as the input.
    // Read only the col2 and col0 columns of the input table. Use record.get(0) to obtain col2 in the load() method of GraphLoader. Both are read in the same sequence.
    job.addInput(TableInfo.builder().tableName("tblname").partSpec("pt1=a/pt2=b").build(), new String[]{"col2", "col0"});
    ```

    **Note:**

    -   Multiple inputs are supported.
    -   The addInput framework reads records from the input table and transfers the records to the user-defined GraphLoader to load graph data.
    -   You cannot specify partition filter conditions. For more information about limits, see [Limits of MaxCompute Graph](/intl.en-US/Development/Graph/Limits.md).
-   Job output definition

    ```
    GraphJob job = new GraphJob();
    // If the output table is a partitioned table, the last level of partitions must be provided.
    job.addOutput(TableInfo.builder().tableName("table_name").partSpec("pt1=a/pt2=b").build());
    // true indicates that the code overwrites partitions specified by tableinfo. The value true is similar to the INSERT OVERWRITE statement. The value false is similar to the INSERT INTO statement.
    job.addOutput(TableInfo.builder().tableName("table_name").partSpec("pt1=a/pt2=b").lable("output1").build(), true);
    ```

    **Note:**

    -   Multiple outputs are supported, and each output is identified by a label.
    -   A Graph job can use the write\(\) method of WorkContext to write data to an output table during runtime. Multiple outputs must be labeled.

## Read resources

-   Use GraphJob to read resources

    In addition to the JAR command, the following two methods of GraphJob can be used to specify the resources read by Graph:

    ```
    void addCacheResources(String resourceNames)
    void addCacheResourcesToClassPath(String resourceNames)
    ```

-   Use WorkerContext to read resources

    You can read resources from the WorkerContext object.

    ```
    public byte[] readCacheFile(String resourceName) throws IOException;
    public Iterable<byte[]> readCacheArchive(String resourceName) throws IOException;
    public Iterable<byte[]> readCacheArchive(String resourceName, String relativePath)throws IOException;
    public Iterable<WritableRecord> readResourceTable(String resourceName);
    public BufferedInputStream readCacheFileAsStream(String resourceName) throws IOException;
    public Iterable<BufferedInputStream> readCacheArchiveAsStream(String resourceName) throws IOException;
    public Iterable<BufferedInputStream> readCacheArchiveAsStream(String resourceName, String relativePath) throws IOException;
    						
    ```

    **Note:**

    -   Resources can also be read by using the setup\(\) method of WorkerComputer, saved in WorkerValue, and obtained by using getWorkerValue.
    -   If you want to read resources and process them at the same time, you can use the preceding stream APIs. This method reduces memory consumption.
    -   For more information about limits, see [Limits of MaxCompute Graph](/intl.en-US/Development/Graph/Limits.md).

