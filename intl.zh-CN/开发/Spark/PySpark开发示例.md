---
keyword: PySpark开发示例
---

# PySpark开发示例

本文为您介绍PySpark开发示例。

如果要访问MaxCompute表，则需要编译datasource包，详细步骤请参见[搭建开发环境](/intl.zh-CN/开发/Spark/搭建开发环境.md)。

## SparkSQL应用示例（Spark1.6）

详细代码

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

提交运行

```
./bin/spark-submit \
--jars cupid/odps-spark-datasource_xxx.jar \
example.py
```

## SparkSQL应用示例（Spark2.3）

详细代码

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

提交运行

-   Cluster模式提交运行

    ```
    spark-submit --master yarn-cluster \
    --jars cupid/odps-spark-datasource_xxx.jar \
    example.py
    ```

-   Local模式运行

    ```
    cd $SPARK_HOME
    ./bin/spark-submit --master local[4] \
    --driver-class-path cupid/odps-spark-datasource_xxx.jar \
    /path/to/odps-spark-examples/spark-examples/src/main/python/spark_sql.py
    ```

    **说明：**

    -   Local模式访问表依赖Tunnel。
    -   Local模式要用--driver-class-path而非--jars。

## Package依赖

由于MaxCompute集群无法自由安装Python库，PySpark依赖其它Python库、插件、项目时，通常需要在本地打包后通过Spark-submit上传。对于特定依赖，打包环境需与线上环境保持一致。打包方式如下，请根据业务的复杂度进行选择：

-   不打包直接采用公共资源
    -   默认提供python 2.7环境配置

        ```
        spark.hadoop.odps.cupid.resources = public.python-2.7-ucs4.zip
        spark.pyspark.python = ./public.python-2.7-ucs4.zip/python-2.7-ucs4/bin/python2.7
        ```

        第三方库列表如下。

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

    -   默认提供python 3.5环境配置

        ```
        spark.hadoop.odps.cupid.resources = public.python-3.5-ucs4.zip
        spark.pyspark.python = ./public.python-3.5-ucs4.zip/python-3.5-ucs4/bin/python3.5
        ```

        第三方库列表如下。

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

-   本地打egg包。

    例如，mllib需要numpy和setuptool两个插件，因此要制做插件对应的egg包，通过`--py-files`上传。

    **说明：** 由于运行Spark作业的线上环境是Python2.7或者Python3.5，所以本地必须使用Python2.7或者Python3.5打包。

    1.  制作numpy、setuptools egg包。具体操作步骤如下：
        1.  下载numpy和setuptools[安装包](https://pypi.org/)。
        2.  进入setuptools源码路径，执行`Python setup.py bdist_egg`后，会在dist目录生成egg文件。
        3.  进入numpy源码路径，执行`Python setupeggs.py bdist_egg`后，会在dist目录生成egg文件。
    2.  提交Spark作业。

        ```
        cd $SPARK_HOME
        ./bin/spark-submit --master yarn-cluster \
        --jars cupid/odps-spark-datasource_2.11-3.3.2-hotfix1.jar \
        --py-files /path/to/numpy-1.7.1-py2.7-lunux-x85_64.egg,/path/to/setuptools-33.1.1-py2.7.egg \
        app.py
        ```

-   本地打Python包。

    如果依赖链太长，导致egg包无法满足需求，或依赖包含so文件等无法通过zipimport引入的文件，则需要把所有依赖下载后打Python包，再把整个Python包上传。为了保证打包环境与线上环境一致（例如mac环境下的打包文件与线上环境不兼容），使用docker容器进行打包，具体步骤如下：

    **说明：** 此方法以打包python3.7运行环境为例，请根据您的实际情况进行修改。

    1.  Docker镜像制作，在安装了docker环境的宿主机新建一个Dockerfile文件。示例文件如下。

        ```
        FROM centos:7.6.1810
        RUN set -ex \
            # 预安装所需组件。
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
        # 设置默认为Python3。
        RUN set -ex \
            # 备份旧版本Python。
            && mv /usr/bin/python /usr/bin/python27 \
            && mv /usr/bin/pip /usr/bin/pip-python27 \
            # 配置默认为Python3
            && ln -s /usr/local/python3/bin/python3.7 /usr/bin/python \
            && ln -s /usr/local/python3/bin/pip3 /usr/bin/pip
        # 修复因修改Python版本导致yum失效问题。
        RUN set -ex \
            && sed -i "s#/usr/bin/python#/usr/bin/python27#" /usr/bin/yum \
            && sed -i "s#/usr/bin/python#/usr/bin/python27#" /usr/libexec/urlgrabber-ext-down \
            && yum install -y deltarpm
        # 更新pip版本。
        RUN pip install --upgrade pip
        ```

    2.  构建镜像并运行容器。在Dockerfile文件的目录下运行如下命令。

        ```
        docker build -t python-centos:3.7 .
        docker run -itd --name python3.7 python-centos:3.7
        ```

    3.  进入容器安装所需的Python依赖库。

        ```
        docker attach python3.7
        pip install [所需依赖库]
        ```

    4.  打包Python环境。

        ```
        cd /usr/local/
        zip -r python3.7.zip python3/
        ```

    5.  拷贝容器中的Python环境到宿主机。退出容器，在宿主机运行如下命令。

        ```
        docker cp python3.7:/usr/local/python3.7.zip
        ```

        **说明：** 退出容器命令为ctrl+P+Q。

    6.  上传Python3.7.zip包到Maxcompute上。由于DataWorks最大只能上传50MB的包，大于50MB的包请通过Maxcompute客户端上传。命令请参考[添加资源](/intl.zh-CN/开发/常用命令/资源操作.md)。
    7.  提交作业时需要在spark-default.conf或DataWorks配置项中添加以下配置即可。

        ```
        spark.hadoop.odps.cupid.resources=[project名].python3.7.zip  
        spark.pyspark.python=./[project名].python3.7.zip/python3/bin/python3.7
        ```


