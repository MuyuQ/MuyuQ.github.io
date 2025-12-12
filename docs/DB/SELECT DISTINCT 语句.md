# SQL学习：掌握INNER JOIN、DISTINCT和HAVING关键字

在数据库查询中，掌握各种SQL关键字对于高效检索和处理数据至关重要。本文将详细介绍INNER JOIN、DISTINCT和HAVING这三个常用的SQL关键字，帮助你更好地理解和应用它们。


## INNER JOIN关键字

INNER JOIN关键字用于返回两个表中字段匹配关系的记录。它只返回两个表中满足连接条件的记录。

### 基本语法

```sql
SELECT column_name(s)
FROM table1
INNER JOIN table2
ON table1.column_name = table2.column_name;
```

### 实例说明

以下示例展示了如何使用INNER JOIN获取网站名称及其访问日志：

```sql
SELECT Websites.name, access_log.count, access_log.date
FROM Websites
INNER JOIN access_log
ON Websites.id = access_log.site_id
ORDER BY access_log.count;
```

在这个例子中：
- `Websites`和`access_log`是两个要连接的表
- `ON Websites.id = access_log.site_id`是连接条件
- 结果集包含两个表中满足连接条件的所有记录

## ON和WHERE的区别

在SQL查询中，ON和WHERE子句都用于过滤数据，但它们的作用时机和目的不同：

- **ON子句**：在生成临时表时就会进行条件对比，用于确定哪些记录应该被连接
- **WHERE子句**：在临时表生成之后再进行过滤，用于进一步筛选结果

理解这个区别有助于优化查询性能和获得预期的结果。

## SELECT DISTINCT语句

在实际的数据表中，一个列可能会包含多个重复值。当我们只需要列出不同的值时，可以使用DISTINCT关键字。

### 基本语法

```sql
SELECT DISTINCT column_name
FROM table_name;
```

### 实例说明

以下示例展示了如何使用DISTINCT获取不重复的部门名称：

```sql
SELECT DISTINCT adep
FROM ATHLETE;
```

如果需要结合聚合函数使用DISTINCT，可以参考以下示例：

```sql
SELECT adep, SUM(score) 
FROM ATHLETE
INNER JOIN SCORE
ON ATHLETE.Aneo = SCORE.Aneo
GROUP BY adep;
```

## HAVING关键字

WHERE关键字无法与聚合函数一起使用，而HAVING关键字正是为了解决这个问题。HAVING子句用于过滤分组后的结果。

### 基本语法

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value;
```

### 实例说明

以下示例展示了如何使用HAVING筛选总访问量超过200的网站：

```sql
SELECT Websites.name, SUM(access_log.count) AS total_access
FROM Websites
INNER JOIN access_log
ON Websites.id = access_log.site_id
GROUP BY Websites.name
HAVING SUM(access_log.count) > 200
ORDER BY total_access DESC;
```

在这个例子中：
- 使用`GROUP BY`按网站名称分组
- 使用`HAVING`筛选总访问量超过200的记录
- 最终结果按照总访问量降序排列

## 总结

通过掌握INNER JOIN、DISTINCT和HAVING这些关键字，我们可以更灵活地处理数据库查询需求：

- 使用INNER JOIN连接多个表获取关联数据
- 使用DISTINCT去除重复记录获取唯一值
- 使用HAVING对分组后的数据进行筛选

合理运用这些关键字能够显著提升SQL查询的效率和准确性。
