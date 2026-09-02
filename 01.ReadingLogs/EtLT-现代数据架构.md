---
created: 2023-09-14
updated: 2026-09-03
tags:
  - type/note
  - reading/books
  - data-engineering
source: https://blog.devgenius.io/elt-is-dead-and-etlt-will-be-the-end-of-modern-data-processing-architecture-154b87c1cce0
---

# EtLT：从 ELT 到 Extract-t-Load-Transform

> 来源：Dev Genius（Apache SeaTunnel 团队观点，2023-07）。Omnivore 导入整理。

## 核心论点

传统 ELT 把「加载后的大表转换」交给数据仓库，但面对**实时数据与 AI 应用**需求（CDC、流处理、异构 SaaS 源），ETL/ELT 都不够。EtLT 把流程拆为四段：

- **E(xtract)** — 抽取：传统库/文件/SaaS API/Serverless 源；支持 binlog 实时 CDC 与 Kafka 流、批量分片读取。
- **t(ransform)** — 轻量归一化（小写 t，新增阶段）：进入仓库**之前**把异构、非结构化源快速转成可加载的结构化数据（拆分、过滤、改字段格式），批/流两用。
- **L(oad)** — 加载：不止灌数，还适配目标端结构（Schema Evolution 处理、Bulk Load、Reverse ETL、JDBC）。
- **(T)ransform** — 业务转换（大写 T）：在数仓/联邦内由 SQL 完成业务逻辑，批/流皆可。

## 职责划分

| 阶段 | 负责人 | 关注点 |
|------|--------|--------|
| EtL | 数据工程师 | 源数据→结构化，时效性与转换准确性 |
| T | 分析师 / SQL 开发者 / AI 工程师 | 业务规则→SQL，数据质量与业务口径 |

## 我的关联

- 与 dbt 定位互补：dbt 属于「T 阶段」（仓库内 SQL 转换），EtL 的「小 t」由数据管道工具（Data Fusion / SeaTunnel 等）承担。见 [[03.Engineering/dbt/dbt introduction|dbt 介绍]]。
- BigQuery 外部表 / CDC 场景（[[03.Engineering/dbt/dbt BigQuery 外部表|dbt BigQuery 外部表]]）正是「小 t + L」的实践面。

## 待验证

- SeaTunnel 为自家产品营销文，「ELT is dead」表述夸张；EtLT 本质是给 ELT 加了「入库前规范化」环节，此概念与主流 Lakehouse 实践（Medallion 架构）的关系值得对照。
