---
keyword: [task instance, instance status, ]
---

# Task instance

This topic describes task instances in MaxCompute and the status that a task instance may have throughout its lifecycle.

When you start to run SQL, Spark, or MapReduce tasks in MaxCompute, the tasks are instantiated as instances. The lifecycle of task instances includes two stages: Running and Terminated.

Task instances that are in the Running stage are in the Running state. Task instances that are in the Terminated stage can be in the Success, Failed, or Canceled state. You can query or change the status of a task instance based on the ID that MaxCompute assigns to the task instance. Examples:

```
-- Query the status of a task instance.
status *instance\_id*;
-- Terminate a task instance. After you run the kill command, the status of the task instance changes to Canceled.
kill *instance\_id*; 
-- View the operational logs of a task instance.
wait *instance\_id*; 
```

In the preceding commands, instance\_id indicates the ID of the task instance that you want to manage. Replace this parameter with the actual instance ID.

