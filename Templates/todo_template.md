---
tags:
  - type/todo
created: <% tp.date.now("YYYY-MM-DD") %>
updated: <% tp.date.now("YYYY-MM-DD") %>
status: active
source:
---

# <%= tp.file.title %>

> 条目格式：`- [ ] 标题 ｜ P2 ｜ 截止 YYYY-MM-DD ｜ [[关联笔记]]`（截止日期与关联可省略，优先级缺省 P2）。
> 状态由所在区块表达：`## 进行中` = doing，`## 待办` = todo，`## 归档` = done / dropped。
> 完成时把整行挪到 `## 归档`、勾成 `- [x]`，并在行尾补 `done YYYY-MM-DD`；放弃的任务同样进归档并标 `❌`。
> 清单清空后把 frontmatter 的 `status` 改为 `done`；需要写进当天日志时手动嵌入 `![[本文件#待办]]`。

## 进行中

## 待办

- [ ] 示例：写 dbt 增量模型 ｜ P1 ｜ 截止 <% tp.date.now("YYYY-MM-DD", 7) %> ｜ [[03.Engineering/dbt/dbt introduction]] <!-- 示例行，建完笔记后整行删除 -->

## 归档
