---
keyword: [Design a workflow, Zero-load node, ODPS SQL node]
---

# Design workflows

You can design a workflow to arrange nodes to be run during data analytics. This topic describes how to design a workflow in which each node corresponds to a table at a data warehouse layer.

1.  Go to the DataStudio page.

    1.  Log on to the DataWorks console. Click [Workspaces](https://workbench.data.aliyun.com/consolenew#/projectlist) in the left-side navigation pane. On the page that appears, select **China \(Shanghai\)** in the top navigation bar.

    2.  Find the target workspace and click **Data Analytics** in the Actions column.

2.  Go to the DataStudio page. On the Data Analytics tab, double-click the Workshop workflow. The canvas for editing the workflow appears on the right.

3.  Drag Zero-Load Node to the canvas. In the Create Node dialog box that appears, set Node Name to start and click Commit.

4.  Drag ODPS SQL to the canvas. In the Create Node dialog box that appears, set Node Name to ods\_user\_trace\_log and click Commit. Use the same method to create another two nodes and name them dw\_user\_trace\_log and rpt\_user\_trace\_log, respectively.

    **Note:** The ods\_user\_trace\_log, dw\_user\_trace\_log, and rpt\_user\_trace\_log nodes represent the operational data store \(ODS\), common data model \(CDM\), and application data store \(ADS\) layers of a data warehouse, respectively. For more information, see [Divide a data warehouse into layers]().


