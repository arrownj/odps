---
keyword: partition pruning
---

# Check whether partition pruning is effective

This topic describes how to check whether partition pruning is effective.

## Background information

A MaxCompute partitioned table is a table with partitions. You can specify one or more columns as the partition key to create a partitioned table. If you have specified the name of a partition that you want to access, MaxCompute reads data only from that partition and does not scan the entire table. This reduces costs and improves efficiency.

Partition pruning allows you to specify filter conditions for partition key columns. This way, MaxCompute reads data only from the partitions that meet the filter conditions that you have specified in SQL statements. This avoids the errors and waste of resources that are caused by full table scans. However, partition pruning may not take effect sometimes.

This topic describes partition pruning from the following aspects:

-   Check whether partition pruning is effective
-   Scenarios where partition pruning does not take effect

## Check whether partition pruning is effective

To check whether partition pruning is effective for a query, execute the EXPLAIN statement to view the execution plan of the query.

-   For a query where partition pruning does not take effect:

    ```
    explain
    select seller_id
    from xxxxx_trd_slr_ord_1d
    where ds=rand();
    ```

    The execution plan indicates that all the 1,344 partitions of Table xxxxx\_trd\_slr\_ord\_1d are read.

-   For a query where partition pruning is effective:

    ```
    explain
    select seller_id
    from xxxxx_trd_slr_ord_1d
    where ds='20150801';
    ```

    ![Partition pruning](../images/p99318.png)

    The execution plan indicates that only Partition 20150801 of Table xxxxx\_trd\_slr\_ord\_1d is read.


## Scenarios where partition pruning does not take effect

-   Improper use of UDFs

    If you use user-defined functions \(UDFs\) or specific built-in functions to specify partitions, partition pruning may not take effect. In this case, we recommend that you execute the EXPLAIN statement to check whether partition pruning is effective.

    ```
    explain
    select ...
    from xxxxx_base2_brd_ind_cw
    where ds = concat(SPLIT_PART(bi_week_dim(' ${bdp.system.bizdate}'), ',', 1), SPLIT_PART(bi_week_dim(' ${bdp.system.bizdate}'), ',', 2))
    ```

    **Note:** For more information about UDF-based partition pruning, see the "WHERE" section in [WHERE](/intl.en-US/Development/SQL/Select Operation/SELECT syntax.md).

-   Improper use of joins

    When you join tables, pay attention to the following rules:

    -   If partition pruning conditions are specified in the WHERE clause, partition pruning is effective.
    -   If partition pruning conditions are specified in the ON clause, partition pruning is effective for the secondary table, but not the primary table.
    The following examples describe how partition pruning works when three different types of JOIN operations are performed:

    -   LEFT OUTER JOIN
        -   For a query where partition pruning conditions are specified in the ON clause:

            ```
            set odps.sql.allow.fullscan=true;
            explain
            select a.seller_id
                ,a.pay_ord_pbt_1d_001
            from xxxxx_trd_slr_ord_1d a
            left outer join
                 xxxxx_seller b
            on a.seller_id=b.user_id
            and a.ds='20150801'
            and b.ds='20150801';
            ```

            ![**](../images/p99324.png)

            The execution plan indicates that partition pruning is effective for the right table, but not the left table.

        -   For a query where partition pruning conditions are specified in the WHERE clause:

            ```
            set odps.sql.allow.fullscan=true;
            explain
            select a.seller_id
                ,a.pay_ord_pbt_1d_001
            from xxxxx_trd_slr_ord_1d a
            left outer join
                xxxxx_seller b
            on a.seller_id=b.user_id
            where a.ds='20150801'
            and b.ds='20150801';
            ```

            ![&&](../images/p99327.png)

            The execution plan indicates that partition pruning is effective for both tables.

    -   RIGHT OUTER JOIN

        A RIGHT OUTER JOIN operation is similar to a LEFT OUTER JOIN operation. If partition pruning conditions are specified in the ON clause, partition pruning is effective only for the left table, but not the right table. If partition pruning conditions are specified in the WHERE clause, partition pruning is effective for both tables.

    -   FULL OUTER JOIN

        Partition pruning is effective only when partition pruning conditions are specified in the WHERE clause, but not the ON clause.


## Impact and consideration

-   If partition pruning does not take effect, the query performance can be greatly deteriorated. This issue can be hardly discovered. We recommend that you check whether partition pruning is effective before you commit the code.
-   To use UDFs for partition pruning, you must modify the classes of the UDFs or add `set odps.sql.udf.ppr.deterministic = true;` before the SQL statements to execute. For more information, see [WHERE](/intl.en-US/Development/SQL/Select Operation/SELECT syntax.md).

