---
title: AI 工程核心框架
type: hub
tags: [AI, 工程, 框架, 系统设计]
created: 2026-04-21
updated: 2026-04-21
---

# AI 工程核心框架

从**系统设计原理** → **构建方法** → **工程化工具** 的完整路径。

## 第一层：系统设计原理

[[concept_ai系统设计原则]] — 为什么这样设计系统？

- 可扩展性
- 可维护性
- 成本效益

## 第二层：构建范式

### 单 Agent 系统

基础 Agent 的设计、推理、决策流程

### 多 Agent 系统（核心）

[[concept_multiagent综述]] — 理论基础和系统分类

[[concept_multiagent对抗改进]] — 实践方案（当前重点）

**关键洞察**：
- 对抗式改进：多 Agent 通过相互批评、改进来优化输出
- 适用场景：复杂推理、代码生成、知识综合
- 对比 RAG：对抗式改进 vs. 检索增强 的选择标准

[[concept_agent自学习闭环]] — 单个 Agent 的自我改进机制

## 第三层：工程化工具

[[concept_wiki系统]] — 知识管理的 AI 工程化方案（含 Ingest/Query/Lint）

[[concept_harness_engineering]] — 引导 AI 而非增强 AI：工作流是管道，知识才是护城河

[[concept_知识分层架构]] — 团队工程交付的知识体系：五层存储 × 五种类型 × 三级成熟度

## 第四层：Skill 工程

[[concept_skill_engineering]] — 从 Prompt 玩具到可执行协议：Skill 的工程范式

[[concept_manifest_driven_workflow]] — 任务状态外部化：manifest 驱动的任务编排与恢复

[[concept_asset_pipeline_and_qa]] — 从生成材料到可信产物：资产流水线与双层 QA

## 实践应用

[[project_ai工程框架]] — 知识图谱自动构建的多 Agent 方案（已暂停，可作参考）

---

## 深度学习路径

1. **了解原理**（1-2 周）→ 阅读系统设计原则、MultiAgent 综述
2. **学习方案**（2-3 周）→ 深入对抗式改进方案、Hermes 机制
3. **动手实践**（4+ 周）→ 在项目中应用、构建 Wiki 系统
4. **复盘优化**（持续）→ 定期 Lint、Ingest 新知识


## 相关页面

- [[concept_multiagent综述]] — 核心理论
- [[guide_ainative研发方法论]] — 核心方法论
- [[concept_wiki系统]] — 知识管理系统
- [[project_工程方法论]] — 工程实践
