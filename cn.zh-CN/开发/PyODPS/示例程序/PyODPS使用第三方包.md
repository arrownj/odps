---
keyword: [第三方包, PyODPS]
---

# PyODPS使用第三方包

本文为您介绍如何在PyODPS中使用第三方包。

-   已[开通MaxCompute](/cn.zh-CN/准备工作/开通MaxCompute.md)。
-   已[开通DataWorks](https://common-buy.aliyun.com/)。

1.  下载下表中的资源包。

    |包名|文件名|上传资源名|
    |:-|:--|:----|
    |python-dateutil|[python-dateutil-2.6.0.zip](http://mirrors.aliyun.com/pypi/packages/95/8e/71125f3f24771f50e630b5a6fa9fd209a9f167dcbc3aad65a48cb3dd5694/python-dateutil-2.6.0.zip#md5=530f7b56e36fa42ada6c02a17b15660c)|python-dateutil.zip|
    |pytz|[pytz-2017.2.zip](http://mirrors.aliyun.com/pypi/packages/a4/09/c47e57fc9c7062b4e83b075d418800d322caa87ec0ac21e6308bd3a2d519/pytz-2017.2.zip#md5=f89bde8a811c8a1a5bac17eaaa94383c)|pytz.zip|
    |six|[six-1.11.0.tar.gz](http://mirrors.aliyun.com/pypi/packages/16/d8/bc6316cf98419719bd59c91742194c111b6f2e85abac88e496adefaf7afe/six-1.11.0.tar.gz#md5=d12789f9baf7e9fb2524c0c64f1773f8)|six.tar.gz|
    |pandas|[pandas-0.20.2-cp27-cp27m-manylinux1\_x86\_64.whl](http://mirrors.aliyun.com/pypi/packages/44/39/e71009a0ebdbb6206b9fbde0367fc5cb5bb7fdb4521ae785ca7bd63d36aa/pandas-0.20.2-cp27-cp27m-manylinux1_x86_64.whl#md5=31a4d180048f72337d53cc7b87424568)|pandas.zip|
    |scipy|[scipy-0.19.0-cp27-cp27m-manylinux1\_x86\_64.whl](http://mirrors.aliyun.com/pypi/packages/ae/94/28ca6f9311e2351bb68da41ff8c1bc8f82bb82791f2ecd34efa953e60576/scipy-0.19.0-cp27-cp27m-manylinux1_x86_64.whl#md5=0e49f7fc8d31c1c79f0a4d63b29e8a1f)|scipy.zip|
    |scikit-learn|[scikit\_learn-0.18.1-cp27-cp27m-manylinux1\_x86\_64.whl](http://mirrors.aliyun.com/pypi/packages/ca/dd/a18dba8ab879b13b43c3838a25887585a45101f4bffa398e1883e1e3d395/scikit_learn-0.18.1-cp27-cp27m-manylinux1_x86_64.whl#md5=b068bde57f00d285cc89eb0b8615fcae)|sklearn.zip|

    **说明：** pandas、scipy和scikit-learn资源文件需要您手动压缩为与**上传资源名**同类型的压缩包，才可以上传。

2.  登录[DataWorks控制台](https://workbench.data.aliyun.com/console)。

3.  新建业务流程。

    1.  进入**数据开发**，右键单击**业务流程**，选择**新建业务流程**。

    2.  在**新建业务流程**对话框，输入**业务名称**，单击**新建**，完成业务流程创建。

4.  在DataWorks中创建和提交资源。

    1.  在**DataStudio**（数据开发）页面，鼠标悬停至![新建](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/1553593061/p177535.png)图标，选择**MaxCompute** \> **资源** \> **Archive**

        您也可以打开相应的业务流程，右键单击**MaxCompute**，选择**新建** \> **资源** \> **Archive**

    2.  在**新建资源**对话框，单击**点击上传**，选择python-dateutil-2.6.0.zip文件。

        ![点击上传资源](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/2390659951/p52923.jpg)

    3.  将资源名称命名为python.dateutil.zip，单击**确定**。

        ![资源重命名](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3390659951/p52940.jpg)

    4.  单击![提交](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3390659951/p148533.png)，完成本次上传。

        ![提交资源](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3390659951/p52944.jpg)

    5.  按照上述步骤完成资源pytz.zip、six.tar.gz、pandas.zip、sklearn.zip和scipy.zip的创建与提交。

5.  新建PyODPS节点。

    1.  右键单击业务流程，选择**新建** \> **MaxCompute** \> **PyODPS 2**。

    2.  输入**节点名称**，单击**提交**。

    3.  在新建的PyODPS节点中，写入代码。

        代码示例如下。

        ```
        def test(x):
            from sklearn import datasets, svm
            from scipy import misc
            import numpy as np
        
            iris = datasets.load_iris()
            assert iris.data.shape == (150, 4)
            assert np.array_equal(np.unique(iris.target),  [0, 1, 2])
        
            clf = svm.LinearSVC()
            clf.fit(iris.data, iris.target)
            pred = clf.predict([[5.0, 3.6, 1.3, 0.25]])
            assert pred[0] == 0
        
            assert misc.face().shape is not None
        
            return x
        
        from odps import options
        
        hints = {
            'odps.isolation.session.enable': True
        }
        libraries = ['python-dateutil.zip', 'pytz.zip', 'six.tar.gz', 'pandas.zip', 'scipy.zip', 'sklearn.zip']
        
        iris = o.get_table('pyodps_iris').to_df()
        
        print iris[:1].sepallength.map(test).execute(hints=hints, libraries=libraries)
                                    
        ```

6.  单击![运行](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3390659951/p148536.png)。

7.  在**运行日志**中查看运行结果，如下所示。

    ```
    Sql compiled:
    CREATE TABLE tmp_pyodps_a3172c30_a0d7_4c88_bc39_434168263897 LIFECYCLE 1 AS
    SELECT pyodps_udf_1576485276_94d9d978_af66_4e27_a874_e787022dfb3d(t1.`sepallength`) AS `sepallength`
    FROM WB_BestPractice_dev.`pyodps_iris` t1
    LIMIT 1
    
    Instance ID: 20191216083438175gcv6n4pr2
      Log view: http://logview.odps.aliyun.com/logview/?h=xxxxxx
    
       sepallength
    0          5.1
    ```

    **说明：** 最佳实践案例请参见[PyODPS节点实现结巴中文分词]()。


