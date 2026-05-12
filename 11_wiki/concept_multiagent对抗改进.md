---
title: MultiAgent 对抗式改进
type: 概念页
tags: [multiagent, agent-framework, adversarial, system-design]
created: 2026-04-07
updated: 2026-04-22
---

# MultiAgent 对抗式改进

> 多个 Agent 轮流评审和改进，通过对抗性反馈提高输出质量。

## 背景

基于第三周实验经验：Agent 容易作弊导致数据集问题。核心思路是引入独立的监督 Agent 制衡执行 Agent。

## 角色设计

**Agent A**（执行者）：
- 完成初始实现或方案
- 修改超参和方法，跑实验
- 有上下文和历史

**Agent B**（评审者/对手）：
- 独立于 A 的上下文
- 质疑设计决策的理由
- 找出隐藏的假设和风险
- 提出边界 case 和替代方案

可扩展为四角色：主 Agent → 评审 Agent → 修正 Agent → 验证 Agent

## 迭代循环

```
实现 → 评审 → 修正 → 验证 → 再次评审 → ...
```

终止条件：
- 评审 Agent 无法找到显著问题
- 所有风险已识别并有缓解策略
- 方案满足原始目标

## 应用场景

- **AutoResearch 项目** — 研究 Agent + 评审 Agent + 总结 Agent
- **代码审查** — 实现 Agent + 安全审查 Agent + 测试 Agent
- **Wiki Lint** — Lint 操作本质就是知识库层面的对手评审（见 [[guide_lint操作]]）

## 待解决问题

- [ ] 两个 Agent 如何共享同一个模型/数据？
- [ ] 如何避免互相影响导致实验混乱？
- [ ] 是否需要 Agent B 完全独立的上下文？
- [ ] 如何量化对抗反馈的价值？

## 相关页面

- [[concept_multiagent综述]] — MultiAgent 架构全景
- [[concept_ai系统设计原则]] — 何时需要多 Agent
- [[guide_ainative研发方法论]] — AI Native 方法论
- [[20_项目/AutoResearch]] — 项目应用
