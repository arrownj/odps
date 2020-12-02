---
keyword: [reference third-party packages in Python UDFs, NumPy, dynamic-link library]
---

# Reference third-party packages in Python UDFs

MaxCompute allows you to reference third-party packages in Python user-defined functions \(UDFs\), such as NumPy packages, third-party packages that need to be compiled, and third-party packages that are dependent on dynamic-link libraries \(DLLs\). This topic describes how to reference third-party packages in Python UDFs.

You can use one of the following methods to reference third-party packages in Python UDFs:

-   [Reference NumPy packages in Python 3 UDFs](#section_fvb_vln_9z0)

    You must change the file name extension of the NumPy package, use the MaxCompute client to upload the NumPy package, and then register a function. After the function is registered, it can be called in Python 3 UDFs.

-   [Reference third-party packages that need to be compiled](#section_3sz_bc7_5cf)

    You must compile the setup.py script in a third-party package to generate a wheel package in an environment that is compatible with MaxCompute, and change the file name extension. Then, use the MaxCompute client to upload the third-party package and register a function. After the function is registered, it can be called in Python UDFs. We recommend that you use a Linux operating system. If you use a Windows operating system, we recommend that you use Docker.

-   [Reference third-party packages that are dependent on DLLs](#section_uxo_u1g_j8o)

    You must compile the .so library file based on the source code of a third-party package to generate a wheel package, and change the file name extension. Then, use the MaxCompute client to upload both the wheel package and the .so library file and register a function. After the function is registered, it can be called in Python UDFs.


## Prerequisites

Make sure that the following prerequisites are met:

-   Python is installed. We recommend that you install Python 3.
-   The [MaxCompute client](/intl.en-US/Tools and Downloads/Client.md) is installed and configured. For more information, see [Install and configure the odpscmd client](/intl.en-US/Prepare/Install and configure the odpscmd client.md).
-   [pip](https://pip.pypa.io/en/stable/installing/?spm=a2c4g.11186623.2.11.1d257cafW1D5nG), setuptools, and wheel are installed if you want to use Python UDFs to reference third-party packages that need to be compiled. You can run the `pip install setuptools` command to install setuptools and the `pip install wheel` command to install wheel.
-   PROJ 6 is installed if you use a third-party package of GDAL 3.0 or later.
-   [Docker](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/) is installed if you use Docker to compile third-party packages. For more information, see [Docker documentation](https://docs.docker.com/get-docker/).

## Reference NumPy packages in Python 3 UDFs

You can use Python 3 in MaxCompute to reference NumPy packages. By default, Python 2 in MaxCompute has the NumPy library installed. You do not need to manually upload NumPy packages in Python 2. To reference a NumPy package in Python 3 UDFs, perform the following steps:

1.  In the **Download files** section of the [PyPI](https://pypi.org/project/numpy/#files) page, click the package whose name ends with cp37-cp37m-manylinux1\_x86\_64.whl to download it.

    ![Download the NumPy package](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/2814096061/p181499.png)

    **Note:** If you download another package, the operation may fail. In this example, NumPy 1.19.2 is used.

2.  Change the file name extension of the downloaded NumPy package to .zip.

    Example: numpy-1.19.2-cp37-cp37m-manylinux1\_x86\_64.zip.

3.  Use the [MaxCompute client](/intl.en-US/Tools and Downloads/Client.md) to upload the NumPy package to a MaxCompute project. For more information about how to upload the package, see [Resource operations](/intl.en-US/Development/Common commands/Resource operations.md).

    Example command:

    ```
    ADD ARCHIVE D:\Downloads\numpy-1.19.2-cp37-cp37m-manylinux1_x86_64.zip -f;
    ```

4.  Compile a Python UDF script and save it as a PY file.

    In this example, the saved file is named import\_numpy.py. The following code shows the Python UDF script:

    ```
    from odps.udf import annotate
    
    @annotate("->string")
    class TryImport(object): # The class name is TryImport.
        def __init__(self):
            import sys
            sys.path.insert(0, 'work/numpy-1.19.2-cp37-cp37m-manylinux1_x86_64.zip') # The NumPy package.
    
        def evaluate(self):
            import numpy
            return "import succeed"
    ```

5.  Use the [MaxCompute client](/intl.en-US/Tools and Downloads/Client.md) to upload the import\_numpy.py script to a MaxCompute project as a resource.

    Example command:

    ```
    ADD PY D:\Desktop\import_numpy.py -f;
    ```

6.  Use the uploaded import\_numpy.py script and NumPy package to register a user-defined function on the [MaxCompute client](/intl.en-US/Tools and Downloads/Client.md). For more information about how to register a function, see [Function operations](/intl.en-US/Development/Common commands/Function operations.md).

    In this example, the registered function is named numpy. Example command:

    ```
    CREATE FUNCTION numpy AS 'import_numpy.TryImport' USING 'doc_test_dev/resources/import_numpy.py,numpy-1.19.2-cp37-cp37m-manylinux1_x86_64.zip';
    ```

    **Note:** When you register a function, you must add a NumPy package, such as numpy-1.19.2-cp37-cp37m-manylinux1\_x86\_64.zip, to the resource list.

7.  After you register the function, write SQL statements to call the new function. Make sure that Python 3 is enabled to execute SQL statements. For more information, see [Python 3 UDFs](/intl.en-US/Development/SQL/UDF/Python 3 UDFs.md).


## Reference third-party packages that need to be compiled

If a third-party package is a TAR.GZ package downloaded from [PyPI](https://pypi.org/project/numpy/#files) or a source code package downloaded from GitHub, the setup.py file is sometimes contained in the root directory of the decompressed third-party package. To use this type of third-party package, you must compile the setup.py file to generate a wheel package in an environment that is compatible with MaxCompute. Then, upload the package as a resource and register a function. This way, you can call third-party packages in Python UDFs. For more information about how to upload a resource and register a function, see [Reference NumPy packages in Python 3 UDFs](#section_fvb_vln_9z0).

**Note:**

-   Third-party packages run in Linux. We recommend that you compile third-party packages in a Linux operating system. If you compile third-party packages in a Windows operating system, compatibility issues may occur.
-   If you use a Windows operating system, we recommend that you use the required Python version to compile the setup.py file and generate a wheel package in the [Docker](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/) container created from the quay.io/pypa/manylinux2010\_x86\_64 image. The available Python versions include `/opt/python/cp27-cp27m/bin/python` and `/opt/python/cp37-cp37m/bin/python3`.

If you use a Linux operating system, make sure that the following requirements are met:

-   A compatible Python version is used. You can run the following command on the CLI of your system to check the Python version:

    ```
    python -c "import wheel.pep425tags; print(wheel.pep425tags.get_abi_tag())"
    ```

    -   If `cp27m` or `cp37m` is returned, the version meets the compatibility requirements.
    -   If `cp27mu` or `cp37mu` is returned, the version does not meet the compatibility requirements. In this case, you must run the `./configure --enable-unicode=ucs2` command to change the Python coding format to UCS-2.
-   If code in the C or C++ programming system is required, your Linux operating system must be compatible with the GNU Compiler Collection \(GCC\) version in use.

    **Note:** We recommend that you use GCC 4.9.2 or earlier. If the GCC version is later than 4.9.2, the .so file in the generated wheel package may be incompatible with MaxCompute.


If all the requirements are met, perform the following operations to compile the setup.py file and generate a wheel package:

1.  Decompress a third-party package to your computer and run the required command to go to the folder where the setup.py file is located.

    For example, the GDAL-3.2.0.zip package is downloaded. After you decompress the package, the setup.py file is stored in D:\\Downloads\\GDAL-3.2.0. Example command:

    ```
    cd D:\Downloads\GDAL-3.2.0
    ```

    ![File path after decompression](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/2814096061/p181717.png)

2.  Run the following command to check whether bdist\_wheel is returned:

    ```
    python setup.py --help-command
    ```

    -   If yes, go to [Step 3](#step_l89_2b9_mya).
    -   If no, change `from distutils.core import setup` to `from setuptools import setup` in the setup.py file. Then, go to [Step 3](#step_l89_2b9_mya).
3.  Run the following command to compile the setup.py file and generate a wheel package:

    ```
    python setup.py bdist_wheel # The wheel package is stored in the dist directory.
    ```


## Reference third-party packages that are dependent on DLLs

Some third-party packages for Python depend on Python libraries and other DLLs. This section describes how to use the [Docker](http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/) container to compile the .so library file and generate a wheel package that can be used in MaxCompute. The container is created from the quay.io/pypa/manylinux2010\_x86\_64 image. GDAL 3.0.4 is used in this example. You must upload the generated .so library file, wheel package, or NumPy package as a resource and register a function. After the function is registered, it can be called in Python UDFs. For more information about how to upload a resource and register a function, see [Reference NumPy packages in Python 3 UDFs](#section_fvb_vln_9z0).

**Note:** Before you perform the following operations, make sure that Docker is installed. For more information, see [Docker documentation](https://docs.docker.com/engine/reference/commandline/docker/).

To reference third-party packages that are dependent on DLLs in Python UDFs, perform the following operations:

1.  View dependencies in the **Dependencies** section of the [PyPI](https://pypi.org/project/GDAL/#description) page.

    The following figure shows the dependencies of GDAL 3.0.4.

    ![View dependencies](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/2814096061/p181816.png)

    **Note:** The preceding figure shows that the dependencies include libgdal and numpy. To obtain libgdal, compile the GDAL source code in the Docker container. To obtain numpy, download the NumPy package on the [PyPI](https://pypi.org/project/numpy/#files) page or from the Docker container.

2.  Download the NumPy package.

    You can use one of the following methods to download the NumPy package:

    -   In the **Download files** section of the [PyPI](https://pypi.org/project/numpy/#files) page, click the package whose name ends with cp37-cp37m-manylinux1\_x86\_64.whl to download it.

        **Note:** If Python 2 is used, perform the following operations to download the NumPy package: In the **Navigation** section of the [PyPI](https://pypi.org/project/numpy/#files) page, click **Release history**, select 1.16.6 or an earlier version, and then click the package whose name ends with cp27-cp27m-manylinux1\_x86\_64.whl.

    -   Run the `/opt/python/cp37-cp37m/bin/pip download numpy -d . /` command in the Docker container created from the quay.io/pypa/manylinux2010\_x86\_64 image to download the NumPy package to the current directory.
3.  Compile the. so library file.

    1.  Download the [GDAL 3.0.4 source code](https://trac.osgeo.org/gdal/wiki/DownloadSource) file and decompress it to your computer.

    2.  Download the Docker container created from the quay.io/pypa/manylinux2010\_x86\_64 image and enter the input mode of the client:

        Example commands:

        ```
        docker pull quay.io/pypa/manylinux2010_x86_64
        docker run -it quay.io/pypa/manylinux1_x86_64 /bin/bash
        ```

    3.  Upload the GDAL 3.0.4 source code to a Docker container:

        Example command:

        ```
        docker cp ./gdal-3.0.4 <CONTAINER ID>:/opt/source/  
        ```

        For more information about how to obtain CONTAINER ID, see [docker ps](https://docs.docker.com/engine/reference/commandline/ps/).

4.  Compile GDAL 3.0.4 source code in the container. For more information, see [BuildingOnUnix](https://trac.osgeo.org/gdal/wiki/BuildingOnUnix).

    Example commands:

    ```
    # Specify the directory to install PROJ 6 in the configure field.
    ./configure --prefix=/path/to/install/prefix --with-proj=/path/to/install/proj6/prefix
    make
    make install
    export PATH=/path/to/install/prefix/bin:$PATH
    export LD_LIBRARY_PATH=/path/to/install/prefix/lib:$LD_LIBRARY_PATH
    export GDAL_DATA=/path/to/install/prefix/share/gdal
    # Test
    gdalinfo --version
    ```

    The following errors may occur during compilation:

    -   `configure: error: PROJ 6 symbols not found`: If this error occurs, install PROJ 6 to support GDAL 3.0 and later.
    -   `fatal error: zlib.h: No such file or directory`: If this error occurs, use the `yum install zlib-devel` command instead.
5.  Run the required Docker download commands to download two. so library files \(not symbolic links\) to your computer. Obtain libgdal.so from the lib folder in the installation directory of GDAL and libproj.so from the lib folder in the installation directory of PROJ 6.

6.  Generate a GDAL wheel package in the Docker container. For more information, see [BuildingOnUnix](https://trac.osgeo.org/gdal/wiki/BuildingOnUnix).

    Example commands:

    ```
    # If NumPy is required, install NumPy first.
    /opt/python/cp37-cp37m/bin/pip install numpy
    # Switch to the directory for GDAL source code.
    cd swig/python
    # Generate a wheel package and save it in the dist directory. Example: GDAL-3.0.4-cp37-cp37m-linux_x86_64.whl.
    /opt/python/cp37-cp37m/bin/python setup.py bdist_wheel
    ```

7.  Upload the generated .so library file, wheel package, or NumPy package as a resource and register a function. After the function is registered, it can be called in Python UDFs. For more information about how to upload a resource and register a function, see [Reference NumPy packages in Python 3 UDFs](#section_fvb_vln_9z0).

    Take note of the following items:

    -   When you upload resources, you must upload libgdal.so and libproj.so as file resources and numpy-1.19.2-cp37-cp37m-manylinux1\_x86\_64.zip and GDAL-3.0.4-cp37-cp37m-linux\_x86\_64.zip as archive resources.
    -   When you register functions, you must add libgdal.so, libproj.so, numpy-1.19.2-cp37-cp37m-manylinux1\_x86\_64.zip, and GDAL-3.0.4-cp37-cp37m-linux\_x86\_64.zip to the resource list of the functions.
    Sample code for Python UDFs:

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
            sys.path.insert(0, 'work/GDAL-3.0.4-cp37-cp37m-linux_x86_64.zip') # The GDAL package after compilation.
            sys.path.insert(0, 'work/numpy-1.19.2-cp37-cp37m-manylinux1_x86_64.zip') # The NumPy package.
    
        def evaluate(self):
            from osgeo import gdal
            from osgeo import ogr
            from osgeo import osr
            from osgeo import gdal_array
            from osgeo import gdalconst
            return "import succeed"
    ```

    **Note:** If an error that indicates libgdal.so.26 and libproj.so.15 cannot be identified occurs, you must change libgdal.so to libgdal.so.26 and libproj.so to libproj.so.15.


