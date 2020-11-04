---
keyword: [region, endpoint, tunnel endpoint]
---

# Configure endpoints

This topic describes the regions where MaxCompute is available and MaxCompute connection methods. This topic also describes issues arising from use with other Alibaba Cloud services such as Elastic Compute Service \(ECS\), Tablestore, and Object Storage Service \(OSS\).

![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/9642659951/p1423.png)

MaxCompute provides two types of endpoints:

-   MaxCompute endpoint: used to send all requests except for data upload and download requests to MaxCompute. For example, you can send a request to create a table, delete a function, or create a job.
-   MaxCompute Tunnel endpoint: used to upload data to and download data from MaxCompute. To upload and download data through MaxCompute Tunnel, you can initiate a request by using the MaxCompute Tunnel endpoint.

    **Note:**

    -   Downloading data through MaxCompute Tunnel is charged at different prices in different regions, depending on the specific deployment and network connection status of MaxCompute Tunnel.
    -   If your Tunnel endpoint needs to connect over the internal network, you must configure an internal MaxCompute endpoint. Otherwise, traffic may be routed to the Internet and you may be charged for downloads over the Internet.

## Connection methods and billing rules for data downloads

You can access MaxCompute and MaxCompute Tunnel by using the following connection methods:

-   Internet
-   Alibaba Cloud classic network
-   Alibaba Cloud Virtual Private Cloud \(VPC\)

**Note:** You only need to specify a network when you want to connect to a project. You do not need to specify a network when you create a project.

Data uploads

Uploading data through MaxCompute Tunnel is free of charge, as shown in the preceding figure.

Data downloads

When you download data from MaxCompute to ECS instances in any region through MaxCompute Tunnel, the billing rules depend on whether the source and destination are in the same region:

-   If the source and destination are in the same region, downloading data through MaxCompute Tunnel is free of charge over both the classic network and VPC.
-   If the source and destination are not in the same region or cannot connect in the same region, data is downloaded through MaxCompute Tunnel over the Internet. You are charged for data downloads over the Internet.

    **Note:** The deployment and network connection status of MaxCompute Tunnel may vary with regions. Therefore, we cannot guarantee permanent connectivity for MaxCompute Tunnel if you select the Alibaba Cloud classic network or VPC.


## Connectivity configurations for accessing external tables

MaxCompute 2.0 supports reading and writing OSS data and Tablestore data. For more information, see [Access OSS unstructured data](/intl.en-US/Development/External table/Access OSS data by using the built-in extractor.md) and [Access Tablestore unstructured data](/intl.en-US/Development/External table/Access Table Store data.md).

Note the following points when you configure network connectivity:

-   If MaxCompute is in the same region as Tablestore or OSS, we recommend that you select the Alibaba Cloud classic network or VPC connection method. You can also select the Internet connection method.
-   If MaxCompute is not in the same region as Tablestore or OSS, we recommend that you select the Internet connection method. If you select the Alibaba Cloud classic network or VPC connection method, we do not guarantee network connectivity.

## Regions where MaxCompute is available and corresponding endpoints

MaxCompute is available in regions both inside and outside mainland China. You can apply for MaxCompute in a desired region. Your data storage and computing resources are consumed in this region.

**Note:** Both HTTP and HTTPS are supported in endpoints \(aliyun\). If you need to encrypt your requests, use HTTPS.

