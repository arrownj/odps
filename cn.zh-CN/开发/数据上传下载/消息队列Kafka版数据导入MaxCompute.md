# 消息队列Kafka版数据导入MaxCompute

本文为您介绍如何将消息队列Kafka版数据导入MaxCompute。

## 背景信息

[消息队列Kafka版](/cn.zh-CN/产品简介/什么是消息队列Kafka版？.md)是阿里云基于Apache Kafka构建的高吞吐量、高可扩展性的分布式消息队列服务，广泛用于日志收集、监控数据聚合、流式数据处理、在线和离线分析等，是大数据生态中不可或缺的产品之一，阿里云提供全托管服务，用户无需部署运维，更专业、更可靠、更安全。

MaxCompute与消息队列Kafka版服务紧密集成，借助消息队列Kafka版服务的MaxCompute Sink Connector，无需第三方工具及二次开发，即可满足将指定Topic数据持续导入MaxCompute数据表的需求。极大简化Kafka消息队列数据进入MaxCompute的集成链路，并显著降低开发和运维成本。

## 操作方法

您需要创建MaxCompute Sink Connector，并将数据从消息队列Kafka版实例的数据源Topic导出至MaxCompute的表，操作详情请参见[创建MaxCompute Sink Connector](/cn.zh-CN/用户指南/Connector/创建Connector/创建MaxCompute Sink Connector.md)。

