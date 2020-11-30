---
keyword: switch billing methods
---

# Switch billing methods

MaxCompute allows you to switch between the subscription and pay-as-you-go billing methods. This topic describes how to switch the billing method of a project by changing its quota group.

Both pay-as-you-go and subscription resources of MaxCompute are purchased.

The differences between subscription and pay-as-you-go billing methods are only in the billing and running modes of computing resources. The billing methods for storage and download resources are the same.

The computing jobs in a subscription project can use only the purchased subscription computing resources. Subscription computing resources are exclusive resources, and pay-as-you-go resources are shared resources. Computing resources are CU resources.

The following table describes the scenarios in which you switch billing methods.

|Scenario|Description|Effective period|
|--------|-----------|----------------|
|Switch the billing method from pay-as-you-go to subscription|Supported. You must purchase CUs of MaxCompute before you switch the billing method. You are not allowed to switch the billing method for a project across regions.|The new billing method takes effect immediately. However, if a job is running, the new billing method does not take effect until the next time you run the job.|
|Switch the billing method from subscription to pay-as-you-go|Supported. The fees you already paid are not refunded. However, you can still create projects to use the purchased CUs. After you purchase MaxCompute CUs, you can create multiple projects to share these CUs.|

**Note:** We recommend that you do not frequently switch billing methods because this may affect the running time of your computing jobs.

1.  Log on to the [MaxCompute console](https://workbench-intl.data.aliyun.com/consolenew#/MCEngines), and select a region in the upper-left corner.

2.  Find the MaxCompute project for which you want to switch the billing method, and click **Switch quota group** in the Actions column to go to the **Workspace Management** page.

3.  In the **Compute Engine information** section, click the **MaxCompute** tab and modify Quota group switching.

    ![Switch the quota group](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/en-US/4888176061/p180602.png)

    -   If you switch the billing method from pay-as-you-go to subscription for the project, select a subscription quota group.
    -   If you switch the billing method from subscription to pay-as-you-go for the project, select Default pay-as-you-go quota.

        **Note:** If the workspace that corresponds to a project is in standard mode, you need only to switch the billing method for the production project. The billing methods for the development and production projects are the same. If you switch the billing method from pay-as-you-go to subscription, you can switch the subscription quota groups separately for the two projects by using [Use MaxCompute Management](/intl.en-US/Management/Resource and job management/MaxCompute Management.md). For more information, see [Change the quota group of a project](/intl.en-US/Management/Resource and job management/MaxCompute Management.mdsection_biq_anv_gnl).


