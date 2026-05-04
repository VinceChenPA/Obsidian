---
tags:
  - engineering/dbt
---
# dbt 介绍

dbt (data build tool) 是一个开源的数据转换工具，帮助数据团队以工程化最佳实践构建和维护数据管道。

## 什么是 dbt？

dbt 是一个用于数据仓库建模和数据分析的开源工具，专注于数据转换（T in ELT）。它使用 SQL 作为主要语言，结合 Jinja2 模板引擎，让数据分析师和工程师能够在现代数据仓库（如 BigQuery、Snowflake、Redshift）中直接定义数据模型，无需关心执行编排。

## 核心能力

- **SQL 模型**：用 SQL select 语句定义数据转换，dbt 将其编译为表或视图
- **Jinja2 模板**：支持控制结构（if/for）、环境变量、宏（macros）等，把 SQL 变成编程环境
- **依赖管理**：模型间可相互引用，dbt 自动构建依赖图并确定执行顺序
- **测试**：内置数据质量测试（唯一性、非空、外键等），支持自定义测试
- **文档**：自动生成数据文档和血缘关系图
- **版本控制**：与 Git 集成，支持代码审查和回滚

## 优势

- 提高数据质量和可靠性（测试 + 文档）
- 模块化代码，提升复用性和可维护性
- 自动化编译、调度、文档生成
- 团队协作（Git + Code Review）
- 数据血缘可视化，便于排障

## 使用场景

- 数据仓库建模（事实表、维度表）
- 数据质量检查
- 数据报告生成

## 快速入门

1. 安装：`pip install dbt-core`
2. 初始化项目：`dbt init`
3. 编写 SQL 模型文件
4. 运行：`dbt run`
5. 测试：`dbt test`
6. 生成文档：`dbt docs generate`

## 参考链接

- [dbt Tutorial — startdataengineering.com](https://www.startdataengineering.com/post/dbt-data-build-tool-tutorial/)
- [dbt 官方文档](https://docs.getdbt.com/)

---

## Overview (English)

dbt is a powerful open-source tool that transforms the way data teams build and maintain data pipelines.

1. **Organizes Data Transformations with SQL** — centralized codebase for data transformations
2. **Enforces Best Practices** — modularity, version control, testing, documentation
3. **Streamlines Workflows** — automates compilation, scheduling, documentation
4. **Empowers Collaboration** — version control and clear model structure
5. **Provides Comprehensive Documentation** — living record of your data pipeline
6. **Visualizes Data Lineage** — dependency graphs for tracing data flow