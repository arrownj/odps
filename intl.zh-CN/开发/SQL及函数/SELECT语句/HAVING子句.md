---
keyword: [HAVING子句, 聚合函数]
---

# HAVING子句

MaxCompute SQL的WHERE关键字无法与聚合函数一起使用，此时您可以使用HAVING子句来实现。

命令格式如下。

```
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
```

例如有一张订单表`Orders`，包括客户名称（`Customer`）、订单金额（`OrderPrice`）、订单日期（`Order_date`）和订单号（`Order_id`）四个字段。现在需要查找订单总额少于2000的客户，SQL语句如下。

```
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice)<2000
```

