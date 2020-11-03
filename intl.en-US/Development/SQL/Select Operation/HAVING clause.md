---
keyword: [HAVING clause, aggregate function]
---

# HAVING clause

In MaxCompute SQL, the WHERE keyword cannot be used together with aggregate functions. In this case, you can use the HAVING clause.

Write an SQL SELECT statement that contains a HAVING clause in the following syntax:

```
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
```

For example, a table named `Orders` contains the following fields: `Customer`, `OrderPrice`, `Order_date`, and `Order_id`. If you want to search for customers whose total order amount is less than 2,000, you can execute the following SQL statement:

```
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice)<2000
```

