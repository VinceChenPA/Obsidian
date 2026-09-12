---
created: 2026-08-29
tags:
  - engineering/dbt
  - engineering/bigquery
---
# dbt BigQuery 外部表

dbt-bigquery 支持创建 BigQuery 外部表，有三种方式：

## 1. 内置 `materialized='external'`（推荐，dbt-bigquery 1.8+）

在 model 或 dbt_project.yml 中配置：

```yaml
models:
  - name: my_external_model
    config:
      materialized: external
      location: 'gs://my-bucket/path/*.parquet'
      format: parquet        # parquet / csv / orc / avro / json 等
      partition_by:          # 可选
        - field: date
          data_type: date
      auto_refresh: true     # 可选，分区自动刷新
      auto_refresh_interval: 60
```

要点：

- 数据存在 Cloud Storage 中，BigQuery 查询时实时读取
- 外部表只读，不能用 DML 修改
- 需要同时具备 BigQuery 表和 GCS bucket 的读取权限

## 2. dbt-external-tables 包（旧方式）

在 `sources` 中定义 `external` 属性，用 `stage_external_sources` 宏建表/刷新分区：

```yml
sources:
  - name: snowplow
    tables:
      - name: event
        external:
          location: gs://my-bucket/path/
          partitions:
            - name: event_date
              data_type: date
        columns:
          - name: app_id
            data_type: varchar(255)
```

dbt-bigquery 1.8 之前主要靠这个包；Snowflake、Redshift 等平台目前仍使用此方式。

## 3. 直接写 DDL

在 model 里直接写 BigQuery 原生 DDL（2020 年 10 月 GA）：

```sql
CREATE OR REPLACE EXTERNAL TABLE `project.dataset.my_table`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://my-bucket/path/*.parquet']
);
```

## 在 dbt macro 中使用外部表 DDL

可以。macro 是 SQL 模板，外部表 DDL 可在宏中执行：

```sql
-- macros/create_my_external.sql
{% macro create_my_external_table() %}
  {% set ddl %}
    CREATE OR REPLACE EXTERNAL TABLE `project.dataset.my_table`
    OPTIONS (
      format = 'PARQUET',
      uris = ['gs://my-bucket/path/*.parquet']
    )
  {% endset %}
  {% do run_query(ddl) %}
{% endmacro %}
```

调用方式：

- **hooks**：`+on-run-start: "{{ create_my_external_table() }}"`（最常用）
- **自定义 materialization**：在 `{{ materialization }}` 块中拼 DDL 并 `run_query`
- **动态生成**：宏支持循环/参数化，按表名、格式等动态拼出多个外部表 DDL，比 yaml 静态配置灵活

注意：外部表是独立于 dbt 关系的 DDL，宏中不能对其使用 `ref()`。若只是普通建表，优先用内置 `materialized='external'`。

## 需要给出 column 列表吗

视方式而定：

- **原生 DDL / 内置 external**：column 列表可选。
  - Parquet / Avro / ORC 自带 schema，BigQuery 自动读取，无需（也不可自定义）列定义
  - CSV / JSON 无 schema：省略则自动检测（类型推断可能不准），也可显式声明精确控制：

    ```sql
    CREATE OR REPLACE EXTERNAL TABLE `project.dataset.my_table`
    (
      app_id STRING,
      platform STRING,
      event_date DATE
    )
    OPTIONS (
      format = 'CSV',
      uris = ['gs://my-bucket/path/*.csv']
    );
    ```

- **dbt-external-tables 包**：一般要求列出全部 columns（CSV 列顺序必须与文件一致；Parquet 等按列名匹配）。

## 参考资料

- [BigQuery 外部表介绍](https://docs.cloud.google.com/bigquery/docs/external-tables)
- [dbt-external-tables 包](https://github.com/dbt-labs/dbt-external-tables)
- [dbt external 资源属性](https://docs.getdbt.com/reference/resource-properties/external)