-   Regions and endpoints for the Internet connection method

    |Region|City|MaxCompute available or not|Public endpoint|Public Tunnel endpoint|
    |:-----|:---|:--------------------------|:--------------|:---------------------|
    |China \(Hangzhou\)|Hangzhou|Available|http://service.cn-hangzhou.maxcompute.aliyun.com/api|http://dt.cn-hangzhou.maxcompute.aliyun.com|
    |China \(Shanghai\)|Shanghai|Available|http://service.cn-shanghai.maxcompute.aliyun.com/api|http://dt.cn-shanghai.maxcompute.aliyun.com|
    |China East 2 Finance|Shanghai|Available|http://service.odps-cn-shanghai-finance-1f.maxcompute.aliyun.com/api|http://dt.odps-cn-shanghai-finance-1f.maxcompute.aliyun.com|
    |China \(Beijing\)|Beijing|Available|http://service.cn-beijing.maxcompute.aliyun.com/api|http://dt.cn-beijing.maxcompute.aliyun.com|
    |China \(Zhangjiakou\)|Zhangjiakou|Available|http://service.cn-zhangjiakou.maxcompute.aliyun.com/api|http://dt.cn-zhangjiakou.maxcompute.aliyun.com|
    |China \(Shenzhen\)|Shenzhen|Available|http://service.cn-shenzhen.maxcompute.aliyun.com/api|http://dt.cn-shenzhen.maxcompute.aliyun.com|
    |China \(Hong Kong\)|Hong Kong|Available|http://service.cn-hongkong.maxcompute.aliyun.com/api|http://dt.cn-hongkong.maxcompute.aliyun.com|
    |Singapore|Singapore|Available|http://service.ap-southeast-1.maxcompute.aliyun.com/api|http://dt.ap-southeast-1.maxcompute.aliyun.com|
    |Australia \(Sydney\)|Sydney|Available|http://service.ap-southeast-2.maxcompute.aliyun.com/api|http://dt.ap-southeast-2.maxcompute.aliyun.com|
    |Malaysia \(Kuala Lumpur\)|Kuala Lumpur|Available|http://service.ap-southeast-3.maxcompute.aliyun.com/api|http://dt.ap-southeast-3.maxcompute.aliyun.com|
    |Indonesia \(Jakarta\)|Jakarta|Available|http://service.ap-southeast-5.maxcompute.aliyun.com/api|http://dt.ap-southeast-5.maxcompute.aliyun.com|
    |Japan \(Tokyo\)|Tokyo|Available|http://service.ap-northeast-1.maxcompute.aliyun.com/api|http://dt.ap-northeast-1.maxcompute.aliyun.com|
    |Germany \(Frankfurt\)|Frankfurt|Available|http://service.eu-central-1.maxcompute.aliyun.com/api|http://dt.eu-central-1.maxcompute.aliyun.com|
    |US \(Silicon Valley\)|Silicon Valley|Available|http://service.us-west-1.maxcompute.aliyun.com/api|http://dt.us-west-1.maxcompute.aliyun.com|
    |US \(Virginia\)|Virginia|Available|http://service.us-east-1.maxcompute.aliyun.com/api|http://dt.us-east-1.maxcompute.aliyun.com|
    |India \(Mumbai\)|Mumbai|Available|http://service.ap-south-1.maxcompute.aliyun.com/api|http://dt.ap-south-1.maxcompute.aliyun.com|
    |UAE \(Dubai\)|Dubai|Available|http://service.me-east-1.maxcompute.aliyun.com/api|http://dt.me-east-1.maxcompute.aliyun.com|
    |UK \(London\)|London|Available|http://service.eu-west-1.maxcompute.aliyun.com/api|http://dt.eu-west-1.maxcompute.aliyun.com|

