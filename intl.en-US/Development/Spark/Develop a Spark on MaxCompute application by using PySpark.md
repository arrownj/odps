---
keyword: develop a Spark on MaxCompute application by using PySpark
---

# Develop a Spark on MaxCompute application by using PySpark

This topic describes how to develop a Spark on MaxCompute application by using PySpark.

If you want to access MaxCompute tables in your application, you must provide the odps-spark-datasource package. For more information, see [Set up a Spark on MaxCompute development environment](/intl.en-US/Development/Spark/Set up a Spark on MaxCompute development environment.md).

## Develop a Spark SQL application in Spark 1.6

Sample code:

```
from pyspark import SparkContext, SparkConf
from pyspark.sql import OdpsContext
if __name__ == '__main__':
    conf = SparkConf().setAppName("odps_pyspark")
    sc = SparkContext(conf=conf)
    sql_context = OdpsContext(sc)
    sql_context.sql("DROP TABLE IF EXISTS spark_sql_test_table")
    sql_context.sql("CREATE TABLE spark_sql_test_table(name STRING, num BIGINT)")
    sql_context.sql("INSERT INTO TABLE spark_sql_test_table SELECT 'abc', 100000")
    sql_context.sql("SELECT * FROM spark_sql_test_table").show()
    sql_context.sql("SELECT COUNT(*) FROM spark_sql_test_table").show()
```

Run the following command to submit and run the code:

```
./bin/spark-submit \
--jars cupid/odps-spark-datasource_xxx.jar \
example.py
```

## Develop a Spark SQL application in Spark 2.3

Sample code:

```
from pyspark.sql import SparkSession
if __name__ == '__main__':
    spark = SparkSession.builder.appName("spark sql").getOrCreate()
    spark.sql("DROP TABLE IF EXISTS spark_sql_test_table")
    spark.sql("CREATE TABLE spark_sql_test_table(name STRING, num BIGINT)")
    spark.sql("INSERT INTO spark_sql_test_table SELECT 'abc', 100000")
    spark.sql("SELECT * FROM spark_sql_test_table").show()
    spark.sql("SELECT COUNT(*) FROM spark_sql_test_table").show()
```

Submit and run the code.

-   Run the following command to submit and run the code in Cluster mode:

    ```
    spark-submit --master yarn-cluster \
    --jars cupid/odps-spark-datasource_xxx.jar \
    example.py
    ```

-   Run the following command to submit and run the code in Local mode:

    ```
    cd $SPARK_HOME
    ./bin/spark-submit --master local[4] \
    --driver-class-path cupid/odps-spark-datasource_xxx.jar \
    /path/to/odps-spark-examples/spark-examples/src/main/python/spark_sql.py
    ```

    **Note:**

    -   In Local mode, the Tunnel service is required to access MaxCompute tables.
    -   In Local mode, you must use the --driver-class-path option to specify resources. Do not use the --jars option.

## Upload required packages

A MaxCompute cluster does not allow you to install Python libraries. If your Spark on MaxCompute application depends on Python libraries, plug-ins, or projects, you can configure your application to use public resources that are provided by third parties. Alternatively, you can compress the required resources to packages on your computer and upload the packages to MaxCompute. For some special resources, the Python version on the computer that you use to compress the resources must be the same as the Python version in the MaxCompute cluster where your application is run. Select a method to provide required resources for your application based on the complexity of your business.

-   Use public resources.
    -   Use public resources for Python 2.7.

        ```
        spark.hadoop.odps.cupid.resources = public.python-2.7-ucs4.zip
        spark.pyspark.python = ./public.python-2.7-ucs4.zip/python-2.7-ucs4/bin/python2.7
        ```

        The following third-party libraries are available:

        ```
        $./bin/pip list
        Package         Version 
        --------------- --------
        boto            2.49.0  
        boto3           1.9.147 
        botocore        1.12.147
        bz2file         0.98    
        certifi         2019.3.9
        chardet         3.0.4   
        docutils        0.14    
        futures         3.2.0   
        gensim          3.7.3   
        idna            2.8     
        jieba           0.39    
        jmespath        0.9.4   
        numpy           1.16.1  
        pandas          0.24.1  
        patsy           0.5.1   
        pip             19.1.1  
        pyodps          0.8.1   
        python-dateutil 2.8.0   
        pytz            2018.9  
        redis           3.2.1   
        requests        2.21.0  
        s3transfer      0.2.0   
        scipy           1.2.1   
        setuptools      41.0.1  
        six             1.12.0  
        smart-open      1.8.3   
        statsmodels     0.9.0   
        urllib3         1.24.3  
        wheel           0.33.4
        xgboost         0.82 
        sklearn         0.0.13
        matplotlib      2.2.4
        ```

    -   Use public resources for Python 3.5.

        ```
        spark.hadoop.odps.cupid.resources = public.python-3.5-ucs4.zip
        spark.pyspark.python = ./public.python-3.5-ucs4.zip/python-3.5-ucs4/bin/python3.5
        ```

        The following third-party libraries are available:

        ```
        Package         Version  
        --------------- ---------
        boto            2.49.0   
        boto3           1.9.172  
        botocore        1.12.172
        bs4             0.0.1 
        bz2file         0.98     
        certifi         2019.6.16
        chardet         3.0.4    
        docutils        0.14     
        gensim          3.7.3    
        idna            2.8      
        jieba           0.39     
        jmespath        0.9.4    
        numpy           1.16.4   
        pandas          0.24.2   
        patsy           0.5.1    
        pip             19.1.1   
        pyodps          0.8.1    
        python-dateutil 2.8.0    
        pytz            2019.1   
        redis           3.2.1    
        requests        2.22.0   
        s3transfer      0.2.1    
        scipy           1.3.0    
        setuptools      41.0.1   
        six             1.12.0   
        smart-open      1.8.4    
        statsmodels     0.9.0    
        urllib3         1.25.3   
        wheel           0.33.4   
        xgboost         0.90
        sklean2         0.0.13
        matplotlib      3.0.3
        ```

