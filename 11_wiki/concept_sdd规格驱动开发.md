---
title: SDD 规格驱动开发
type: concept
tags: [Agent, SDD, 工程方法论, 架构设计, Vibe-Coding]
created: 2026-05-08
updated: 2026-05-08
source: 十年老技术开发的 AI Agent 探索之路
---

# SDD 规格驱动开发

> SDD（Spec-Driven Development）：把一件事从模糊想法逐步转成可执行单元，并把这个过程完整留下来。

---

## 核心命题

SDD 在 Agent 场景里的价值不只是"先写文档再开发"，而是：

1. **把模糊需求变成明确规格**——让 AI 的每一步都有据可查
2. **完整留痕**——每一步的 prompt、状态、输出、错误全记录
3. **为自举创造前提**——Agent 能"自己修自己"的根本条件

---

## SDD 文档结构

```
feedbacks/
├── sdd/
│   ├── spec.md      # 规格说明：目标、验收标准
│   ├── plan.md      # 技术方案：涉及文件、实现步骤
│   └── tasks.md     # 任务清单：每步描述、状态
├── tasks.json       # 机器可读的任务执行状态
└── debug/
    ├── prompts/     # 每一步的 prompt
    └── agent.log    # 执行日志
```

每个文件的作用：

| 文件 | 解决的问题 |
|------|-----------|
| `spec.md` | 把"分页有问题"变成"切换页码后列表数据未刷新，原因是查询参数未传递 page 参数" |
| `plan.md` | 把问题变成可执行的技术方案 |
| `tasks.md` | 把方案拆成步骤，每步有明确输入、输出、完成标准 |

---

## SDD vs Vibe Coding

| 维度 | Vibe Coding | SDD |
|------|-------------|-----|
| 前期成本 | 极低（几句话） | 较高（写 spec/plan） |
| 后期代价 | 10 倍 debug 时间 | 可预测，有据可循 |
| 上下文管理 | 随代码增长而混乱 | 文档结构化，持久稳定 |
| 适合场景 | 原型/一次性脚本 | 正式项目、Agent 系统 |

> **Vibe Coding 是先易后难。SDD 是先难后易。大道如夷，而民好径。**

---

## 留痕的真正目的

留痕不是为了 debug，而是为了**进化**。没有记录，系统只能不断"重来一次"；有了记录，系统才能回答：

- 它当时看到了什么输入？
- 为什么做出这个判断？
- Prompt 在哪个环节失效了？
- 哪些动作已经足够稳定，可以固化成 Skill？

---

## SDD 的自举效应

当 Agent 系统有了完整的 SDD 流程后，它可以处理关于自身的 bug：

```
[用户反馈] 系统某个页面有 bug
↓ 系统自动触发 SDD 流程
[spec.md] 定义问题和验收标准
[plan.md] AI 分析代码，生成修复方案
[执行] 修改代码 → 验证 → 完成
```

**自举的三个前提：**
1. 清晰的设计文档——AI 知道每个模块该做什么
2. SDD 标准流程——spec → plan → tasks 的固定路径
3. `constitution.md`——架构约束文件，定义代码组织规范、命名规则、模块边界

> 捷径的尽头是弯路，大道的尽头是自由。

---

## Related Guides

- [[guide_ainative研发方法论]] — 提案/设计/任务三件套的团队实践
- [[concept_agent自学习闭环]] — 留痕如何支撑自学习
- [[concept_skill_engineering]] — 从 SDD 中沉淀 Skill 的方法

---

## 相关页面

- [[concept_agent开发基础]] — Agent 系统五层地基
- [[concept_ai系统设计原则]] — 决策层级（何时不用 Agent）
- [[concept_goal_driven自治系统]] — SDD 的下一阶段演进
- [[concept_harness_engineering]] — 工程约束与自举
- [[project_工程方法论]] — 个人工程方法论
- [[AI/Articles/十年老技术开发的 AI Agent 探索之路]] — 原始素材
