---
title: 第一次 Ingest 工作流示例
type: guide
tags: [ingest, workflow, example, wiki]
created: 2026-04-26
updated: 2026-04-26
---

# 第一次 Ingest 工作流示例

> 本页是 [[guide_ingest操作]] 的配套示例，演示从一篇原始文章到 Wiki 页面的完整 Ingest 过程。

---

## 示例材料

**来源**：`00_raw/articles/karpathy_wiki知识管理.md`
**主题**：Karpathy 关于 LLM-Wiki 知识管理模式的核心观点

---

## Step 1：读取原始材料

打开 `00_raw/articles/karpathy_wiki知识管理.md`，通读全文，识别核心概念。

---

## Step 2：提取概念

从文章中提炼出 3-5 个核心概念：

1. **LLM-Wiki 模式** — LLM 持续维护 Markdown 知识库，而非每次从零检索
2. **知识复利** — 每新增节点与现有节点形成连接，价值指数增长
3. **三大操作** — Ingest / Query / Lint 形成飞轮

→ 这些概念被归入 [[concept_wiki系统]]，无需新建独立页面（已覆盖）。

---

## Step 3：检查现有页面，决定新建还是更新

| 概念 | 操作 | 目标页面 |
|------|------|---------|
| LLM-Wiki 整体理念 | 更新 | [[concept_wiki系统]] |
| 知识积累模式 | 更新 | [[concept_wiki系统]] |
| Ingest/Query/Lint 操作 | 已有独立页面，仅补充链接 | [[guide_ingest操作]] |

---

## Step 4：补充交叉链接

在 [[concept_wiki系统]] 的"相关页面"段落追加来源引用，并在相关 guide 页面的 Backlinks 段落登记。

---

## Step 5：更新 _index

在 [[_index]] 的 `concept_*` 分区确认 [[concept_wiki系统]] 已列出，描述准确。

---

## 耗时参考

| 阶段 | 预计时间 |
|------|---------|
| 读取原文 | 2 min |
| 提取概念 + 决策 | 3 min |
| 更新/创建页面 | 5 min |
| 补充链接 | 2 min |
| **合计** | **~12 min** |

---

## 相关页面

- [[guide_ingest操作]] — Ingest 完整操作规范
- [[concept_wiki系统]] — 本次 Ingest 的主要目标页面
- [[guide_lint操作]] — Ingest 后可运行 Lint 检查结构完整性
