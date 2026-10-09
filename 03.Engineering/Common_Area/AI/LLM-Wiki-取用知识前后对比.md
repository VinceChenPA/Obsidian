---
tags:
  - type/note
  - LLM-Wiki
  - knowledge-management
created: 2026-09-13
updated: 2026-10-09
status: done
source: 会话问答归档（Query）
---

# LLM Wiki 前后：取用知识的区别

> **范围**：本页只回答"引入前 vs 引入后的**取用体验**差在哪"，并用 TAVR 实例走一遍；RAG 与 wiki 的机制对比、人类/LLM 分工、三层架构、Ingest/Query/Lint 三种工作流、最佳实践与 Obsidian 技巧以 [[03.Engineering/Common_Area/AI/LLM-Wiki-中文实践指南|LLM Wiki 中文实践指南]] 为准，此处不重复。

## 对比总览

RAG 与 wiki 的机制差异（RAG 每次提问现检索、知识不累积 vs wiki 消化一次持续保鲜）已在 [[03.Engineering/Common_Area/AI/LLM-Wiki-中文实践指南|LLM Wiki 中文实践指南]] §核心思想中给出，此处不重述。本页只补充该页未覆盖的**取用侧**差别：

- **取用路径**：引入前每次提问都要全库找片段、现场拼合；引入后先读索引（`Home.md` → `MOC.md`）直达已综合好的页面。
- **答案形态**：引入前是碎片拼接，依赖当次检索命中；引入后页面已含交叉引用、矛盾标记与更新后的综合。
- **结果稳定性**（本页独有的维度）：传统方式同一问题在不同时间因检索命中不同可能拼出不同结果；LLM Wiki 读的是同一份被维护的页面，结果稳定。
- **人看的东西**：引入前面对原始文件堆；引入后有 MOC 索引 + 关系图谱，知识全貌可见。

## 本库实例：问"TAVR 和外科手术的区别？"

**引入前**
1. agent 翻找 `05.Health/心脏瓣膜病/` 原始笔记；
2. 现场拼凑回答；
3. 答案用完即弃，下次再问重新拼。

**引入后**
1. 沿 `Home.md → 05.Health/MOC.md` 直达 4 篇已互链笔记，读整理好的内容作答；
2. 答案有价值时询问是否归档（Query），写回 wiki；
3. 矛盾与缺口由 Lint 周期性暴露。

## 一句话总结

以前知识库是"档案室"（存材料，取用靠检索）；现在是"编译车间"（存加工品，取用读成品，且每问一次可能多一件成品）。

## 为什么有效

结论只留一句：**维护成本趋近于零，wiki 就能活下来。** 论证（知识库的痛点是"维护性簿记"而非阅读与思考；人类因维护成本增长快于价值积累而放弃 wiki，LLM 不会厌烦、不会漏改）已在 [[03.Engineering/Common_Area/AI/LLM-Wiki-中文实践指南|LLM Wiki 中文实践指南]] §为什么有效中完整给出，此处不重复。

## 相关

- [[03.Engineering/Common_Area/AI/LLM-Wiki-中文实践指南|LLM Wiki 中文实践指南]]
- [[03.Engineering/Common_Area/AI/AI-工作流触发指南|AI 工作流触发指南]]
