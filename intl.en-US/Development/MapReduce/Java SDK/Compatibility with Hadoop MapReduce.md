---
keyword: MaxCompute MapReduce interface
---

# Compatibility with Hadoop MapReduce

This topic describes the compatibility between specific MaxCompute MapReduce interfaces and Hadoop MapReduce.

The following table describes whether specific MaxCompute MapReduce interfaces are compatible with Hadoop MapReduce.

|Type|Interface|Compatible with Hadoop MapReduce|
|:---|:--------|:-------------------------------|
|Mapper|void map\(KEYIN key, VALUEIN value, org.apache.hadoop.mapreduce.Mapper.Context context\)|Yes.|
|Mapper|void run\(org.apache.hadoop.mapreduce.Mapper.Context context\)|Yes.|
|Mapper|void setup\(org.apache.hadoop.mapreduce.Mapper.Context context\)|Yes.|
|Reducer|void cleanup\(org.apache.hadoop.mapreduce.Reducer.Context context\)|Yes.|
|Reducer|void reduce\(KEYIN key, VALUEIN value, org.apache.hadoop.mapreduce.Reducer.Context context\)|Yes.|
|Reducer|void run\(org.apache.hadoop.mapreduce.Reducer.Context context\)|Yes.|
|Reducer|void setup\(org.apache.hadoop.mapreduce.Reducer.Context context\)|Yes.|
|Partitioner|int getPartition\(KEY key, VALUE value, int numPartitions\)|Yes.|
|MapContext, which extends TaskInputOutputContext|InputSplit getInputSplit\(\)|No. Exceptions occur.|
|ReduceContext|nextKey\(\)|Yes.|
|ReduceContext|getValues\(\)|Yes.|
|TaskInputOutputContext|getCurrentKey\(\)|Yes.|
|TaskInputOutputContext|getCurrentValue\(\)|Yes.|
|TaskInputOutputContext|getOutputCommitter\(\)|No. Exceptions occur.|
|TaskInputOutputContext|nextKeyValue\(\)|Yes.|
|TaskInputOutputContext|write\(KEYOUT key, VALUEOUT value\)|Yes.|
|TaskAttemptContext|getCounter\(Enum<? \> counterName\)|Yes.|
|TaskAttemptContext|getCounter\(String groupName, String counterName\)|Yes.|
|TaskAttemptContext|setStatus\(String msg\)|Empty implementation.|
|TaskAttemptContext|getStatus\(\)|Empty implementation.|
|TaskAttemptContext|getTaskAttemptID\(\)|No. Exceptions occur.|
|TaskAttemptContext|getProgress\(\)|No. Exceptions occur.|
|TaskAttemptContext|progress\(\)|Yes.|
|Job|addArchiveToClassPath\(Path archive\)|No.|
|Job|addCacheArchive\(URI uri\)|No.|
|Job|addCacheFile\(URI uri\)|No.|
|Job|addFileToClassPath\(Path file\)|No.|
|Job|cleanupProgress\(\)|No.|
|Job|createSymlink\(\)|No. Exceptions occur.|
|Job|failTask\(TaskAttemptID taskId\)|No.|
|Job|getCompletionPollInterval\(Configuration conf\)|Empty implementation.|
|Job|getCounters\(\)|Yes.|
|Job|getFinishTime\(\)|Yes.|
|Job|getHistoryUrl\(\)|Yes.|
|Job|getInstance\(\)|Yes.|
|Job|getInstance\(Cluster ignored\)|Yes.|
|Job|getInstance\(Cluster ignored, Configuration conf\)|Yes.|
|Job|getInstance\(Configuration conf\)|Yes.|
|Job|getInstance\(Configuration conf, String jobName\)|Empty implementation.|
|Job|getInstance\(JobStatus status, Configuration conf\)|No. Exceptions occur.|
|Job|getJobFile\(\)|No. Exceptions occur.|
|Job|getJobName\(\)|Empty implementation.|
|Job|getJobState\(\)|No. Exceptions occur.|
|Job|getPriority\(\)|No. Exceptions occur.|
|Job|getProgressPollInterval\(Configuration conf\)|Empty implementation.|
|Job|getReservationId\(\)|No. Exceptions occur.|
|Job|getSchedulingInfo\(\)|No. Exceptions occur.|
|Job|getStartTime\(\)|Yes.|
|Job|getStatus\(\)|No. Exceptions occur.|
|Job|getTaskCompletionEvents\(int startFrom\)|No. Exceptions occur.|
|Job|getTaskCompletionEvents\(int startFrom, int numEvents\)|No. Exceptions occur.|
|Job|getTaskDiagnostics\(TaskAttemptID taskid\)|No. Exceptions occur.|
|Job|getTaskOutputFilter\(Configuration conf\)|No. Exceptions occur.|
|Job|getTaskReports\(TaskType type\)|No. Exceptions occur.|
|Job|getTrackingURL\(\)|Yes.|
|Job|isComplete\(\)|Yes.|
|Job|isRetired\(\)|No. Exceptions occur.|
|Job|isSuccessful\(\)|Yes.|
|Job|isUber\(\)|Empty implementation.|
|Job|killJob\(\)|Yes.|
|Job|killTask\(TaskAttemptID taskId\)|No.|
|Job|mapProgress\(\)|Yes.|
|Job|monitorAndPrintJob\(\)|Yes.|
|Job|reduceProgress\(\)|Yes.|
|Job|setCacheArchives\(URI\[\] archives\)|No. Exceptions occur.|
|Job|setCacheFiles\(URI\[\] files\)|No. Exceptions occur.|
|Job|setCancelDelegationTokenUponJobCompletion\(boolean value\)|No. Exceptions occur.|
|Job|setCombinerClass\(Class<? extends Reducer\> cls\)|Yes.|
|Job|setCombinerKeyGroupingComparatorClass\(Class<? extends RawComparator\> cls\)|Yes.|
|Job|setGroupingComparatorClass\(Class<? extends RawComparator\> cls\)|Yes.|
|Job|setInputFormatClass\(Class<? extends InputFormat\> cls\)|Empty implementation.|
|Job|setJar\(String jar\)|Yes.|
|Job|setJarByClass\(Class<? \> cls\)|Yes.|
|Job|setJobName\(String name\)|Empty implementation.|
|Job|setJobSetupCleanupNeeded\(boolean needed\)|Empty implementation.|
|Job|setMapOutputKeyClass\(Class<? \> theClass\)|Yes.|
|Job|setMapOutputValueClass\(Class<? \> theClass\)|Yes.|
|Job|setMapperClass\(Class<? extends Mapper\> cls\)|Yes.|
|Job|setMapSpeculativeExecution\(boolean speculativeExecution\)|Empty implementation.|
|Job|setMaxMapAttempts\(int n\)|Empty implementation.|
|Job|setMaxReduceAttempts\(int n\)|Empty implementation.|
|Job|setNumReduceTasks\(int tasks\)|Yes.|
|Job|setOutputFormatClass\(Class<? extends OutputFormat\> cls\)|No. Exceptions occur.|
|Job|setOutputKeyClass\(Class<? \> theClass\)|Yes.|
|Job|setOutputValueClass\(Class<? \> theClass\)|Yes.|
|Job|setPartitionerClass\(Class<? extends Partitioner\> cls\)|Yes.|
|Job|setPriority\(JobPriority priority\)|No. Exceptions occur.|
|Job|setProfileEnabled\(boolean newValue\)|Empty implementation.|
|Job|setProfileParams\(String value\)|Empty implementation.|
|Job|setProfileTaskRange\(boolean isMap, String newValue\)|Empty implementation.|
|Job|setReducerClass\(Class<? extends Reducer\> cls\)|Yes.|
|Job|setReduceSpeculativeExecution\(boolean speculativeExecution\)|Empty implementation.|
|Job|setReservationId\(ReservationId reservationId\)|No. Exceptions occur.|
|Job|setSortComparatorClass\(Class<? extends RawComparator\> cls\)|No. Exceptions occur.|
|Job|setSpeculativeExecution\(boolean speculativeExecution\)|Yes.|
|Job|setTaskOutputFilter\(Configuration conf, org.apache.hadoop.mapreduce.Job.TaskStatusFilter newValue\)|No. Exceptions occur.|
|Job|setupProgress\(\)|No. Exceptions occur.|
|Job|setUser\(String user\)|Empty implementation.|
|Job|setWorkingDirectory\(Path dir\)|Empty implementation.|
|Job|submit\(\)|Yes.|
|Job|toString\(\)|No. Exceptions occur.|
|Job|waitForCompletion\(boolean verbose\)|Yes.|
|Task Execution & Environment|mapreduce.map.java.opts|Empty implementation.|
|Task Execution & Environment|mapreduce.reduce.java.opts|Empty implementation.|
|Task Execution & Environment|mapreduce.map.memory.mb|Empty implementation.|
|Task Execution & Environment|mapreduce.reduce.memory.mb|Empty implementation.|
|Task Execution & Environment|mapreduce.task.io.sort.mb|Empty implementation.|
|Task Execution & Environment|mapreduce.map.sort.spill.percent|Empty implementation.|
|Task Execution & Environment|mapreduce.task.io.soft.factor|Empty implementation.|
|Task Execution & Environment|mapreduce.reduce.merge.inmem.thresholds|Empty implementation.|
|Task Execution & Environment|mapreduce.reduce.shuffle.merge.percent|Empty implementation.|
|Task Execution & Environment|mapreduce.reduce.shuffle.input.buffer.percent|Empty implementation.|
|Task Execution & Environment|mapreduce.reduce.input.buffer.percent|Empty implementation.|
|Task Execution & Environment|mapreduce.job.id|Empty implementation.|
|Task Execution & Environment|mapreduce.job.jar|Empty implementation.|
|Task Execution & Environment|mapreduce.job.local.dir|Empty implementation.|
|Task Execution & Environment|mapreduce.task.id|Empty implementation.|
|Task Execution & Environment|mapreduce.task.attempt.id|Empty implementation.|
|Task Execution & Environment|mapreduce.task.is.map|Empty implementation.|
|Task Execution & Environment|mapreduce.task.partition|Empty implementation.|
|Task Execution & Environment|mapreduce.map.input.file|Empty implementation.|
|Task Execution & Environment|mapreduce.map.input.start|Empty implementation.|
|Task Execution & Environment|mapreduce.map.input.length|Empty implementation.|
|Task Execution & Environment|mapreduce.task.output.dir|Empty implementation.|
|JobClient|cancelDelegationToken\(Token <DelegationTokenIdentifier\> token\)|No. Exceptions occur.|
|JobClient|close\(\)|Empty implementation.|
|JobClient|displayTasks\(JobID jobId, String type, String state\)|No. Exceptions occur.|
|JobClient|getAllJobs\(\)|No. Exceptions occur.|
|JobClient|getCleanupTaskReports\(JobID jobId\)|No. Exceptions occur.|
|JobClient|getClusterStatus\(\)|No. Exceptions occur.|
|JobClient|getClusterStatus\(boolean detailed\)|No. Exceptions occur.|
|JobClient|getDefaultMaps\(\)|No. Exceptions occur.|
|JobClient|getDefaultReduces\(\)|No. Exceptions occur.|
|JobClient|getDelegationToken\(Text renewer\)|No. Exceptions occur.|
|JobClient|getFs\(\)|No. Exceptions occur.|
|JobClient|getJob\(JobID jobid\)|No. Exceptions occur.|
|JobClient|getJob\(String jobid\)|No. Exceptions occur.|
|JobClient|getJobsFromQueue\(String queueName\)|No. Exceptions occur.|
|JobClient|getMapTaskReports\(JobID jobId\)|No. Exceptions occur.|
|JobClient|getMapTaskReports\(String jobId\)|No. Exceptions occur.|
|JobClient|getQueueAclsForCurrentUser\(\)|No. Exceptions occur.|
|JobClient|getQueueInfo\(String queueName\)|No. Exceptions occur.|
|JobClient|getQueues\(\)|No. Exceptions occur.|
|JobClient|getReduceTaskReports\(JobID jobId\)|No. Exceptions occur.|
|JobClient|getReduceTaskReports\(String jobId\)|No. Exceptions occur.|
|JobClient|getSetupTaskReports\(JobID jobId\)|No. Exceptions occur.|
|JobClient|getStagingAreaDir\(\)|No. Exceptions occur.|
|JobClient|getSystemDir\(\)|No. Exceptions occur.|
|JobClient|getTaskOutputFilter\(\)|No. Exceptions occur.|
|JobClient|getTaskOutputFilter\(JobConf job\)|No. Exceptions occur.|
|JobClient|init\(JobConf conf\)|No. Exceptions occur.|
|JobClient|isJobDirValid\(Path jobDirPath, FileSystem fs\)|No. Exceptions occur.|
|JobClient|jobsToComplete\(\)|No. Exceptions occur.|
|JobClient|monitorAndPrintJob\(JobConf conf, RunningJob job\)|No. Exceptions occur.|
|JobClient|renewDelegationToken\(Token<DelegationTokenIdentifier\> token\)|No. Exceptions occur.|
|JobClient|run\(String\[\] argv\)|No. Exceptions occur.|
|JobClient|runJob\(JobConf job\)|Yes.|
|JobClient|setTaskOutputFilter\(JobClient.TaskStatusFilter newValue\)|No. Exceptions occur.|
|JobClient|setTaskOutputFilter\(JobConf job, JobClient.TaskStatusFilter newValue\)|No. Exceptions occur.|
|JobClient|submitJob\(JobConf job\)|Yes.|
|JobClient|submitJob\(String jobFile\)|No. Exceptions occur.|
|JobConf|deleteLocalFiles\(\)|No. Exceptions occur.|
|JobConf|deleteLocalFiles\(String subdir\)|No. Exceptions occur.|
|JobConf|normalizeMemoryConfigValue\(long val\)|Empty implementation.|
|JobConf|setCombinerClass\(Class<? extends Reducer\> theClass\)|Yes.|
|JobConf|setCompressMapOutput\(boolean compress\)|Empty implementation.|
|JobConf|setInputFormat\(Class<? extends InputFormat\> theClass\)|No. Exceptions occur.|
|JobConf|setJar\(String jar\)|No. Exceptions occur.|
|JobConf|setJarByClass\(Class cls\)|No. Exceptions occur.|
|JobConf|setJobEndNotificationURI\(String uri\)|No. Exceptions occur.|
|JobConf|setJobName\(String name\)|Empty implementation.|
|JobConf|setJobPriority\(JobPriority prio\)|No. Exceptions occur.|
|JobConf|setKeepFailedTaskFiles\(boolean keep\)|No. Exceptions occur.|
|JobConf|setKeepTaskFilesPattern\(String pattern\)|No. Exceptions occur.|
|JobConf|setKeyFieldComparatorOptions\(String keySpec\)|No. Exceptions occur.|
|JobConf|setKeyFieldPartitionerOptions\(String keySpec\)|No. Exceptions occur.|
|JobConf|setMapDebugScript\(String mDbgScript\)|Empty implementation.|
|JobConf|setMapOutputCompressorClass\(Class<? extends CompressionCodec\> codecClass\)|Empty implementation.|
|JobConf|setMapOutputKeyClass\(Class<? \> theClass\)|Yes.|
|JobConf|setMapOutputValueClass\(Class<? \> theClass\)|Yes.|
|JobConf|setMapperClass\(Class<? extends Mapper\> theClass\)|Yes.|
|JobConf|setMapRunnerClass\(Class<? extends MapRunnable\> theClass\)|No. Exceptions occur.|
|JobConf|setMapSpeculativeExecution\(boolean speculativeExecution\)|Empty implementation.|
|JobConf|setMaxMapAttempts\(int n\)|Empty implementation.|
|JobConf|setMaxMapTaskFailuresPercent\(int percent\)|Empty implementation.|
|JobConf|setMaxPhysicalMemoryForTask\(long mem\)|Empty implementation.|
|JobConf|setMaxReduceAttempts\(int n\)|Empty implementation.|
|JobConf|setMaxReduceTaskFailuresPercent\(int percent\)|Empty implementation.|
|JobConf|setMaxTaskFailuresPerTracker\(int noFailures\)|Empty implementation.|
|JobConf|setMaxVirtualMemoryForTask\(long vmem\)|Empty implementation.|
|JobConf|setMemoryForMapTask\(long mem\)|Yes.|
|JobConf|setMemoryForReduceTask\(long mem\)|Yes.|
|JobConf|setNumMapTasks\(int n\)|Yes.|
|JobConf|setNumReduceTasks\(int n\)|Yes.|
|JobConf|setNumTasksToExecutePerJvm\(int numTasks\)|Empty implementation.|
|JobConf|setOutputCommitter\(Class<? extends OutputCommitter\> theClass\)|No. Exceptions occur.|
|JobConf|setOutputFormat\(Class<? extends OutputFormat\> theClass\)|Empty implementation.|
|JobConf|setOutputKeyClass\(Class<? \> theClass\)|Yes.|
|JobConf|setOutputKeyComparatorClass\(Class<? extends RawComparator\> theClass\)|No. Exceptions occur.|
|JobConf|setOutputValueClass\(Class<? \> theClass\)|Yes.|
|JobConf|setOutputValueGroupingComparator\(Class<? extends RawComparator\> theClass\)|No. Exceptions occur.|
|JobConf|setPartitionerClass\(Class<? extends Partitioner\> theClass\)|Yes.|
|JobConf|setProfileEnabled\(boolean newValue\)|Empty implementation.|
|JobConf|setProfileParams\(String value\)|Empty implementation.|
|JobConf|setProfileTaskRange\(boolean isMap, String newValue\)|Empty implementation.|
|JobConf|setQueueName\(String queueName\)|No. Exceptions occur.|
|JobConf|setReduceDebugScript\(String rDbgScript\)|Empty implementation.|
|JobConf|setReducerClass\(Class<? extends Reducer\> theClass\)|Yes.|
|JobConf|setReduceSpeculativeExecution\(boolean speculativeExecution\)|Empty implementation.|
|JobConf|setSessionId\(String sessionId\)|Empty implementation.|
|JobConf|setSpeculativeExecution\(boolean speculativeExecution\)|No. Exceptions occur.|
|JobConf|setUseNewMapper\(boolean flag\)|Yes.|
|JobConf|setUseNewReducer\(boolean flag\)|Yes.|
|JobConf|setUser\(String user\)|Empty implementation.|
|JobConf|setWorkingDirectory\(Path dir\)|Empty implementation.|
|FileInputFormat|N/A|No. Exceptions occur.|
|TextInputFormat|N/A|Yes.|
|InputSplit|mapred.min.split.size.|No. Exceptions occur.|
|FileSplit|map.input.file|No. Exceptions occur.|
|RecordWriter|N/A|No. Exceptions occur.|
|RecordReader|N/A|No. Exceptions occur.|
|OutputFormat|N/A|No. Exceptions occur.|
|OutputCommitter|abortJob\(JobContext jobContext, int status\)|No. Exceptions occur.|
|OutputCommitter|abortJob\(JobContext context, JobStatus.State runState\)|No. Exceptions occur.|
|OutputCommitter|abortTask\(TaskAttemptContext taskContext\)|No. Exceptions occur.|
|OutputCommitter|abortTask\(TaskAttemptContext taskContext\)|No. Exceptions occur.|
|OutputCommitter|cleanupJob\(JobContext jobContext\)|No. Exceptions occur.|
|OutputCommitter|cleanupJob\(JobContext context\)|No. Exceptions occur.|
|OutputCommitter|commitJob\(JobContext jobContext\)|No. Exceptions occur.|
|OutputCommitter|commitJob\(JobContext context\)|No. Exceptions occur.|
|OutputCommitter|commitTask\(TaskAttemptContext taskContext\)|No. Exceptions occur.|
|OutputCommitter|needsTaskCommit\(TaskAttemptContext taskContext\)|No. Exceptions occur.|
|OutputCommitter|needsTaskCommit\(TaskAttemptContext taskContext\)|No. Exceptions occur.|
|OutputCommitter|setupJob\(JobContext jobContext\)|No. Exceptions occur.|
|OutputCommitter|setupJob\(JobContext jobContext\)|No. Exceptions occur.|
|OutputCommitter|setupTask\(TaskAttemptContext taskContext\)|No. Exceptions occur.|
|OutputCommitter|setupTask\(TaskAttemptContext taskContext\)|No. Exceptions occur.|
|Counter|getDisplayName\(\)|Yes.|
|Counter|getName\(\)|Yes.|
|Counter|getValue\(\)|Yes.|
|Counter|increment\(long incr\)|Yes.|
|Counter|setValue\(long value\)|Yes.|
|Counter|setDisplayName\(String displayName\)|Yes.|
|DistributedCache|CACHE\_ARCHIVES|No. Exceptions occur.|
|DistributedCache|CACHE\_ARCHIVES\_SIZES|No. Exceptions occur.|
|DistributedCache|CACHE\_ARCHIVES\_TIMESTAMPS|No. Exceptions occur.|
|DistributedCache|CACHE\_FILES|No. Exceptions occur.|
|DistributedCache|CACHE\_FILES\_SIZES|No. Exceptions occur.|
|DistributedCache|CACHE\_FILES\_TIMESTAMPS|No. Exceptions occur.|
|DistributedCache|CACHE\_LOCALARCHIVES|No. Exceptions occur.|
|DistributedCache|CACHE\_LOCALFILES|No. Exceptions occur.|
|DistributedCache|CACHE\_SYMLINK|No. Exceptions occur.|
|DistributedCache|addArchiveToClassPath\(Path archive, Configuration conf\)|No. Exceptions occur.|
|DistributedCache|addArchiveToClassPath\(Path archive, Configuration conf, FileSystem fs\)|No. Exceptions occur.|
|DistributedCache|addCacheArchive\(URI uri, Configuration conf\)|No. Exceptions occur.|
|DistributedCache|addCacheFile\(URI uri, Configuration conf\)|No. Exceptions occur.|
|DistributedCache|addFileToClassPath\(Path file, Configuration conf\)|No. Exceptions occur.|
|DistributedCache|addFileToClassPath\(Path file, Configuration conf, FileSystem fs\)|No. Exceptions occur.|
|DistributedCache|addLocalArchives\(Configuration conf, String str\)|No. Exceptions occur.|
|DistributedCache|addLocalFiles\(Configuration conf, String str\)|No. Exceptions occur.|
|DistributedCache|checkURIs\(URI\[\] uriFiles, URI\[\] uriArchives\)|No. Exceptions occur.|
|DistributedCache|createAllSymlink\(Configuration conf, File jobCacheDir, File workDir\)|No. Exceptions occur.|
|DistributedCache|createSymlink\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getArchiveClassPaths\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getArchiveTimestamps\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getCacheArchives\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getCacheFiles\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getFileClassPaths\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getFileStatus\(Configuration conf, URI cache\)|No. Exceptions occur.|
|DistributedCache|getFileTimestamps\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getLocalCacheArchives\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getLocalCacheFiles\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getSymlink\(Configuration conf\)|No. Exceptions occur.|
|DistributedCache|getTimestamp\(Configuration conf, URI cache\)|No. Exceptions occur.|
|DistributedCache|setArchiveTimestamps\(Configuration conf, String timestamps\)|No. Exceptions occur.|
|DistributedCache|setCacheArchives\(URI\[\] archives, Configuration conf\)|No. Exceptions occur.|
|DistributedCache|setCacheFiles\(URI\[\] files, Configuration conf\)|No. Exceptions occur.|
|DistributedCache|setFileTimestamps\(Configuration conf, String timestamps\)|No. Exceptions occur.|
|DistributedCache|setLocalArchives\(Configuration conf, String str\)|No. Exceptions occur.|
|DistributedCache|setLocalFiles\(Configuration conf, String str\)|No. Exceptions occur.|
|IsolationRunner|N/A|No. Exceptions occur.|
|Profiling|N/A|Empty implementation.|
|Debugging|N/A|Empty implementation.|
|Data Compression|N/A|Yes.|
|Skipping Bad Records|N/A|No. Exceptions occur.|
|Job Authorization|mapred.acls.enabled|No. Exceptions occur.|
|Job Authorization|mapreduce.job.acl-view-job|No. Exceptions occur.|
|Job Authorization|mapreduce.job.acl-modify-job|No. Exceptions occur.|
|Job Authorization|mapreduce.cluster.administrators|No. Exceptions occur.|
|Job Authorization|mapred.queue.queue-name.acl-administer-jobs|No. Exceptions occur.|
|MultipleInputs|N/A|No. Exceptions occur.|
|Multi\{anchor:\_GoBack\}pleOutputs|N/A|Yes.|
|org.apache.hadoop.mapreduce.lib.db|N/A|No. Exceptions occur.|
|org.apache.hadoop.mapreduce.security|N/A|No. Exceptions occur.|
|org.apache.hadoop.mapreduce.lib.jobcontrol|N/A|No. Exceptions occur.|
|org.apache.hadoop.mapreduce.lib.chain|N/A|No. Exceptions occur.|
|org.apache.hadoop.mapreduce.lib.db|N/A|No. Exceptions occur.|

