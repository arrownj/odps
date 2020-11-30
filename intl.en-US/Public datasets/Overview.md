---
keyword: public dataset of MaxCompute
---

# Overview

If you have activated MaxCompute, you can use the MaxCompute query editor to obtain and query the tables from public datasets. This helps you quickly get started with MaxCompute. This topic describes the public datasets of MaxCompute and how to use the MaxCompute query editor to query and analyze data in the public datasets.

The open data of MaxCompute mainly refers to the data in the datasets of the click rate predictions for ads displayed on Taobao.com. The datasets are provided by Alibaba Group. For more information about the fields in the datasets, see [Tianchi dataset](https://tianchi.aliyun.com/dataset/dataDetail?dataId=56#). The data is stored in the MAXCOMPUTE\_PUBLIC\_DATA project of MaxCompute.

## Disclaimer

Data in the public datasets of MaxCompute is only for product testing. The data is not periodically updated and its accuracy is not ensured. Therefore, do not use the data in the production process.

## Usage notes

You can authorize all MaxCompute users to access the public datasets by using a special authorization mechanism of MaxCompute. When you use the public datasets, take note of the following items:

-   All data is stored in the public MaxCompute project MAXCOMPUTE\_PUBLIC\_DATA. No MaxCompute users belong to this project. Therefore, when you compile an SQL script to access the public datasets, you must specify the project name before the table name. The following statement shows an example:

    ```
    SELECT * FROM MAXCOMPUTE_PUBLIC_DATA.raw_sample limit 10;
    ```

    **Note:** You can view the data in the public datasets free of charge. However, **you are charged when you execute query statements**. For more information about billing rules,see [Computing pricing](/intl.en-US/Pricing/Computing pricing.md).

-   You cannot find the tables in the public datasets on the **Data Map** page of DataWorks because cross-project access is required.

## Public datasets

The following tables describe the details of each public dataset in the MAXCOMPUTE\_PUBLIC\_DATA project.

-   Raw samples

    Raw samples consist of the ad click logs within an eight-day period of more than one million users that are randomly sampled from Taobao.com.

    |Project name|MAXCOMPUTE\_PUBLIC\_DATA|
    |Table name|raw\_sample|
    |Update cycle|Fixed data is provided and is not updated.|
    |Schema query|`DESC MAXCOMPUTE_PUBLIC_DATA.table_name;`|
    |Query example|`SELECT * FROM MAXCOMPUTE_PUBLIC_DATA.raw_sample limit 10;`|

-   Basic ad information

    This dataset contains basic information about some ads in the raw\_sample table.

    |Project name|MAXCOMPUTE\_PUBLIC\_DATA|
    |Table name|ad\_feature|
    |Update cycle|Fixed data is provided and is not updated.|
    |Schema query|`DESC MAXCOMPUTE_PUBLIC_DATA.table_name;`|
    |Query example|`SELECT * FROM MAXCOMPUTE_PUBLIC_DATA.ad_feature limit 10;`|

-   Basic user information

    This dataset contains basic information about all users in the raw\_sample table.

    |Project name|MAXCOMPUTE\_PUBLIC\_DATA|
    |Table name|user\_profile|
    |Update cycle|Fixed data is provided and is not updated.|
    |Schema query|`DESC MAXCOMPUTE_PUBLIC_DATA.table_name;`|
    |Query example|`SELECT * FROM MAXCOMPUTE_PUBLIC_DATA.user_profile limit 10;`|

-   User behavior logs

    This dataset contains the shopping behavior of all users in the raw\_sample table within a 22-day period.

    |Project name|MAXCOMPUTE\_PUBLIC\_DATA|
    |Table name|behavior\_log|
    |Update cycle|Fixed data is provided and is not updated.|
    |Schema query|`DESC MAXCOMPUTE_PUBLIC_DATA.table_name;`|
    |Query example|`SELECT * FROM MAXCOMPUTE_PUBLIC_DATA.behavior_log limit 10;`|


