---
keyword: SDK for Python
---

# SDK for Python

This topic describes how to use MaxCompute SDK for Python to create, view, and delete a table. MaxCompute SDK for Python is also called PyODPS.

## Obtain a PyODPS project

A project is a basic operation unit in MaxCompute. For more information, see [Project](/intl.en-US/Product Introduction/Definitions/Project.md).

You can perform the following basic operations for a project:

-   Obtain a project: You can use the `get_project` method of a MaxCompute entry object to obtain a project. The PyODPS node in DataWorks contains a global variable `odps` or `o`, which is the MaxCompute entry. You do not need to manually define the MaxCompute entry.

    ```
    project = o.get_project('project_name')  # If you specify a project, data of the specified project is returned.
    project = o.get_project()              # If you do not specify a project, data of the current project is returned.
    ```

    project\_name is required when you use the get\_project method.

-   Verify that the project exists: Use the `exist_project` method to check whether the project exists.

## Perform basic operations on tables

You can perform the following basic operations on tables:

-   Use the `list_tables` method of the entry object to query all tables in a project.

    ```
    # Query all tables in a project.
    for table in odps.list_tables():
    ```

-   Use the `exist_table` method of the entry object to check whether a specific table exists. Then, use the `get_table` method to obtain the table.

    ```
    t = odps.get_table('table_name')
    t.schema
    odps.Schema {
      c_int_a                 bigint
      c_int_b                 bigint
      c_double_a              double
      c_double_b              double
      c_string_a              string
      c_string_b              string
      c_bool_a                boolean
      c_bool_b                boolean
      c_datetime_a            datetime
      c_datetime_b            datetime
    }
    t.lifecycle
    -1
    print(t.creation_time)
    2014-05-15 14:58:43
    t.is_virtual_view
    False
    t.size
    1408
    t.schema.columns
    [<column c_int_a, type bigint>,
     <column c_int_b, type bigint>,
     <column c_double_a, type double>,
     <column c_double_b, type double>,
     <column c_string_a, type string>,
     <column c_string_b, type string>,
     <column c_bool_a, type boolean>,
     <column c_bool_b, type boolean>,
     <column c_datetime_a, type datetime>,
     <column c_datetime_b, type datetime>]            
    ```


## Create a table schema

You can use one of the following methods to create a table schema:

-   Create a schema based on table columns and optional partitions.

    ```
    from odps.models import Schema, Column, Partition
    columns = [Column(name='num', type='bigint', comment='the column'),
               Column(name='num2', type='double', comment='the column2')]
    partitions = [Partition(name='pt', type='string', comment='the partition')]
    schema = Schema(columns=columns, partitions=partitions)
    schema.columns
    [<column num, type bigint>,
     <column num2, type double>,
     <partition pt, type string>]
    schema.partitions
    [<partition pt, type string>]
    schema.names  # Obtain the names of non-partitioned fields.
    ['num', 'num2']
    schema.types  # Obtain the data types of non-partitioned fields.
    [bigint, double]
    ```

-   Create a schema by calling the `Schema.from_lists()` method. This method is more convenient, but you cannot directly set comments for columns and partitions.

    ```
    schema = Schema.from_lists(['num', 'num2'], ['bigint', 'double'], ['pt'], ['string'])
    schema.columns
    [<column num, type bigint>,
     <column num2, type double>,
     <partition pt, type string>]
    ```


## Create a table

You can call the `create_table()` method to create a table by using one of the following methods:

-   Use a table schema to create a table.

    ```
    table = o.create_table('my_new_table', schema)
    table = o.create_table('my_new_table', schema, if_not_exists=True)  # Create the table only if no table with the same name exists.
    table = o.create_table('my_new_table', schema, lifecycle=7)  # Set the lifecycle of the table.
    ```

-   Create a table by specifying the names and data types of the fields to be contained in the table.

    ```
    # Create a non-partitioned table.
    table = o.create_table('my_new_table', 'num bigint, num2 double', if_not_exists=True)
    # Create a partitioned table with the specified table columns and partition fields.
    table = o.create_table('my_new_table', ('num bigint, num2 double', 'pt string'), if_not_exists=True)
    ```

    By default, when you create a table, you can only use the BIGINT, DOUBLE, DECIMAL, STRING, DATETIME, BOOLEAN, MAP, and ARRAY data types. If you need to use other data types such as TINYINT and STRUCT, you must set `options.sql.use_odps2_extension` to True. Example:

    ```
    from odps import options
    options.sql.use_odps2_extension = True
    table = o.create_table('my_new_table', 'cat smallint, content struct<title:varchar(100), body:string>')
    ```


