---
keyword: [Python UDF引用第三方包, Numpy, 动态链接库]
---

# Python UDF使用第三方包

MaxCompute支持您在Python UDF中引用第三方包，例如Numpy包、需要编译的第三方包或依赖动态链接库的第三方包。本文为您介绍如何通过Python UDF引用第三方包。

通过Python UDF使用第三方包支持的场景如下：

-   [使用Numpy包（Python 3 UDF）](#section_fvb_vln_9z0)

    您需要修改Numpy包的后缀格式，基于MaxCompute客户端上传Numpy包，并注册函数。函数注册成功后即可通过Python 3 UDF调用。

-   [使用需要编译的第三方包](#section_3sz_bc7_5cf)

    您需要在与MaxCompute兼容的环境下，对第三方资源包中的setup.py脚本进行编译生成WHEEL包，并修改后缀格式。基于MaxCompute客户端上传包，并注册函数。函数注册成功后即可通过Python UDF调用。推荐使用Linux环境，Windows用户推荐使用Docker。

-   [使用依赖动态链接库的第三方包](#section_uxo_u1g_j8o)

    您需要基于第三方包的源码编译so链接库，然后编译生成WHEEL包，并修改后缀格式。基于MaxCompute客户端上传包和so链接库文件，并注册函数。函数注册成功后即可通过Python UDF调用。


## 前提条件

在执行操作前，请确认已完成如下操作：

-   已安装Python环境。推荐使用Python 3。
-   已安装并配置[MaxCompute客户端](/intl.zh-CN/工具及下载/客户端.md)。客户端配置详情请参见[安装并配置客户端](/intl.zh-CN/准备工作/安装并配置客户端.md)。
-   如果您通过Python UDF使用需要编译的第三方包，请确认已安装[pip](https://pip.pypa.io/en/stable/installing/?spm=a2c4g.11186623.2.11.1d257cafW1D5nG)、setuptools（通过`pip install setuptools`安装）和wheel（通过`pip install wheel`安装）。
-   如果您使用的第三方包为GDAL 3.0及以上版本，请确认已安装PROJ 6。
-   如果您通过Docker编译第三方包，请确认已安装[Docker](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/)，详情请参见[Docker安装文档](https://docs.docker.com/get-docker/)。

## 使用Numpy包（Python 3 UDF）

您可以通过MaxCompute内置的Python 3环境使用Numpy包。MaxCompute内置的Python 2环境默认安装了Numpy，不需要手动上传Numpy包。通过Python 3 UDF使用Numpy包的步骤如下：

1.  在[PyPI](https://pypi.org/project/numpy/#files)页面的**Download files**区域，单击文件名后缀为cp37-cp37m-manylinux1\_x86\_64.whl的Numpy包进行下载。

    ![下载numpy包](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/3946715061/p181499.png)

    **说明：** 其它后缀名称的包可能会加载失败。本文以1.19.2版本为例进行介绍。

2.  修改下载的Numpy包后缀为ZIP格式。

    例如numpy-1.19.2-cp37-cp37m-manylinux1\_x86\_64.zip。

3.  通过[MaxCompute客户端](/intl.zh-CN/工具及下载/客户端.md)上传Numpy包至MaxCompute项目空间。上传资源详情请参见[资源操作](/intl.zh-CN/开发/常用命令/资源操作.md)。

    命令示例如下：

    ```
    ADD ARCHIVE D:\Downloads\numpy-1.19.2-cp37-cp37m-manylinux1_x86_64.zip -f;
    ```

4.  编写Python UDF脚本，保存为PY格式文件。

    假设此处保存的脚本名称为import\_numpy.py。Python UDF脚本示例如下：

    ```
    from odps.udf import annotate
    
    @annotate("->string")
    class TryImport(object): #类名为TryImport。
        def __init__(self):
            import sys
            sys.path.insert(0, 'work/numpy-1.19.2-cp37-cp37m-manylinux1_x86_64.zip') #Numpy包。
    
        def evaluate(self):
            import numpy
            return "import succeed"
    ```

5.  通过[MaxCompute客户端](/intl.zh-CN/工具及下载/客户端.md)将import\_numpy.py脚本以资源形式上传至MaxCompute项目空间。

    命令示例如下：

    ```
    ADD PY D:\Desktop\import_numpy.py -f;
    ```

6.  使用上传的import\_numpy.py脚本及Numpy包，通过[MaxCompute客户端](/intl.zh-CN/工具及下载/客户端.md)注册自定义函数。注册函数详情请参见[函数操作](/intl.zh-CN/开发/常用命令/函数操作.md)。

    假设注册的自定义函数名为numpy。命令示例如下：

    ```
    CREATE FUNCTION numpy AS 'import_numpy.TryImport' USING 'doc_test_dev/resources/import_numpy.py,numpy-1.19.2-cp37-cp37m-manylinux1_x86_64.zip';
    ```

    **说明：** 注册函数时资源列表里需要加上Numpy包，例如numpy-1.19.2-cp37-cp37m-manylinux1\_x86\_64.zip。

7.  完成注册后您即可编写SQL语句调用新建的自定义函数。执行SQL语句时需要开启 Python3，详情请参见[Python 3 UDF](/intl.zh-CN/开发/SQL及函数/UDF/Python 3 UDF.md)。


## 使用需要编译的第三方包

如果第三方包是[PyPI](https://pypi.org/project/numpy/#files)页面中格式为TAR.GZ的压缩包，或从GitHub下载的源码包，这些包解压后的根目录下有时会存在setup.py文件。在使用这种类型的第三方包前，您需要先在与MaxCompute兼容的环境下将setup.py编译生成WHEEL包，然后再执行上传资源及注册函数操作，即可通过Python UDF调用第三方包。上传资源及注册函数操作请参见[使用Numpy包（Python 3 UDF）](#section_fvb_vln_9z0)。

**说明：**

-   由于第三方包是运行在Linux环境的，推荐您使用Linux环境编译第三方包，使用Windows环境编译会存在包不兼容的问题。
-   如果您使用Windows环境，推荐在[Docker](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/)的quay.io/pypa/manylinux2010\_x86\_64镜像容器中使用对应版本的Python（`/opt/python/cp27-cp27m/bin/python`或`/opt/python/cp37-cp37m/bin/python3`）编译生成WHEEL包。

以Linux环境为例，确保环境兼容的关注点如下：

-   需要使用兼容的Python版本。在系统的命令行窗口执行如下命令，检查环境Python版本。

    ```
    python -c "import wheel.pep425tags; print(wheel.pep425tags.get_abi_tag())"
    ```

    -   如果返回值为`cp27m`或`cp37m`，表示Python版本满足兼容性。
    -   如果返回值为`cp27mu`或`cp37mu`，表示Python版本不满足兼容性。您需要在系统的命令行窗口执行`./configure --enable-unicode=ucs2`命令配置Python编码格式为UCS2。
-   如果涉及C或C++代码依赖，需要使用兼容的GCC（GNU Compiler Collection）版本。

    **说明：** 推荐您使用GCC 4.9.2及以下版本，GCC版本高于4.9.2时，编译生成的WHEEL包中的SO类型文件可能与MaxCompute环境不兼容。


确认环境满足兼容性要求后，以Linux系统为例，通过setup.py生成WHEEL包的步骤如下：

1.  将第三方包解压到本地，在系统的命令行窗口，切换路径至setup.py文件所在文件夹。

    例如，下载的包为GDAL-3.2.0.zip，解压后setup.py文件所在路径为D:\\Downloads\\GDAL-3.2.0，命令示例如下：

    ```
    cd D:\Downloads\GDAL-3.2.0
    ```

    ![解压路径](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/3946715061/p181717.png)

2.  在系统的命令行窗口，执行如下命令查看返回结果中是否有bdist\_wheel。

    命令示例如下：

    ```
    python setup.py --help-command
    ```

    -   如果返回结果中有bdist\_wheel，请执行[步骤3](#step_l89_2b9_mya)。
    -   如果返回结果中没有bdist\_wheel，您需要修改setup.py脚本，将脚本中的`from distutils.core import setup`修改为`from setuptools import setup`，然后再执行[步骤3](#step_l89_2b9_mya)。
3.  在系统的命令行窗口，执行如下命令编译生成WHEEL包。

    ```
    python setup.py bdist_wheel # WHEEL包在dist目录下。
    ```


## 使用依赖动态链接库的第三方包

部分Python第三方包除了依赖Python库，可能还会依赖其它动态链接库的依赖。以GDAL 3.0.4为例，为您介绍如何使用[Docker](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/)的quay.io/pypa/manylinux2010\_x86\_64镜像容器，编译相关的so链接库，并编译生成可以在MaxCompute上使用的WHEEL包。基于生成的so链接库文件、WHEEL包或Numpy包，再执行上传资源及注册函数操作，即可通过Python UDF调用第三方包。上传资源及注册函数操作请参见[使用Numpy包（Python 3 UDF）](#section_fvb_vln_9z0)。

**说明：** 请确认您已安装Docker后再执行后续步骤。Docker操作详情请参见[Docker文档](https://docs.docker.com/engine/reference/commandline/docker/)。

通过Python UDF使用依赖so链接库的第三方包的操作步骤如下：

1.  查看依赖项。您可以在[PyPI](https://pypi.org/project/GDAL/#description)页面的**Dependencies**区域查看依赖项。

    例如GDAL 3.0.4的依赖项如下。

    ![查看依赖项](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/5568715061/p181816.png)

    **说明：** 在图示中，依赖项包括libgdal和numpy。libgdal需要在镜像容器中通过编译GDAL源码得到，numpy需要在[PyPI](https://pypi.org/project/numpy/#files)页面或通过Docker镜像容器下载Numpy包。

2.  下载Numpy包。

    您可以选择如下两种方式之一下载Numpy包：

    -   在[PyPI](https://pypi.org/project/numpy/#files)页面的**Download files**区域，单击文件名后缀为cp37-cp37m-manylinux1\_x86\_64.whl的Numpy包进行下载。

        **说明：** 需要注意的是，如果您的Python环境为Python 2，Numpy包需要在[PyPI](https://pypi.org/project/numpy/#files)页面左侧的**Navigation**区域，单击**Release history**，选择1.16.6及之前版本，且后缀为cp27-cp27m-manylinux1\_x86\_64.whl的包进行下载。

    -   在Docker的quay.io/pypa/manylinux2010\_x86\_64镜像容器中，执行`/opt/python/cp37-cp37m/bin/pip download numpy -d ./`命令下载Numpy包到当前目录。
3.  编译so链接库。

    1.  下载[GDAL 3.0.4源码](https://trac.osgeo.org/gdal/wiki/DownloadSource)并解压到本地。

    2.  使用Docker下载quay.io/pypa/manylinux2010\_x86\_64镜像容器，进入终端输入模式。

        命令示例如下：

        ```
        docker pull quay.io/pypa/manylinux2010_x86_64
        docker run -it quay.io/pypa/manylinux1_x86_64 /bin/bash
        ```

    3.  上传GDAL 3.0.4源码到镜像容器中。

        命令示例如下：

        ```
        docker cp ./gdal-3.0.4 <CONTAINER ID>:/opt/source/  
        ```

        CONTAINER ID获取方式请参见[docker ps](https://docs.docker.com/engine/reference/commandline/ps/)。

4.  在镜像容器中编译GDAL 3.0.4，操作详情请参见[BuildingOnUnix](https://trac.osgeo.org/gdal/wiki/BuildingOnUnix)。

    命令示例如下：

    ```
    # configure选项中需要指定安装PROJ 6的位置。
    ./configure --prefix=/path/to/install/prefix --with-proj=/path/to/install/proj6/prefix
    make
    make install
    export PATH=/path/to/install/prefix/bin:$PATH
    export LD_LIBRARY_PATH=/path/to/install/prefix/lib:$LD_LIBRARY_PATH
    export GDAL_DATA=/path/to/install/prefix/share/gdal
    # Test
    gdalinfo --version
    ```

    编译过程中可能出现如下报错：

    -   `configure: error: PROJ 6 symbols not found`：GDAL 3.0及以上版本依赖PROJ 6，需要您下载安装PROJ 6。
    -   `fatal error: zlib.h: No such file or directory`：改用`yum install zlib-devel`命令编译。
5.  使用Docker下载命令将两个so链接库（非软链接）下载到本机，在GDAL、PROJ 6安装目录的lib文件夹下获取libgdal.so和libproj.so。

6.  在镜像容器中制作GDAL WHEEL包。操作详情请参见[BuildingOnUnix](https://trac.osgeo.org/gdal/wiki/BuildingOnUnix)。

    命令示例如下：

    ```
    # 如果需要numpy支持，要先安装numpy。
    /opt/python/cp37-cp37m/bin/pip install numpy
    # 切换到GDAL源码目录下。
    cd swig/python
    # 生成WHEEL包，在dist目录下。WHEEL包示例：GDAL-3.0.4-cp37-cp37m-linux_x86_64.whl
    /opt/python/cp37-cp37m/bin/python setup.py bdist_wheel
    ```

7.  基于生成的so链接库文件、WHEEL包或Numpy包，再执行上传资源及注册函数操作，即可实现通过Python UDF使用第三方包。上传资源及注册函数操作请参见[使用Numpy包（Python 3 UDF）](#section_fvb_vln_9z0)。

    需要注意的是：

    -   在上传资源时，您需要以FILE资源形式上传libgdal.so和libproj.so，以ARCHIVE资源形式上传numpy-1.19.2-cp37-cp37m-manylinux1\_x86\_64.zip和GDAL-3.0.4-cp37-cp37m-linux\_x86\_64.zip。
    -   在注册函数时，您需要在函数资源列表中添加libgdal.so、libproj.so、numpy-1.19.2-cp37-cp37m-manylinux1\_x86\_64.zip和GDAL-3.0.4-cp37-cp37m-linux\_x86\_64.zip。
    Python UDF代码示例如下：

    ```
    # coding: utf-8
    from odps.udf import annotate
    from odps.distcache import get_cache_file
    
    def include_file(file_name):
        import os, sys
        so_file = get_cache_file(file_name, 'b')
        
        with open(so_file.name, 'rb') as fp:
            content=fp.read()
            so = open(file_name, "wb")
            so.write(content)
            so.flush()
            so.close()
    
    @annotate("->string")
    class TryImport(object):
        def __init__(self):
            import sys
            include_file('libgdal.so.26')
            include_file('libproj.so.15')
            sys.path.insert(0, 'work/GDAL-3.0.4-cp37-cp37m-linux_x86_64.zip') #编译后的GDAL包。
            sys.path.insert(0, 'work/numpy-1.19.2-cp37-cp37m-manylinux1_x86_64.zip') #Numpy包。
    
        def evaluate(self):
            from osgeo import gdal
            from osgeo import ogr
            from osgeo import osr
            from osgeo import gdal_array
            from osgeo import gdalconst
            return "import succeed"
    ```

    **说明：** 运行时如果报错找不到libgdal.so.26、libproj.so.15，您需要修改libgdal.so、libproj.so为libgdal.so.26、libproj.so.15。


