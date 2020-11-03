---
keyword: [task, Task]
---

# Task

A task is the basic computing unit of MaxCompute. Computing jobs such as those involving SQL and MapReduce are completed by using tasks.

MaxCompute parses most of the tasks that you submit, especially computing tasks such as [SQL DML statements](/intl.en-US/Development/SQL/Insert Operation/INSERT OVERWRITE and INSERT INTO.md) and [MapReduce](/intl.en-US/Development/MapReduce/Summary/MapReduce.md) tasks. Then, MaxCompute generates task execution plans based on the parsing results. An execution plan consists of several mutually dependent stages.

The execution plan can be logically defined as a directed graph. In this graph, vertices represent stages, and edges represent dependencies between the stages. MaxCompute executes the stages based on the dependencies in the graph \(execution plan\). A single stage comprises multiple threads, also known as workers. The workers in each stage work together to complete computing for the stage. Different workers in the same stage process different data but run on the same execution logic. A computing task is instantiated as an instance when it is executed. You can perform operations on the instance. For example, you can run the [status](/intl.en-US/Development/Common commands/Instance operations.md) command to query the instance status and the [kill](/intl.en-US/Development/Common commands/Instance operations.md) command to terminate the instance.

Some MaxCompute tasks, such as SQL DDL statements, are not computing tasks. These tasks only need to read and modify metadata in MaxCompute. MaxCompute does not generate execution plans for these tasks.

**Note:** MaxCompute does not convert all the requests to tasks. For example, operations on [projects](/intl.en-US/Product Introduction/Definitions/Project.md), [resources](/intl.en-US/Product Introduction/Definitions/Resource.md), [user-defined functions \(UDFs\)](/intl.en-US/Product Introduction/Definitions/Function.md), and [instances](/intl.en-US/Product Introduction/Definitions/Instance.md) are not processed as tasks.