## Delete a table

You can call the `delete_table()` method to delete an existing table.

```
o.delete_table('my_table_name', if_exists=True)  # Delete a table only if the table exists.
 t.drop()  # Call the drop() method to delete a table if the table exists.
```

## Manage table partitions

-   Check whether a table is partitioned.

    ```
    if table.schema.partitions:
     print('Table %s is partitioned.' % table.name)
    ```

-   Iterate over all the partitions in a table.

    ```
    for partition in table.partitions:
         print(partition.name)
    for partition in table.iterate_partitions(spec='pt=test'):
    # Iterate over level-2 partitions.
    ```

-   Check whether a partition exists.

    ```
    table.exist_partition('pt=test,sub=2015')
    ```

-   Obtain a partition.

    ```
    partition = table.get_partition('pt=test')
     print(partition.creation_time)
    2015-11-18 22:22:27
     partition.size
    0
    ```

-   Create a partition.

    ```
    t.create_partition('pt=test', if_not_exists=True)  # Create a partition only if no partition with the same name exists.
    ```

-   Delete a partition.

    ```
    t.delete_partition('pt=test', if_exists=True)  # Delete a partition only if the partition exists.
    partition.drop()  # Call the drop() method to delete a partition if the partition exists.
    ```


## PyODPS SQL

PyODPS supports MaxCompute SQL queries and provides methods to obtain query results.

-   Execute SQL statements.

    You can use the `execute_sql('statement')` and `run_sql('statement')` methods of the entry object to execute SQL statements. For more information about return values, see [Task instance](/intl.en-US/Development/PyODPS/Basic operations/Task instances.md).

    ```
    odps.execute_sql('select * from table_name')  # Execute SQL statements in synchronous mode. Threads are blocked until the execution of the SQL statements is completed.
    instance = odps.run_sql('select * from table_name')  # Execute SQL statements in asynchronous mode.
    instance.wait_for_success() # Threads are blocked until the execution of the SQL statements is completed.
    ```

-   Obtain SQL query results.

    You can perform the `open_reader` operation on the instance on which SQL statements are executed to read the SQL query results. When the query results are being read, the following situations may occur:

    -   The SQL statements return structured data.

        ```
        with o.execute_sql('select * from table_name').open_reader() as reader:
            for record in reader:
            # Process each record.
        ```

    -   If the `DESC` command is executed, you can use `reader.raw` to obtain the raw SQL query results.

        ```
        with o.execute_sql('desc table_name').open_reader() as reader:
        print(reader.raw)
        ```


## PyODPS resources

Typically, MaxCompute resources are used in UDFs and MapReduce. You can use the following methods to perform basic operations on the resources:

-   `list_resources`: lists all resources in a project.
-   `exist_resource`: checks whether a resource exists.
-   `delete_resource`: deletes a resource. You can also use the Resource object to call the `drop` method to delete a resource.
-   `create_resource`: creates a resource.
-   `open_resource`: reads a resource.

PyODPS supports two types of resources: file and table.

