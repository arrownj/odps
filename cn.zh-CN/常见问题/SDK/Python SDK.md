---
keyword: Python SDK常见问题
---

# Python SDK

本文为您介绍Python SDK常见问题。

-   安装常见问题：
    -   [PyODPS安装时提示Warning: XXX not installed，如何处理？](#section_wvb_11n_svl)
    -   [PyODPS安装时提示Project Not Found，如何处理？](#section_3al_tte_z1a)
    -   [PyODPS安装时报错Syntax Error，如何处理？](#section_ck4_e3u_b0d)
    -   [Mac上安装PyODPS时报错Permission Denied，如何处理？](#section_i9f_v10_z0x)
    -   [Mac上sudo安装PyODPS报错Operation Not Permitted，如何处理？](#section_ia8_b07_b7n)
    -   [PyODPS安装后执行import odps，报错No Module Named ODPS，如何处理？](#section_k6e_pjr_rlz)
    -   [PyODPS安装后执行from odps import \*，报错Cannot Import Name ODPS怎么处理？](#section_hlq_lc3_j78)
    -   [PyODPS安装后执行import odps，报错Cannot Import Module odps怎么处理？](#section_a6g_hv8_y1k)
    -   [PyODPS使用时报错sourceIP is not in the white list怎么处理？](#section_aiz_u7h_js9)
    -   [Jupyter前端UI报错，怎么处理？](#section_xnq_haq_ara)
-   开发常见问题：
    -   [调用Dataframe的head方法报错IndexError:listindexoutofrange是什么原因？](#section_4e1_vd7_9fw)
    -   [如何避免嵌套循环执行慢的情况？](#section_rgv_ous_03f)
    -   [什么情况下可以下载PyODPS数据到本地处理？](#section_aso_h65_gr6)
    -   [如何使用Pandas计算后端进行本地Debug？](#section_il2_iez_f8r)
    -   [如何利用Python语言特性来实现丰富的功能？](#section_mut_zhu_vnz)
    -   [为什么尽量使用内建算子，而不是自定义函数？](#section_xza_nty_m22)
    -   -   使用常见问题：
    -   [o.gettable\('table\_name'\).size中size字段的含义是什么？](#section_g1v_vnv_zjs)
    -   [如何制定Tunnel Endpoint？](#section_04w_fh4_ajs)
    -   [PyODPS如何使用包含cPython的第三方包？](#section_zga_grc_vy9)
    -   [PyODPS打印日志中，中文自动转化为编码显示，如何显示成原始中文？](#section_c3p_3su_p2j)
    -   [MaxCompute是否支持Python SQLAchemy访问？](#section_245_u1q_g06)
    -   [PyODPS中向表写入数据的两种方式open\_writer\(\)和write\_table\(\)有什么区别？](#section_y21_aqm_j23)
    -   [PyODPS中不支持Insert Overwrite的写入方式该如何处理？](#section_mkw_srn_j0n)
    -   [PyODPS中的DataFrame最多可以处理多少数据，对表的大小有限制吗？](#section_pk8_14h_ks4)
    -   [为什么DataWorks PyODPS节点上查出的数据量要少于本地运行的结果？](#section_azu_k57_ae4)
    -   [如何处理写数据时的脏数据报错？](#section_blu_cwh_71o)
    -   [如何处理读取数据时报错Project is protected？](#section_4o4_bhx_bdi)
    -   [在DataFrame中如何使用max\_pt？](#section_nfk_y3e_8yp)
    -   [如何处理在Ipython或者Jupyter下使用PyODPS时的报错ImportError？](#section_du9_frf_cqn)
    -   [如何处理上传Pandas DataFrame到MaxCompute时的报错ODPSError？](#section_92s_oa8_4en)
    -   [通过DataFrame写表时报错生命周期没有指定，怎么处理？](#section_g75_6mf_4yd)
    -   [执行SQL通过open\_reader最多只能取到1万条记录，如何获取多于1万条的记录？](#section_tst_7px_x3v)
    -   [为什么通过DataFrame\(\).schema.partitions获得分区表的分区值为空？](#section_2ln_393_u3l)
    -   [设置options.tunnel.use\_instance\_tunnel = False，为什么字段在odps中定义为DATETIME类型，使用SELECT语句得到的数据为STRING类型？](#section_meh_r7f_0hn)
    -   [PyODPS脚本任务不定时出现连接失败，报错ConnectionError: timed out try catch exception，是什么原因？](#section_xzl_zud_gf6)
    -   [DataFrame如何获得Count实际数字？](#section_hox_9d3_xni)
    -   [MaxCompute对Python是否支持？](#section_wr7_1ji_33b)
    -   [PyODPS节点是否支持Matplotlib等画图库？](#section_ii3_r3n_7p0)
    -   [使用from odps import options options.sql.settings设置MaxCompute运行环境不成功是什么原因？](#section_o1n_s6k_dw1)
    -   [在Shell或Python脚本中，如何执行MaxCompute命令？](#section_wmk_m2c_8x5)
    -   [在MaxCompute环境中安装高版本的Numpy包时报错，是什么原因？](#section_ufb_x8d_by6)
    -   [定义UDF的Python脚本中如何调用第三方库 ？](#section_l63_8x3_sg1)

## PyODPS安装时提示Warning: XXX not installed，如何处理？

报错原因为组件缺失，请参考报错信息中提示的“XXX”信息明确缺失的组件名称，使用`pip`命令安装此组件。

## PyODPS安装时提示Project Not Found，如何处理？

报错原因为：

-   Endpoint配置错误。Endpoint的配置请参见[配置Endpoint](/cn.zh-CN/准备工作/配置Endpoint.md) 。
-   MaxCompute入口对象参数位置填写错误。请检查此项确保其填写正确。

## PyODPS安装时报错Syntax Error，如何处理？

Python版本过低造成的报错。不支持Python2.5及以下版本，建议使用PyODPS主流支持的版本，例如Python2.7.6+、Python3.3+以及Python2.6。

## Mac上安装PyODPS时报错Permission Denied，如何处理？

使用`sudo pip install pyodps`命令安装。

## Mac上sudo安装PyODPS报错Operation Not Permitted，如何处理？

此报错由于系统完整性保护导致。 重启机器，并在重启中按 **⌘**+**R**键，此后在终端中运行如下命令可以解决此问题。

```
csrutil disable
reboot       
```

更多信息请参见[Operation Not Permitted when on root - El Capitan \(rootless disabled\)](https://stackoverflow.com/questions/32659348/operation-not-permitted-when-on-root-el-capitan-rootless-disabled)。

## PyODPS安装后执行import odps，报错No Module Named ODPS，如何处理？

此报错说明无法加载ODPS Package。无法加载的原因有如下几种：

-   安装了多个Python版本。

    Search Path（通常是当前目录）中包含`odps.py`或者`init.py`文件且名为odps的文件夹。解决方法如下：

    -   如果是文件夹重名，请修改文件夹名称。
    -   如果是曾经安装过一个名为odps的Python包，请使用`pip uninstall odps`进行删除。
-   同时安装了Python2和Python3版本。
-   同时安装了CPython、Anaconda和Miniconda，而当前使用的Python下并未安装PyODPS。

## PyODPS安装后执行from odps import \*，报错Cannot Import Name ODPS怎么处理？

请检查当前工作路径下是否存在名为**odps.py**的文件。若存在，请改名后再执行导入操作。

## PyODPS安装后执行import odps，报错Cannot Import Module odps怎么处理？

此报错通常是由于PyODPS遇到了依赖问题。请联系PyODPS技术支持钉钉群（11701793）。

## PyODPS使用时报错sourceIP is not in the white list怎么处理？

该项目空间存在白名单保护，请咨询项目Owner。

## Jupyter前端UI报错，怎么处理？

-   卸载并重新安装Jupyter、Ipywidgets以及Widgetsnbextension。
-   使用如下命令执行Bash操作。

    ```
    jupyter nbextension enable pyodps/main      
    ```


## 在Shell或Python脚本中，如何执行MaxCompute命令？

MaxCompute命令支持`-f`参数，在脚本或其它程序中直接以`odps -f <文件路径>`的方式读取MaxCompute命令文件。

也可以使用`-e`参数直接运行语句，使用`-s`参数运行脚本。

## 使用from odps import options options.sql.settings设置MaxCompute运行环境不成功是什么原因？

-   问题现象：使用PyODPS运行SQL，在申请MaxCompute实例前，通过如下代码设置MaxCompute运行环境。

    ```
    from odps import options
    options.sql.settings = {'odps.sql.mapper.split.size': 32}     
    ```

    运行任务后只启动了6个Mapper，set的设置并没有成功。 在客户端执行`set odps.stage.mapper.split.size=32`，一分钟运行完毕。

-   解决方法：客户端和PyODPS里设置的参数不一致。客户端的参数是`odps.stage.mapper.split.size`，而PyODPS里的参数是 `odps.sql.mapper.split.size`。

## PyODPS节点是否支持Matplotlib等画图库？

画图库需要GPU资源，当前DataWorks PyODPS节点不支持。

## MaxCompute对Python是否支持？

目前MaxCompute已经提供了Python版本的SDK支持。因为Python沙箱策略尚未成熟，出于安全因素考虑，暂不提供基于Python的UDF、MapReduce和Graph的SDK支持。

关于更多Python的帮助信息，请参见 [开源社区](https://github.com/aliyun/aliyun-odps-python-sdk)。

## DataFrame如何获得Count实际数字？

1.  在MaxCompute客户端上执行如下命令创建MaxCompute表来初始化DataFrame。

    ```
    iris = DataFrame(o.get_table('pyodps_iris'))        
    ```

2.  在DataFrame上执行Count获取DataFrame的总行数。

    ```
    iris.count()      
    ```

3.  由于DataFrame上的操作并不会立即执行，只有当用户显示调用Execute方法或者立即执行的方法时，才会真正执行。此时为了防止Count方法延迟执行，可输入如下命令。

    ```
    df.count().execute()    
    ```


获取DataFrame实际数量的相关方法请参见[聚合操作](/cn.zh-CN/开发/PyODPS/DataFrame/聚合操作.md)。详细的PyODPS方法延迟执行操作请参见[执行](/cn.zh-CN/开发/PyODPS/DataFrame/执行.md)。

## 调用Dataframe的head方法报错IndexError:listindexoutofrange是什么原因？

由于`list[index]`为空没有元素或`list[index]`超出范围。

## 如何避免嵌套循环执行慢的情况？

建议您通过Dict数据结构记录下循环的执行结果，最后在循环外统一导入到Dateframe对象中。如果您将Dateframe对象代码`df=XXX`放置在外层循环中，会导致每次循环计算都生成一个Dateframe对象，从而降低嵌套循环整体的执行速度。

## 什么情况下可以下载PyODPS数据到本地处理？

以下两种情况下可以下载Pyodps数据到本地：

-   数据量很小的情况下进行本地数据处理。
-   如果需要对单行数据应用一个Python函数，或者执行一行变多行的操作，这时使用PyODPS DataFrame就可以轻松完成，并且可以完全发挥MaxCompute的并行计算能力。

    例如有一份JSON串数据，需要把JSON串按Key-Value对展开成一行，代码如下所示。

    ```
    In [12]: df
                   json
    0  {"a": 1, "b": 2}
    1  {"c": 4, "b": 3}
    
    In [14]: from odps.df import output
    
    In [16]: @output(['k', 'v'], ['string', 'int'])
        ...: def h(row):
        ...:     import json
        ...:     for k, v in json.loads(row.json).items():
        ...:         yield k, v
        ...:   
    
    In [21]: df.apply(h, axis=1)
       k  v
    0  a  1
    1  b  2
    2  c  4
    3  b  3                          
    ```


## 如何使用Pandas计算后端进行本地Debug？

您可以利用以下两种方式来进行本地Debug。这两种方式除了初始化方法不同，后续代码完全一致：

-   通过Pandas DataFrame创建的PyODPS DataFrame可以使用Pandas执行本地计算。
-   使用MaxCompute表创建的DataFrame可以在MaxCompute上执行。

示例代码如下。

```
df = o.get_table('movielens_ratings').to_df()
DEBUG = True
if DEBUG:
    df = df[:100].to_pandas(wrap=True)       
```

当所有后续代码都编写完成，本地的测试速度非常快。当测试结束后，您可以把Debug改为False，这样后续就能在MaxCompute上执行全量的计算。

推荐您使用MaxCompute Studio来执行本地PyODPS程序调试。

## 如何利用Python语言特性来实现丰富的功能？

-   编写Python函数。

    计算两点之间的距离有多种计算方法，例如欧氏距离、曼哈顿距离等，您可以定义一系列函数，在计算时根据具体情况调用相应的函数即可。

    ```
    def euclidean_distance(from_x, from_y, to_x, to_y):
        return ((from_x - to_x) ** 2 + (from_y - to_y) ** 2).sqrt()
    
    def manhattan_distance(center_x, center_y, x, y):
       return (from_x - to_x).abs() + (from_y - to_y).abs()                      
    ```

    调用如下

    ```
    In [42]: df
         from_x    from_y      to_x      to_y
    0  0.393094  0.427736  0.463035  0.105007
    1  0.629571  0.364047  0.972390  0.081533
    2  0.460626  0.530383  0.443177  0.706774
    3  0.647776  0.192169  0.244621  0.447979
    4  0.846044  0.153819  0.873813  0.257627
    5  0.702269  0.363977  0.440960  0.639756
    6  0.596976  0.978124  0.669283  0.936233
    7  0.376831  0.461660  0.707208  0.216863
    8  0.632239  0.519418  0.881574  0.972641
    9  0.071466  0.294414  0.012949  0.368514
    
    In [43]: euclidean_distance(df.from_x, df.from_y, df.to_x, df.to_y).rename('distance')
       distance
    0  0.330221
    1  0.444229
    2  0.177253
    3  0.477465
    4  0.107458
    5  0.379916
    6  0.083565
    7  0.411187
    8  0.517280
    9  0.094420
    
    In [44]: manhattan_distance(df.from_x, df.from_y, df.to_x, df.to_y).rename('distance')
       distance
    0  0.392670
    1  0.625334
    2  0.193841
    3  0.658966
    4  0.131577
    5  0.537088
    6  0.114198
    7  0.575175
    8  0.702558
    9  0.132617                       
    ```

-   利用Python语言的条件和循环语句。

    如果用户要计算的表保存在数据库，需要根据配置来对表的字段进行处理，然后对所有表进行UNION或者JOIN操作。这时如果用SQL实现是相当复杂的，但是用DataFrame处理则会非常简单。

    例如，您有30张表需要合成一张表，此时如果使用SQL，则需要对30张表执行Union ALL操作。如果使用PyODPS，如下代码就可以完成。

    ```
    table_names = ['table1', ..., 'tableN']
    dfs = [o.get_table(tn).to_df() for tn in table_names]
    reduce(lambda x, y: x.union(y), dfs) 
    
    ## reduce语句等价于如下代码。
    df = dfs[0]
    for other_df in dfs[1:]:
        df = df.union(other_df)       
    ```


## 为什么尽量使用内建算子，而不是自定义函数？

计算过程中使用自定义函数比使用内建算子速度慢很多，因此建议使用内建算子。

对于百万行的数据，当一行应用了自定义函数后，执行时间从7秒延长到了27秒。如果有更大的数据集、更复杂的操作，时间的差距可能会更大。

## o.gettable\('table\_name'\).size中size字段的含义是什么？

SIZE字段表示表的物理存储大小。

## 如何制定Tunnel Endpoint？

通过`options.tunnel.endpoint`设置，详情请参见[aliyun-odps-python-sdk](https://github.com/aliyun/aliyun-odps-python-sdk/blob/master/docs/source/options.rst)。

## PyODPS如何使用包含cPython的第三方包？

建议打包成WHEEL格式后使用，可参见[如何制作可以在MaxCompute上使用的crcmod](https://yq.aliyun.com/articles/691754)。

## PyODPS打印日志中，中文自动转化为编码显示，如何显示成原始中文？

您可以使用类似 `print ("我叫 %s" % ('abc'))`输入方式解决 。目前仅Python2涉及此类问题。

## MaxCompute是否支持Python SQLAchemy访问？

不支持。

## PyODPS中向表写入数据的两种方式open\_writer\(\)和write\_table\(\)有什么区别？

每次调用`write_table()`，MaxCompute都会在服务端生成一个文件。这一操作需要较大的时间开销，同时过多的文件会降低后续的查询效率，还可能造成服务端内存不足。因此，建议在使用`write_table()`方法时，一次性写入多组数据或者传入一个Generator对象。

`open_writer()`默认写入到Block中。

## PyODPS中不支持Insert Overwrite的写入方式该如何处理？

您可以先使用`TRUNCATE`命令清空数据，然后使用普通WRITE方式写入数据。

## PyODPS中的DataFrame最多可以处理多少数据，对表的大小有限制吗？

PyODPS对表的大小没有限制。本地Pandas创建Dataframe的大小受限于本地内存的大小。

## 为什么DataWorks PyODPS节点上查出的数据量要少于本地运行的结果？

Dataworks上默认未开启Instance Tunnel，即`instance.open_reader`默认使用Result接口，最多可以获取一万条记录。

开启Instance Tunnel后，您可以通过`reader.count`获取记录数。如果您需要迭代获取全部数据，则需要通过设置`options.tunnel.limit_instance_tunnel = False`关闭Limit限制实现。

## 如何处理写数据时的脏数据报错？

-   详细报错信息如下。

    ```
    Invalid protobuf tag. Perhaps the datastream from server is crushed.
    ```

-   解决方法：通常这种报错是由脏数据导致，请您检查数据列数是否和目标表一致。

## 如何处理读取数据时报错Project is protected？

Project上的安全策略禁止读取表中的数据，如果想使用全部数据，可以使用以下方法：

-   联系Project Owner增加例外规则。
-   使用DataWorks或其他脱敏工具先对数据进行脱敏，导出到非保护Project，再进行读取。

如果只想查看部分数据，可使用如下方法：

-   改用`o.execute_sql('select * from <table_name>').open_reader()`。
-   改用`DataFrame，o.get_table('<table_name>').to_df()`。

## 在DataFrame中如何使用max\_pt？

使用`odps.df.func`模块来调用MaxCompute内建函数。

```
from odps.df import func
df = o.get_table('your_table').to_df()
df[df.ds == func.max_pt('your_project.your_table')]  # ds是分区字段。     
```

## 如何处理在Ipython或者Jupyter下使用PyODPS时的报错ImportError？

在代码头部增加`from odps import errors`。

如果增加`from odps import errors`也报错则是缺少Ipython组件，执行`pip install -U jupyter`解决此问题。

## 如何处理上传Pandas DataFrame到MaxCompute时的报错ODPSError？

-   问题现象：详细报错信息如下。

    ```
    ODPSError: ODPS entrance should be provided.
    ```

-   问题原因：报错原因为没有找到全局的ODPS入口
-   解决办法：
    -   使用Room机制`%enter`时，会配置全局入口。
    -   对ODPS入口调用`to_global`方法。
    -   使用ODPS参数`DataFrame(pd_df).persist('your_table', odps=odps)`。

## 通过DataFrame写表时报错生命周期没有指定，怎么处理？

-   问题现象：详细报错信息如下。

    ```
    table lifecycle is not specified in mandatory mode
    ```

-   解决办法：Project要求对每张表设置生命周期，因此需要在每次执行时设置如下信息即可。

    ```
    from odps import options
    options.lifecycle = 7  # 此处设置lifecycle的值。lifecycle取值为整数，单位为天。      
    ```


## 执行SQL通过open\_reader最多只能取到1万条记录，如何获取多于1万条的记录？

使用`create table as select ...`将SQL的结果保存成表，再使用 `table.open_reader`读取。

## 为什么通过DataFrame\(\).schema.partitions获得分区表的分区值为空？

这是因为DataFrame不区分分区字段和普通字段，所以获取分区表的分区字段作为普通字段处理。您可以通过如下方式过滤掉分区字段。

```
df = o.get_table().to_df()
df[df.ds == '']       
```

建议您参考[表](/cn.zh-CN/开发/PyODPS/基本操作/表.md)来设置分区或读取分区信息。

## 设置options.tunnel.use\_instance\_tunnel = False，为什么字段在odps中定义为DATETIME类型，使用SELECT语句得到的数据为STRING类型？

在调用Open\_Reader时，PyODPS会默认调用旧的Result接口。此时从服务端得到的数据是CSV格式的，所以DATATIME都是STRING类型。

打开Instance Tunnel，即设置`options.tunnel.use_instance_tunnel = True`，PyODPS会默认调用Instance Tunnel，即可解决此问题。

## PyODPS脚本任务不定时出现连接失败，报错ConnectionError: timed out try catch exception，是什么原因？

此报错可能原因如下：

-   建立连接超时。PyODPS默认的超时时间是5s，解决方法如下：
    -   可以在代码头部加上如下代码，增加超时时间隔。

        ```
        workaround from odps import options 
        options.connect_timeout=30                        
        ```

    -   捕获异常，进行重试。
-   由于沙箱限制，会造成部分机器禁止网络访问。建议您使用独享调度资源组执行任务，解决此问题。

## 在DataWorks上执行PyODPS，运行时报错Got killed，怎么处理？

-   问题现象：运行时报错如下。

    ```
    ERROR:pyodps.pyodpswrapper:Got killed.
    ```

-   问题原因：在DataWorks上使用PyODPS时，为了防止对DataWorks的Gate Way造成压力，DataWorks对运行时的内存和CPU都有限制。该限制由DataWorks统一管理。PyODPS使用限制，请参见[使用限制](/cn.zh-CN/开发/PyODPS/工具平台使用指南/在DataWorks上使用PyODPS.md)。

    **说明：** 通过PyODPS发起的SQL和DataFrame任务（除to\_pandas外）不受此限制。

-   解决办法：如果您发现有Got killed报错，即内存使用超限，进程被中止。请尽量避免本地的数据操作规避此问题。

## 在MaxCompute环境中安装高版本的Numpy包时报错，是什么原因？

-   问题现象：在MaxCompute环境中安装高版本的Numpy包时，报错如下。

    ```
    ImportError: this version of pandas is incompatible with numpy = 1.12.0 to use this pandas version.
    ```

-   问题原因：仅支持安装环境中不存在的Numpy版本，并且要求为Numpy原始版本，即后缀为`***--cp27-cp27m-manylinux1_x86_64.whl`的包。

## 定义UDF的Python脚本中如何调用第三方库 ？

在DataWorks上将第三方包添加为资源即可直接引用。

## Pyodps运行get\_sql\_task\_cost函数报错是什么原因？

-   问题现象：运行时报错如下。

    ```
    NameError: name 'get_task_cost' is not defined.
    ```

-   解决方案：此场景下需要使用execute\_sql\_cost替代get\_sql\_task\_cost。

## DataWorks上的PyODPS节点是否支持Python 3？

DataWorks上的PyODPS节点支持Python 2和Python 3，详情请参见[创建PyODPS 2节点]()和[创建PyODPS 3节点]()。