-   Compress resources to .egg packages on your computer.

    For example, MLlib requires the NumPy and Setuptools plug-ins. You can compress the plug-ins to .egg packages and then upload the packages by using the `--py-files` option.

    **Note:** MaxCompute uses Python 2.7 or Python 3.5 to run Spark on MaxCompute applications. Therefore, you must compress the plug-ins by using Python 2.7 or Python 3.5 on your computer.

    1.  Compress the NumPy and Setuptools plug-ins to .egg packages. Perform the following steps:
        1.  Download the NumPy and Setuptools plug-ins from the [Python Package Index](https://pypi.org/).
        2.  Go to the source code directory of the Setuptools plug-in and run the `python setup.py bdist_egg` command. A .egg package is generated in the dist directory.
        3.  Go to the source code directory of the NumPy plug-in and run the `python setupeggs.py bdist_egg` command. A .egg package is generated in the dist directory.
    2.  Run the following command to submit your Spark on MaxCompute application:

        ```
        cd $SPARK_HOME
        ./bin/spark-submit --master yarn-cluster \
        --jars cupid/odps-spark-datasource_2.11-3.3.2-hotfix1.jar \
        --py-files /path/to/numpy-1.7.1-py2.7-lunux-x85_64.egg,/path/to/setuptools-33.1.1-py2.7.egg \
        app.py
        ```

-   Compress all Python libraries to a package on your computer.

    If your Spark on MaxCompute application depends on a large number of resources, or files such as .so files that cannot be imported by the zipimport module, you can download and compress all Python libraries to a package, and then upload the package. The package that you compress on your computer may not be usable in the MaxCompute cluster where your application is run. For example, a package that is compressed on a Mac computer cannot be used in the MaxCompute cluster. To avoid this issue, you can use a Docker container to compress the package by performing the following steps:

    **Note:** In the following example, Python 3.7 is used as an example. Change the Python version as required.

    1.  Create a Dockerfile file on the host where Docker is installed. Sample Dockerfile:

        ```
        FROM centos:7.6.1810
        RUN set -ex \
            # Install required components.
            && yum install -y wget tar libffi-devel zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make initscripts zip\
            && wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz \
            && tar -zxvf Python-3.7.0.tgz \
            && cd Python-3.7.0 \
            && ./configure prefix=/usr/local/python3 \
            && make \
            && make install \
            && make clean \
            && rm -rf /Python-3.7.0* \
            && yum install -y epel-release \
            && yum install -y python-pip
        # Set the default Python version to Python 3.
        RUN set -ex \
            # Back up Python 2.7 resources.
            && mv /usr/bin/python /usr/bin/python27 \
            && mv /usr/bin/pip /usr/bin/pip-python27 \
            # Set the default Python version to Python 3.
            && ln -s /usr/local/python3/bin/python3.7 /usr/bin/python \
            && ln -s /usr/local/python3/bin/pip3 /usr/bin/pip
        # Fix the YUM bug that is caused by the Python version change.
        RUN set -ex \
            && sed -i "s#/usr/bin/python#/usr/bin/python27#" /usr/bin/yum \
            && sed -i "s#/usr/bin/python#/usr/bin/python27#" /usr/libexec/urlgrabber-ext-down \
            && yum install -y deltarpm
        # Update pip.
        RUN pip install --upgrade pip
        ```

    2.  Run the following commands in the directory of the Dockerfile file to build a Docker image and start a Docker container with the image:

        ```
        docker build -t python-centos:3.7 .
        docker run -itd --name python3.7 python-centos:3.7
        ```

    3.  Install required Python libraries in the container.

        ```
        docker attach python3.7
        pip install [Required library]
        ```

    4.  Compress all Python libraries to a package.

        ```
        cd /usr/local/
        zip -r python3.7.zip python3/
        ```

    5.  Exit the container and run the following command on the host to copy the package from the container to the host:

        ```
        docker cp python3.7:/usr/local/python3.7.zip
        ```

        **Note:** You can press Ctrl+P+Q to exit the container.

    6.  Upload the python3.7.zip package to MaxCompute. You can upload only packages that are no larger than 50 MB in size to a DataWorks workspace. If the size of a package exceeds 50 MB, upload the package by using the MaxCompute client. For more information, see [Add a resource](/intl.en-US/Development/Common commands/Resource operations.md).
    7.  Add the following configuration to the spark-default.conf file or add the corresponding settings in DataWorks and then submit your Spark on MaxCompute application:

        ```
        spark.hadoop.odps.cupid.resources=[Project name].python3.7.zip  
        spark.pyspark.python=./[Project name].python3.7.zip/python3/bin/python3.7
        ```