-   File resources

    File resources include `FILE`, `PY`, `JAR`, and `ARCHIVE` files.

    Common operations on file resources:

    -   Create a file resource

        You can specify a resource name, file type, and file-like object or string to call the `create_resource` method to create a file resource.

        ```
        resource = o.create_resource('test_file_resource', 'file', file_obj=open('/to/path/file'))  # Use a file-like object to create a file resource.
        resource = o.create_resource('test_py_resource', 'py', file_obj='import this')  # Use a string to create a file resource.
        ```

    -   Read and modify a file resource

        You can use one of the following methods to open a resource:

        -   Call the `open` method for a file resource to open it.
        -   Call the `open_resource` method at the MaxCompute entry to open a file resource.
        The opened object is a file-like object. The opening modes of file resources are similar to the `open` method predefined in Python. The following example demonstrates the opening modes of file resources:

        ```
        with resource.open('r') as fp:  # Open the specified file in read mode.
             content = fp.read()  # Read all content.
             fp.seek(0)  # Return to the beginning of the file.
             lines = fp.readlines()  # Read multiple lines.
             fp.write('Hello World')  # Error. Data cannot be written to a file in read mode.
        
         with o.open_resource('test_file_resource', mode='r+') as fp:  # Open the file in read/write mode.
             fp.read()
             fp.tell()  # Locate the current position.
             fp.seek(10)
             fp.truncate()  # Truncate the file to the specified length.
             fp.writelines(['Hello\n', 'World\n'])  # Write multiple lines to the file.
             fp.write('Hello World')
             fp.flush()  # Manually call the method to submit the update to MaxCompute.
        ```

        PyODPS supports the following opening modes:

        -   `r`: read mode. The file can be opened but data cannot be written to it.
        -   `w`: write mode. Data can be written to the file, but data in the file cannot be read. If a file is opened in write mode, the file content is cleared first.
        -   `a`: append mode. Data can be added to the end of the file.
        -   `r+`: read/write mode. You can read data from and write data to the file.
        -   `w+`: This mode is similar to the `r+` mode. The only difference is that the file content is cleared first.
        -   `a+`: This mode is similar to the `r+` mode. The only difference is that data can be written only to the end of the file.
        PyODPS also supports the following binary opening modes for some file resources, such as compressed files:

        -   `rb`: binary read mode.
        -   `r+b`: binary read/write mode.
-   Table resources
    -   Create a table resource

        ```
        o.create_resource('test_table_resource', 'table', table_name='my_table', partition='pt=test')
        ```

    -   Update a table resource

        ```
        table_resource = o.get_resource('test_table_resource')
        table_resource.update(partition='pt=test2', project_name='my_project2')
        ```

    -   Obtain a table and a partition

        ```
        table_resource = o.get_resource('test_table_resource')
        table = table_resource.table
        print(table.name)
        partition = table_resource.partition
        print(partition.spec)
        ```

    -   Read and write data

        ```
        table_resource = o.get_resource('test_table_resource')
        with table_resource.open_writer() as writer:
            writer.write([0, 'aaaa'])
            writer.write([1, 'bbbbb'])
            with table_resource.open_reader() as reader:
                for rec in reader:
                    print(rec)
        ```


## DataFrame

