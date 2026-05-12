---
title: MultiAgent 系统综述
type: 综述页
tags: [MultiAgent, 多智能体, Agent, 系统设计, AI工程]
created: 2026-04-12
updated: 2026-04-12
---

# MultiAgent 系统综述

> **核心思想**：多个专化 Agent 协作，完成单个 Agent 无法高质量完成的复杂任务。

---

## 核心定义

MultiAgent 系统是由多个具有不同角色和能力的 LLM Agent 组成的协作架构。每个 Agent 专注于自己的子任务，通过结构化的信息传递和协调机制完成整体目标。

---

## 为什么需要 MultiAgent？

单个 LLM 的局限：
- **注意力分散**：复杂任务难以同时关注所有维度
- **角色冲突**：写作者和评审者不能是同一个实体
- **并行瓶颈**：单线程执行，无法并行化

MultiAgent 的优势：
- **专化分工**：每个 Agent 专精一个角色
- **相互制衡**：评审 Agent 独立验证执行 Agent 的输出
- **并行执行**：多个 Agent 同时处理不同子任务

---

## 主要架构模式

### 1. 对手评审模式（Opposition Review）
见 [[concept_multiagent对抗改进]]。

设置两个 Agent：
- **执行 Agent**：完成主要任务
- **评审 Agent**：专门找问题、提出反对意见

**效果**：显著减少幻觉和错误，提高输出质量。

### 2. 流水线模式（Pipeline）
多个 Agent 串行处理，每个 Agent 的输出是下一个的输入：
```
研究 Agent → 整理 Agent → 写作 Agent → 校验 Agent
```

### 3. 并行分工模式
多个 Agent 同时处理任务的不同部分，最后汇总：
```
Agent A: 模块1 ─┐
Agent B: 模块2 ─┤─→ 合并 Agent → 最终输出
Agent C: 模块3 ─┘
```

---

## 关键设计原则

1. **角色隔离**：每个 Agent 有明确的角色和边界，不越界
2. **信息结构化**：Agent 间传递的信息必须有明确格式
3. **评审独立性**：评审 Agent 不依赖执行 Agent 的自我判断
4. **失败处理**：每个 Agent 节点都需要处理上游失败的情况

---

## 实际应用案例


- **AutoResearch 项目**：研究 Agent + 评审 Agent + 总结 Agent
- **代码评审**：实现 Agent + 安全审查 Agent + 测试 Agent
- **内容生成**：写作 Agent + 事实核查 Agent + 编辑 Agent

---

## 与 LLM-Wiki 系统的关系

[[concept_wiki系统]] 本身就是一个 MultiAgent 模式的应用：
- **执行 Agent**（Claude Code）：做 Ingest、Query、Lint
- **用户**：充当评审者，提供方向和反馈
- **Schema**：定义 Agent 的工作边界

---

## 待深化主题

- [ ] Agent 间通信协议设计
- [ ] 如何评估 MultiAgent 系统的整体质量
- [ ] 成本控制：Agent 数量 vs 质量的权衡
- [ ] 长时间运行的 MultiAgent 系统的状态管理

---

## 相关页面

- [[concept_multiagent对抗改进]] — 对手评审模式详解
- [[concept_ai系统设计原则]] — 何时触发多 Agent 的三条判断标准
- [[concept_ai系统设计原则]] — Agent 系统的四层信息架构
- [[concept_wiki系统]] — Wiki 系统本身就是 MultiAgent 模式的应用
- [[concept_multiagent_wiki洞察]] — 两个系统的深层关联
- [[guide_ainative研发方法论]] — AI Native 方法论
- [[concept_token_efficiency]] — Agent 时代 token 消耗是 Chatbot 的 10-100 倍，效率是 Multi-Agent 大规模商业化的前提
- [[concept_skill_engineering]] — Skill 中子代理并行生成 + 父代理独占写权限的设计模式
- [[concept_manifest_driven_workflow]] — 子代理写权限隔离的具体实现：并行分发，truth commit 集中
