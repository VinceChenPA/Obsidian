---
tags:
  - engineering/sql
---
# Common Table Expression (CTE)

CTE (Common Table Expression) 是一种临时结果集，使用 `WITH` 子句定义，可在后续查询中引用。

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

CTE 提高了复杂查询的可读性和可维护性，常用于递归查询和分步数据处理。
