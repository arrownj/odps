# Command line

PyODPS provides an enhanced command line tool.

After you configure an account, you do not need to enter the account information again. You can perform the following steps to configure an account and call objects:

1.  Import the PyODPS enhancement tool.

    ```
    >>> from odps.inter import setup, enter, teardown
    ```

2.  Configure your account.

    ```
    >>> setup('**your-access_id**', '**your-access-key**', '**your-project**', endpoint='**your-endpoint**')
    ```

    **Note:** If you do not specify the `room` parameter, the `default``room` is used.

3.  Call the enter method to create a room object on any Python interactive interface.

    ```
    >>> room = enter()
    >>> o = room.odps
    >>> o.get_table('table_name')
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

    **Note:** The MaxCompute object is not automatically updated when you change the setup of the `room`. You must call the `enter` method again to retrieve the new `room` object.


After you configure an account and call objects, you can store, retrieve, or delete objects in the `room` or delete the entire `room`.

-   You can store commonly used MaxCompute tables or resources in the `room`.

    ```
    >>> room.store('stored-table', o.get_table('table_name'), desc='Simple stored table example')
    ```

    You can call the `display` method to display the stored objects as a table.

    ```
    >>> room.display()
    default name         desc
    stored-table     Simple stored table example
    iris          Iris dataset
    ```

-   You can run the `room['stored-table']` or `room.iris` command to retrieve the stored objects.

    ```
    >>> room['stored-table']
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

-   You can call the `drop` method to delete objects from the `room`.

    ```
    >>> room.drop('stored-table')
    >>> room.display()
    default name         desc
    iris          Iris dataset
    ```

-   You can call the `teardown` method to delete a `room`. If no parameters are specified, the default `room` is deleted.

    ```
    teardown()
    ```