PyODPS provides a pandas-like API called PyODPS DataFrame. This API makes full use of the computing power of MaxCompute. For more information, see [DataFrame](https://pyodps.readthedocs.io/en/latest/df.html).

Assume that the following tables exist: `pyodps_ml_100k_movies` that contains movie-related data, `pyodps_ml_100k_users` that contains user-related data, and `pyodps_ml_100k_ratings` that contains rating-related data.

1.  Create an entry object of MaxCompute.

    ```
    # Create an entry object of MaxCompute.
    o = ODPS('**your-access-id**', '**your-secret-access-key**',project='**your-project**', endpoint='**your-end-point**'))
    ```

2.  Call a table object and create the DataFrame object named users.

    ```
    from odps.df import DataFrame
    users = DataFrame(o.get_table('pyodps_ml_100k_users'))
    ```

3.  Perform the following operations on the DataFrame object:
    -   View the fields of DataFrame and the types of these fields based on the dtypes attribute.

        ```
        users.dtypes
        ```

    -   Use the head method to obtain the first N data records for a quick data preview.

        ```
        users.head(10)
        ```

        The following table lists the returned results.

        |-|user\_id|age|sex|occupation|zip\_code|
        |:-|:-------|:--|:--|:---------|:--------|
        |0|1|24|M|technician|85711|
        |1|2|53|F|other|94043|
        |2|3|23|M|writer|32067|
        |3|4|24|M|technician|43537|
        |4|5|33|F|other|15213|
        |5|6|42|M|executive|98101|
        |6|7|57|M|administrator|91344|
        |7|8|36|M|administrator|05201|
        |8|9|29|M|student|01002|
        |9|10|53|M|lawyer|90703|

    -   Filter fields.
        -   Filter some fields.

            ```
            users[['user_id', 'age']].head(5)
            ```

            The following table lists the returned results.

            |-|user\_id|age|
            |:-|:-------|:--|
            |0|1|24|
            |1|2|53|
            |2|3|23|
            |3|4|24|
            |4|5|33|

        -   Exclude some fields. Example:

            ```
            >>> users.exclude('zip_code', 'age').head(5)
            ```

            The following table lists the returned results.

            |-|user\_id|sex|occupation|
            |:-|:-------|:--|:---------|
            |0|1|M|technician|
            |1|2|F|other|
            |2|3|M|writer|
            |3|4|M|technician|
            |4|5|F|other|

        -   When you exclude some fields, you may want to obtain new columns based on computation. For example, add the sex\_bool attribute and set it to True if sex is M. Otherwise, set the sex\_bool attribute to False. Example:

            ```
            >>> users.select(users.exclude('zip_code', 'sex'), sex_bool=users.sex == 'M').head(5)
            ```

            The following table lists the returned results.

            |-|user\_id|age|occupation|sex\_bool|
            |:-|:-------|:--|:---------|:--------|
            |0|1|24|technician|True|
            |1|2|53|other|False|
            |2|3|23|writer|True|
            |3|4|24|technician|True|
            |4|5|33|other|False|

    -   Query the number of users aged from 20 to 25. Example:

        ```
        >>> users.age.between(20, 25).count().rename('count')
        943
        ```

    -   Query the numbers of male and female users.

        ```
        >>> users.groupby(users.sex).count()
        ```

        The following table lists the returned results.

        |-|sex|count|
        |:-|:--|:----|
        |0|F|273|
        |1|M|670|

    -   To divide users by occupation, obtain the top 10 occupations with the most population, and sort the occupations in descending order.

        ```
        >>> df = users.groupby('occupation').agg(count=users['occupation'].count())
        >>> df.sort(df['count'], ascending=False)[:10]
        ```

        The following table lists the returned results.

        |-|occupation|count|
        |:-|:---------|:----|
        |0|student|196|
        |1|other|105|
        |2|educator|95|
        |3|administrator|79|
        |4|engineer|67|
        |5|programmer|66|
        |6|librarian|51|
        |7|writer|45|
        |8|executive|32|
        |9|scientist|31|

        The DataFrame API provides the value\_counts method for the same purpose.

        ```
        >>> users.occupation.value_counts()[:10]
        ```

        The following table lists the returned results.

        |-|occupation|count|
        |:-|:---------|:----|
        |0|student|196|
        |1|other|105|
        |2|educator|95|
        |3|administrator|79|
        |4|engineer|67|
        |5|programmer|66|
        |6|librarian|51|
        |7|writer|45|
        |8|executive|32|
        |9|scientist|31|

    -   Show data in a more intuitive graph.

        ```
         %matplotlib inline
        ```

    -   Visualize data on a horizontal bar chart.

        ```
        users['occupation'].value_counts().plot(kind='barh', x='occupation', ylabel='prefession')
        ```

        ![Horizontal bar chart](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/9378559951/p2854.png)

    -   Visualize data on a histogram. Divide users into 30 groups by age and view the histogram of age distribution. Example:

        ```
        >>> users.age.hist(bins=30, title="Distribution of users' ages", xlabel='age', ylabel='count of users')
        ```

        ![Histogram](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0478559951/p2855.png)

    -   Join three tables to obtain a new table and save it.

        ```
        movies = DataFrame(o.get_table('pyodps_ml_100k_movies'))
        ratings = DataFrame(o.get_table('pyodps_ml_100k_ratings'))
        o.delete_table('pyodps_ml_100k_lens', if_exists=True)
        lens = movies.join(ratings).join(users).persist('pyodps_ml_100k_lens')
        lens.dtypes
        ```

        In this example, the following information is returned:

        ```
        odps.Schema {
          movie_id                            int64
          title                               string
          release_date                        string
          video_release_date                  string
          imdb_url                            string
          user_id                             int64
          rating                              int64
          unix_timestamp                      int64
          age                                 int64
          sex                                 string
          occupation                          string
          zip_code                            string
        }
        ```

    -   Divide ages from 0 to 80 into eight age groups.

        ```
        labels = ['0-9', '10-19', '20-29', '30-39', '40-49', '50-59', '60-69', '70-79']
        cut_lens = lens[lens, lens.age.cut(range(0, 81, 10), right=False, labels=labels).rename('age_group')]
        ```

    -   View the first 10 ages with only one user.

        ```
        cut_lens['age_group', 'age'].distinct()[:10]
        ```

        The following table lists the returned results.

        |-|age\_group|age|
        |:-|:---------|:--|
        |0|0-9|7|
        |1|10-19|10|
        |2|10-19|11|
        |3|10-19|13|
        |4|10-19|14|
        |5|10-19|15|
        |6|10-19|16|
        |7|10-19|17|
        |8|10-19|18|
        |9|10-19|19|

    -   View the total rating and average rating of users in each age group. Example:

        ```
        cut_lens.groupby('age_group').agg(cut_lens.rating.count().rename('total_rating'), cut_lens.rating.mean().rename('average_rating'))
        ```

        The following table lists the returned results.

        |-|age\_group|average\_rating|total\_rating|
        |:-|:---------|:--------------|:------------|
        |0|0-9|3.767442|43|
        |1|10-19|3.486126|8181|
        |2|20-29|3.467333|39535|
        |3|30-39|3.554444|25696|
        |4|40-49|3.591772|15021|
        |5|50-59|3.635800|8704|
        |6|60-69|3.648875|2623|
        |7|70-79|3.649746|197|


## Configuration

PyODPS provides a series of configuration options, which can be obtained by using `odps.options` The following tables describe configurable MaxCompute options.

-   General configurations

    |Option|Description|Default value|
    |:-----|:----------|:------------|
    |end\_point|MaxCompute Endpoint|None|
    |default\_project|The default project.|None|
    |log\_view\_host|The Logview host.|None|
    |log\_view\_hours|The retention time of Logview. Unit: hours.|24|
    |local\_timezone|The time zone used. True indicates the local time, and False indicates UTC. The time zone of pytz can also be used.|1|
    |lifecycle|The lifecycle of all tables.|None|
    |temp\_lifecycle|The lifecycle of temporary tables.|1|
    |biz\_id|The ID of the user.|None|
    |verbose|Specifies whether to print logs.|False|
    |verbose\_log|The log receiver.|None|
    |chunk\_size|The size of the write buffer.|1496|
    |retry\_times|The maximum number of request retries.|4|
    |pool\_connections|The number of cached connections in the connection pool.|10|
    |pool\_maxsize|The maximum capacity of the connection pool.|10|
    |connect\_timeout|The time to wait before the connection times out.|5|
    |read\_timeout|The time to wait before the read operation times out.|120|
    |completion\_size|The maximum number of object completion listing items.|10|
    |notebook\_repr\_widget|Specifies whether to use interactive graphs.|True|
    |sql.settings|MaxCompute SQL runs global hints.|None|
    |sql.use\_odps2\_extension|Specifies whether to enable MaxCompute 2.0 language extension.|False|

-   Data upload or download configurations

    |Option|Description|Default value|
    |:-----|:----------|:------------|
    |tunnel.endpoint|Tunnel Endpoint|None|
    |tunnel.use\_instance\_tunnel|Specifies whether to use Instance Tunnel to obtain execution results.|True|
    |tunnel.limited\_instance\_tunnel|Specifies whether to limit the number of data records obtained by using Instance Tunnel.|True|
    |tunnel.string\_as\_binary|Specifies whether to use bytes instead of unicode for data of the STRING type.|False|

-   DataFrame configuration

    |Option|Description|Default value|
    |:-----|:----------|:------------|
    |interactive|Specifies whether DataFrame is used in an interactive environment.|Depends on the measured value|
    |df.analyze|Specifies whether to enable non-MaxCompute built-in functions.|True|
    |df.optimize|Specifies whether to enable full DataFrame optimization.|True|
    |df.optimizes.pp|Specifies whether to enable DataFrame predicate push optimization.|True|
    |df.optimizes.cp|Specifies whether to enable DataFrame column pruning optimization.|True|
    |df.optimizes.tunnel|Specifies whether to enable DataFrame tunnel optimization.|True|
    |df.quote|Specifies whether to use a pair of grave accents \(\`\`\) to mark fields and table names in the backend of MaxCompute SQL.|True|
    |df.libraries|The resource name of the third-party library that is used for DataFrame operation.|None|

-   PyODPS ML configuration

    |Option|Description|Default value|
    |:-----|:----------|:------------|
    |ml.xflow\_project|The default XFlow project name.|algo\_public|
    |ml.use\_model\_transfer|Specifies whether to use ModelTransfer to obtain the PMML model.|True|
    |ml.model\_volume|The volume name used when ModelTransfer is used.|pyodps\_volume|


