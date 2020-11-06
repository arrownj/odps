# EXPLAIN

This topic describes the EXPLAIN statement in MaxCompute SQL. This statement shows the structure of the execution plan of a DML statement. An execution plan is a program that executes SQL statements.

## Syntax

Syntax of the EXPLAIN statement:

```
EXPLAIN <DML query>;
```

## Description

The execution results of an EXPLAIN statement include:

-   Dependencies among all the jobs of this DML statement
-   Dependencies among all the tasks of each job
-   Dependencies among all operators in a task

**Note:** If a query statement is complex, the EXPLAIN statement generates excessive results. As a result, API limits are reached and complete results cannot be obtained. In this case, you can split a query and execute the EXPLAIN statement on each subquery to understand the structure of the job.

## Examples

Example:

```
EXPLAIN
SELECT abs(a.key), b.value FROM src a JOIN src1 b ON a.value = b.value;
```

The output of the `EXPLAIN` statement consists of the following three parts:

-   Job dependencies: `job0 is root job`. This query requires only one job \(`job0`\). Therefore, only one row of information is displayed.
-   Task dependencies. Example:

    ```
    In Job job0:
    root Tasks: M1_Stg1, M2_Stg1
    J3_1_2_Stg1 depends on: M1_Stg1, M2_Stg1
    ```

    `job0` contains three tasks, `M1_Stg1`, `M2_Stg1`, and `J3_1_2_Stg1`. The `J3_1_2_Stg1` task is executed after the `M1_Stg1` and `M2_Stg1` tasks are completed.

    Rules for naming a task:

    -   MaxCompute provides four task types: map, reduce, join, and local work. The first letter in a task name indicates the type of the task. For example, `M2Stg1` is a map task.
    -   The number following the first letter indicates the task ID. This ID is unique among all tasks that correspond to the current query.
    -   Digits that are combined by underscores \(\_\) indicate the task dependencies. For example, `J3_1_2_Stg1` indicates that the current task with the ID of 3 depends on two tasks whose IDs are 1 and 2.
-   Operator structure, where each operator string describes the execution semantics of a task.

    ```
    In Task M1_Stg1:
      Data source: yudi_2.src                       # Data source describes the input of the task.
      TS: alias: a                                  # TableScanOperator
          RS: order: +                              # ReduceSinkOperator
              keys:
                   a.value
              values:
                   a.key
              partitions:
                   a.value
    In Task J3_1_2_Stg1:
      JOIN: a INNER JOIN b                          # JoinOperator
          SEL: Abs(UDFToDouble(a._col0)), b._col5   # SelectOperator
              FS: output: None                      # FileSinkOperator
    In Task M2_Stg1:
      Data source: yudi_2.src1
      TS: alias: b
          RS: order: +
              keys:
                   b.value
              values:
                   b.value
              partitions:
                   b.value
    ```

    Operator descriptions:

    -   TableScanOperator: describes the logic of `FROM` statement blocks in a query statement. The alias of the input table is displayed in the `EXPLAIN` results.
    -   SelectOperator: describes the logic of `SELECT` statement blocks in a query statement. The columns that are transferred to the next operator are displayed in the `EXPLAIN` results. Separate multiple columns with commas \(,\).
        -   If columns are referenced, the value is in the `< alias >.< column_name >` format.
        -   If the result of an expression is transferred, the value is displayed as a function, such as `func1(arg1_1, arg1_2, func2(arg2_1, arg2_2))`.
        -   If constants are transferred, the value is immediately displayed.
    -   FilterOperator: describes the logic of `WHERE` statement blocks in a query statement. The `EXPLAIN` results include a `WHERE` clause, with the display rules that are similar to those of SelectOperator.
    -   JoinOperator: describes the logic of `JOIN` statement blocks in a query statement. The `EXPLAIN` results show which tables are joined in which way.
    -   GroupByOperator: describes the logic of aggregate operations. This operator is displayed if an aggregate function is used in a query. The content of the aggregate function is displayed in the `EXPLAIN` results.
    -   ReduceSinkOperator: describes the logic of data distribution between tasks. If the result of the current task is transferred to another task, ReduceSinkOperator must be used to distribute data at the last stage of the current task. The sorting method of output data records, the distributed keys and values, and columns used to calculate the hash value are displayed in the `EXPLAIN` results.
    -   FileSinkOperator: describes the logic of storage operations on final data records. If an `INSERT` statement block exists in a query, the name of the required table is displayed in the `EXPLAIN` results.
    -   LimitOperator: describes the logic of `LIMIT` statement blocks in a query statement. The number of records specified by `LIMIT` is displayed in the `EXPLAIN` results.
    -   MapjoinOperator: describes `JOIN` operations on large tables. This operator is similar to JoinOperator.