-   Regions and endpoints for the classic network connection method

    |Region|City|MaxCompute available or not|Endpoint|Classic Tunnel endpoint|
    |------|----|---------------------------|--------|-----------------------|
    |China \(Hangzhou\)|Hangzhou|Available|http://service.cn-hangzhou.maxcompute.aliyun-inc.com/api|http://dt.cn-hangzhou.maxcompute.aliyun-inc.com|
    |China \(Shanghai\)|Shanghai|Available|http://service.cn-shanghai.maxcompute.aliyun-inc.com/api|http://dt.cn-shanghai.maxcompute.aliyun-inc.com|
    |China East 2 Finance|Shanghai|Available|http://service.odps-cn-shanghai-finance-1f.maxcompute.aliyun-inc.com/api|http://dt.odps-cn-shanghai-finance-1f.maxcompute.aliyun-inc.com|
    |China \(Beijing\)|Beijing|Available|http://service.cn-beijing.maxcompute.aliyun-inc.com/api|http://dt.cn-beijing.maxcompute.aliyun-inc.com|
    |China \(Zhangjiakou\)|Zhangjiakou|Available|http://service.cn-zhangjiakou.maxcompute.aliyun-inc.com/api|http://dt.cn-zhangjiakou.maxcompute.aliyun-inc.com|
    |China \(Shenzhen\)|Shenzhen|Available|http://service.cn-shenzhen.maxcompute.aliyun-inc.com/api|http://dt.cn-shenzhen.maxcompute.aliyun-inc.com|
    |China \(Hong Kong\)|Hong Kong|Available|http://service.cn-hongkong.maxcompute.aliyun-inc.com/api|http://dt.cn-hongkong.maxcompute.aliyun-inc.com|
    |Singapore|Singapore|Available|http://service.ap-southeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-1.maxcompute.aliyun-inc.com|
    |Australia \(Sydney\)|Sydney|Available|http://service.ap-southeast-2.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-2.maxcompute.aliyun-inc.com|
    |Malaysia \(Kuala Lumpur\)|Kuala Lumpur|Available|http://service.ap-southeast-3.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-3.maxcompute.aliyun-inc.com|
    |Indonesia \(Jakarta\)|Jakarta|Available|http://service.ap-southeast-5.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-5.maxcompute.aliyun-inc.com|
    |Japan \(Tokyo\)|Tokyo|Available|http://service.ap-northeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-northeast-1.maxcompute.aliyun-inc.com|
    |Germany \(Frankfurt\)|Frankfurt|Available|http://service.eu-central-1.maxcompute.aliyun-inc.com/api|http://dt.eu-central-1.maxcompute.aliyun-inc.com|
    |US \(Silicon Valley\)|Silicon Valley|Available|http://service.us-west-1.maxcompute.aliyun-inc.com/api|http://dt.us-west-1.maxcompute.aliyun-inc.com|
    |US \(Virginia\)|Virginia|Available|http://service.us-east-1.maxcompute.aliyun-inc.com/api|http://dt.us-east-1.maxcompute.aliyun-inc.com|
    |India \(Mumbai\)|Mumbai|Available|http://service.ap-south-1.maxcompute.aliyun-inc.com/api|http://dt.ap-south-1.maxcompute.aliyun-inc.com|
    |UAE \(Dubai\)|Dubai|Available|http://service.me-east-1.maxcompute.aliyun-inc.com/api|http://dt.me-east-1.maxcompute.aliyun-inc.com|
    |UK \(London\)|London|Available|http://service.uk-all.maxcompute.aliyun-inc.com/api|http://dt.uk-all.maxcompute.aliyun-inc.com|

