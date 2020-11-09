# IPython

PyODPS provides an IPython plug-in to simplify the operations on MaxCompute.

## IPython enhancement

Some commands are provided for command line enhancement.

-   Run the following commands to load the plug-in:

    ```
    %load_ext odps
    %enter
    ```

    The following result is returned:

    ```
    <odps.inter.Room at 0x11341df10>
    ```

-   In this case, the `o` and `odps` global variables can be retrieved. You can run the `o.get_table` or `odps.get_table` command to call a table.

    ```
    o.get_table('table_name')
    odps.get_table('table_name')
    ```

    The following result is returned:

    ```
    odps.Table
      name: odps_test_sqltask_finance.`table_name`
      schema:
        c_int_a                 : bigint
        c_int_b                 : bigint
        c_double_a              : double
        c_double_b              : double
        c_string_a              : string
        c_string_b              : string
        c_bool_a                : boolean
        c_bool_b                : boolean
        c_datetime_a            : datetime
        c_datetime_b            : datetime
    ```

-   Run the following command to display the stored objects as a table:

    ```
    %stores
    ```

    |default name|desc|
    |:-----------|:---|
    |iris|Iris dataset|


## Object name completion

PyODPS enhances the code completion feature that is provided by IPython. When you write a statement such as `o.get_xxx`, the object name is automatically completed. In the following examples, <tab\> is used to denote pressing the Tab key. When you enter the statement and encounter <tab\>, press the Tab key.

-   You can use the Tab key to complete the object name.

    ```
    o.get_table(<tab>
    ```

-   You can enter the first few characters of an object name and press the Tab key to complete it.

    ```
    o.get_table('tabl<tab>
    ```

    IPython auto-completes the table name that starts with `tabl`.

-   This feature also completes the names of objects in different projects. Syntax:

    ```
    o.get_table(project='project_name', name='tabl<tab>
    o.get_table('tabl<tab>', project='project_name')
    ```

-   If multiple matching objects exist, IPython provides a list. `options.completion_size` specifies the maximum number of objects in the list. The default value is 10.

## SQL statements

PyODPS provides an SQL plug-in to execute MaxCompute SQL statements.

-   You can use %sql to execute a single-line SQL statement.

    ```
    In [37]: %sql select * from pyodps_iris limit 5
    |==========================================|   1 /  1  (100.00%)         3s
    Out[37]:
       sepallength  sepalwidth  petallength  petalwidth         name
    0          5.1         3.5          1.4         0.2  Iris-setosa
    1          4.9         3.0          1.4         0.2  Iris-setosa
    2          4.7         3.2          1.3         0.2  Iris-setosa
    3          4.6         3.1          1.5         0.2  Iris-setosa
    4          5.0         3.6          1.4         0.2  Iris-setosa
    ```

-   You can use `%%sql` to execute a multiple-line SQL statement.

    ```
    In [38]: %%sql
       ....: select * from pyodps_iris
       ....: where sepallength < 5
       ....: limit 5
       ....:
    |==========================================|   1 /  1  (100.00%)        15s
    Out[38]:
       sepallength  sepalwidth  petallength  petalwidth         name
    0          4.9         3.0          1.4         0.2  Iris-setosa
    1          4.7         3.2          1.3         0.2  Iris-setosa
    2          4.6         3.1          1.5         0.2  Iris-setosa
    3          4.6         3.4          1.4         0.3  Iris-setosa
    4          4.4         2.9          1.4         0.2  Iris-setosa
    ```

-   To execute parameterized SQL statements, you can use `:parameter` to specify the parameter.

    ```
    In [1]: %load_ext odps
    
    In [2]: mytable = 'table_name'
    
    In [3]: %sql select * from :mytable
    |==========================================|   1 /  1  (100.00%)         2s
    Out[3]:
       c_int_a  c_int_b  c_double_a  c_double_b  c_string_a  c_string_b c_bool_a  \
    0        0        0       -1203           0           0       -1203     True
    
      c_bool_b         c_datetime_a         c_datetime_b
    0    False  2012-03-30 23:59:58  2012-03-30 23:59:59
    ```

-   For SQL runtime parameters, you can use `%set` to set a global parameter or use SET within an SQLCell to set a local parameter. The following example sets a local parameter, which does not affect the global settings.

    ```
    In [17]: %%sql
             set odps.sql.mapper.split.size = 16;
             select * from pyodps_iris;
    ```

    The following example sets a global parameter. Global settings are applied to all subsequent SQL statements.

    ```
    In [18]: %set odps.sql.mapper.split.size = 16
    ```


## Upload pandas DataFrame objects to MaxCompute tables

PyODPS provides the commands to upload pandas DataFrame objects to MaxCompute tables. You only need to run the `%persist` command. The first parameter `df` is the variable name. The second parameter `pyodps_pandas_df` is the MaxCompute table name.

```
import pandas as pd
import numpy as np

df = pd.DataFrame(np.arange(9).reshape(3, 3), columns=list('abc'))
%persist df pyodps_pandas_df
```

