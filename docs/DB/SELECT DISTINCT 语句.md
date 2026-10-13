# SELECT DISTINCT 语句

本文介绍 INNER JOIN、DISTINCT 和 HAVING 三个 SQL 关键字的用法。


## INNER JOIN 关键字

INNER JOIN 关键字用于返回两个表中满足连接条件的匹配记录。

### 基本语法

```sql
SELECT column_name(s)
FROM table1
INNER JOIN table2
ON table1.column_name = table2.column_name;
```

### 实例说明

以下示例展示了如何使用 INNER JOIN 获取网站名称及其访问日志：

```sql
SELECT Websites.name, access_log.count, access_log.date
FROM Websites
INNER JOIN access_log
ON Websites.id = access_log.site_id
ORDER BY access_log.count;
```

在这个例子中：
- `Websites` 和 `access_log` 是两个要连接的表
- `ON Websites.id = access_log.site_id` 是连接条件
- 结果集包含两个表中满足连接条件的所有记录

## ON 与 WHERE 的区别

在 SQL 查询中，ON 和 WHERE 子句都用于过滤数据，但它们的作用时机和目的不同：

- **ON子句**：在生成临时表时就会进行条件对比，用于确定哪些记录应该被连接
- **WHERE子句**：在临时表生成之后再进行过滤，用于进一步筛选结果

理解这个区别有助于优化查询性能和获得预期的结果。对 INNER JOIN 而言，条件写在 ON 还是 WHERE 中，查询结果相同；这一时机差异主要体现在 LEFT JOIN、RIGHT JOIN 等外连接上。

## SELECT DISTINCT 语句

在实际的数据表中，某一列可能存在重复值。当我们只需要列出不同的值时，可以使用 DISTINCT 关键字。

### 基本语法

```sql
SELECT DISTINCT column_name
FROM table_name;
```

### 实例说明

以下示例展示了如何使用 DISTINCT 获取不重复的部门名称：

```sql
SELECT DISTINCT adep
FROM ATHLETE;
```

如果需要与聚合函数结合使用 DISTINCT，可以参考以下示例：

```sql
SELECT adep, SUM(DISTINCT score)
FROM ATHLETE
INNER JOIN SCORE
ON ATHLETE.Aneo = SCORE.Aneo
GROUP BY adep;
```

`SUM(DISTINCT score)` 会对每个部门内不同的 `score` 值求和，重复的成绩只计一次。

## HAVING 关键字

WHERE 子句无法与聚合函数一起使用，HAVING 子句正是为解决这一问题而设计的，专门用于对分组后的结果进行筛选。

### 基本语法

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value;
```

### 实例说明

以下示例展示了如何使用 HAVING 筛选总访问量超过200的网站：

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
- 使用 `GROUP BY` 按网站名称分组
- 使用 `HAVING` 筛选总访问量超过200的记录
- 最终结果按照总访问量降序排列

## 总结

这三个关键字的用法如下：

- 使用 INNER JOIN 连接多个表获取关联数据
- 使用 DISTINCT 去除重复记录，获取唯一值
- 使用 HAVING 对分组后的数据进行筛选