-   Regions and endpoints for the VPC connection method

    You can only use the following endpoints and Tunnel endpoints when you select the VPC connection method.

    |Region|City|MaxCompute available or not|Endpoint|VPC Tunnel endpoint|
    |:-----|:---|:--------------------------|:-------|:------------------|
    |China \(Hangzhou\)|Hangzhou|Available|http://service.cn-hangzhou.maxcompute.aliyun-inc.com/api|http://dt.cn-hangzhou.maxcompute.aliyun-inc.com|
    |China \(Shanghai\)|Shanghai|Available|http://service.cn-shanghai.maxcompute.aliyun-inc.com/api|http://dt.cn-shanghai.maxcompute.aliyun-inc.com|
    |China East 2 Finance|Shanghai|Available|http://service.odps-cn-shanghai-finance-1f.maxcompute.aliyun-inc.com/api|http://dt.odps-cn-shanghai-finance-1f.maxcompute.aliyun-inc.com|
    |China \(Beijing\)|Beijing|Available|http://service.cn-beijing.maxcompute.aliyun-inc.com/api|http://dt.cn-beijing.maxcompute.aliyun-inc.com|
    |China \(Zhangjiakou\)|Zhangjiakou|Available|http://service.cn-zhangjiakou.maxcompute.aliyun-inc.com/api|http://dt.cn-zhangjiakou.maxcompute.aliyun-inc.com|
    |China \(Shenzhen\)|Shenzhen|Available|http://service.cn-shenzhen.maxcompute.aliyun-inc.com/api|http://dt.cn-shenzhen.maxcompute.aliyun-inc.com|
    |China \(Hong Kong\)|Hong Kong|Available|http://service.cn-hongkong.maxcompute.aliyun-inc.com/api|http://dt.cn-hongkong.maxcompute.aliyun-inc.com|
    |Singapore|Singapore|Available|http://service.ap-southeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-1.maxcompute.aliyun-inc.com|
    |Australia \(Sydney\)|Sydney|Available|http://service.ap-southeast-2.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-2.maxcompute.aliyun-inc.com|
    |Malaysia \(Kuala Lumpur\)|Kuala Lumpur|Available|http://service.ap-southeast-3.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-3.maxcompute.aliyun-inc.com|
    |Indonesia \(Jakarta\)|Jakarta|Available|http://service.ap-southeast-5.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-5.maxcompute.aliyun-inc.com|
    |Japan \(Tokyo\)|Tokyo|Available|http://service.ap-northeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-northeast-1.maxcompute.aliyun-inc.com|
    |Germany \(Frankfurt\)|Frankfurt|Available|http://service.eu-central-1.maxcompute.aliyun-inc.com/api|http://dt.eu-central-1.maxcompute.aliyun-inc.com|
    |US \(Silicon Valley\)|Silicon Valley|Available|http://service.us-west-1.maxcompute.aliyun-inc.com/api|http://dt.us-west-1.maxcompute.aliyun-inc.com|
    |US \(Virginia\)|Virginia|Available|http://service.us-east-1.maxcompute.aliyun-inc.com/api|http://dt.us-east-1.maxcompute.aliyun-inc.com|
    |India \(Mumbai\)|Mumbai|Available|http://service.ap-south-1.maxcompute.aliyun-inc.com/api|http://dt.ap-south-1.maxcompute.aliyun-inc.com|
    |UAE \(Dubai\)|Dubai|Available|http://service.me-east-1.maxcompute.aliyun-inc.com/api|http://dt.me-east-1.maxcompute.aliyun-inc.com|
    |UK \(London\)|London|Available|http://service.uk-all.maxcompute.aliyun-inc.com/api|http://dt.uk-all.maxcompute.aliyun-inc.com|


**Note:** You must specify the MaxCompute endpoint and Tunnel endpoint in the following scenarios:

-   Install and configure the MaxCompute client \(console\). For more information, see [Install and configure the odpscmd client](/intl.en-US/Prepare/Install and configure the odpscmd client.md).
-   Use MaxCompute Studio to connect to a MaxCompute project. For more information, see [Manage project connections](/intl.en-US/Tools and Downloads/MaxCompute Studio/Manage project connections.md).
-   Use MaxCompute SDKs. For more information, see MaxCompute connection configurations in [Java SDK](/intl.en-US/SDK Reference/Java SDK/Java SDK.md) and [SDK for Python](/intl.en-US/SDK Reference/Python SDK/SDK for Python.md).
-   Use the code editor of DataWorks Data Integration or the open source DataX to connect to MaxCompute data sources. For more information, see [Configure a MaxCompute connection]() and [Export SQL execution results](/intl.en-US/Best Practices/SQL/Export SQL execution results.md).

## Access rules

-   In the regions where MaxCompute is available, you can connect to MaxCompute over the Internet, classic network, or VPC.
-   When you use the Tunnel endpoint to download data over the Internet, you are charged USD 0.1166 for each GB of downloaded data.

