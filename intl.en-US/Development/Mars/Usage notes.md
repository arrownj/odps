---
keyword: [Mars cluster operation, read/write operations on MaxCompute tables]
---

# Usage notes

This topic describes how to perform operations on Mars clusters, read and write MaxCompute tables, and obtain the URLs of Mars UI, Logview, and Jupyter Notebook.

For more information about how to develop Mars jobs, see [Mars](https://pyodps.readthedocs.io/zh_CN/latest/mars.html).

## Mars cluster operations

-   Create a Mars cluster

    Run the following commands to create a Mars cluster. This process takes a while to complete.

    ```
    from odps import options
    options.verbose = True  
    # If the preceding commands have been configured in the DataWorks PyODPS 3 node, you do not need to run the two commands.
    client = o.create_mars_cluster(5, 4, 16, min_worker_num=3)
    ```

    Parameter description:

    -   5: the number of worker nodes in the cluster. In this example, the cluster consists of five worker nodes.
    -   4: the number of CPU cores for each worker node. In this example, each worker node has four CPU cores.
    -   16: the memory size of each worker node. In this example, each worker node has 16 GB of memory.

        **Note:**

        -   The memory size that you request for each worker node must be greater than 1 GB. The optimal ratio of CPU cores to the memory size is 1:4. For example, configure a worker node with 4 CPU cores and 16 GB of memory.
        -   You can create a maximum of 30 worker nodes. If the number of worker nodes exceeds the upper limit, the image server may be overloaded. If you want to create more than 30 worker nodes,[submit a ticket](https://workorder-intl.console.aliyun.com/).
    -   min\_worker\_num: the minimum number of worker nodes that must be started for the system to return a client object. If this parameter is set to 3, the system returns a client object after all of the three worker nodes are started.
    If you set `options.verbose` to True when you create a Mars cluster, the URLs of Logview, Mars UI, and Jupyter Notebook of the MaxCompute instance are displayed in the command output. You can use the Mars UI to connect to Mars clusters and query the status of clusters and jobs.

-   Submit a job

    When you create a Mars cluster, the cluster creates a default session that connects to the cluster. You can call the `execute()` method to submit a job to the cluster and run the job in the default session.

    ```
    import mars.dataframe as md
    import mars.tensor as mt
    
    md.DataFrame(mt.random.rand(10, 3)).execute()  # Call the execute() method to submit the job to the created cluster.
    ```

-   Stop and release a cluster

    A Mars cluster is automatically released three days after it is created. If you do not need a Mars cluster any longer, you can call the `client.stop_server()` method to release the cluster.

    ```
    client.stop_server()
    ```


## Read/write operations on MaxCompute tables

Mars can directly read and write MaxCompute tables.

-   Read MaxCompute tables

    Mars calls the `o.to_mars_dataframe` method to read a MaxCompute table and returns a [Mars DataFrame](https://yq.aliyun.com/go/articleRenderRedirect?url=https%3A%2F%2Fdocs.pymars.org%2Fzh_CN%2Flatest%2Fdataframe%2Findex.html).

    ```
    In [1]: df = o.to_mars_dataframe('test_mars')
    In [2]: df.head(6).execute()
    Out[2]:
           col1  col2
    0        0    0
    1        0    1
    2        0    2
    3        1    0
    4        1    1
    5        1    2
    ```

-   Write MaxCompute tables

    Mars calls the `o.persist_mars_dataframe(df, 'table_name')` method to save a Mars DataFrame as a MaxCompute table.

    ```
    In [3]: df = o.to_mars_dataframe('test_mars')
    In [4]: df2 = df + 1
    In [5]: o.persist_mars_dataframe(df2, 'test_mars_persist')  # Save the Mars DataFrame as a MaxCompute table.
    In [6]: o.get_table('test_mars_persist').to_df().head(6)  # Query data by using the PyODPS DataFrame API.
           col1  col2
    0        1    1
    1        1    2
    2        1    3
    3        2    1
    4        2    2
    5        2    3
    ```

-   Use the Jupyter Notebook of a Mars cluster

    When you create a Mars cluster, the cluster creates a Jupyter Notebook for writing code. The new Jupyter Notebook creates a session and submits a job to the Mars cluster.

    ```
    import mars.dataframe as md
    md.DataFrame(mt.random.rand(10, 3)).sum().execute() # Call the execute() method in the Jupyter Notebook to submit the job to the current cluster. You do not need to manually create a session in the Jupyter Notebook.
    ```

    **Note:**

    -   The Jupyter Notebook is not automatically saved. We recommend that you manually save the Jupyter Notebook as required.
    -   You can connect your Jupyter Notebook to an existing Mars cluster. For more information, see [Use an existing Mars cluster](#section_mxb_4df_rm2).

## Other operations

-   Use an existing Mars cluster
    -   Recreate an existing Mars cluster based on the instance ID.

        ```
        client = o.create_mars_cluster(instance_id=**instance-id**)
        ```

    -   To use an existing Mars cluster, create a Mars session to visit the Mars UI URL.

        ```
        from mars.session import new_session
        new_session('**Mars UI address**').as_default() # Set the session as default.
        ```

-   Obtain the URL of the Mars UI

    If you set `options.verbose` to True when you create a Mars cluster, the URL of the Mars UI is automatically displayed in the command output. You can use `client.endpoint` to obtain the URL of the Mars UI.

    ```
    print(client.endpoint)
    ```

-   Obtain the Logview URL of an instance

    If you set `options.verbose` to True when you create a Mars cluster, the Logview URL is automatically displayed in the command output. You can also use `client.get_logview_address()` to obtain the Logview URL.

    ```
    print(client.get_logview_address())
    ```

-   Obtain the Jupyter Notebook URL

    `options.verbose` is set to True when a Mars cluster is created. Then, the cluster automatically prints the address of Jupyter Notebook. You can also use `client.get_notebook_endpoint()` to obtain the Jupyter Notebook URL.

    ```
    print(client.get_notebook_endpoint())
    ```


