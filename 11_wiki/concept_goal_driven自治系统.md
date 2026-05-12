---
title: Goal-Driven 自治系统
type: concept
tags: [Agent, Goal-Driven, 自治系统, 多Agent协作, 状态管理]
created: 2026-05-08
updated: 2026-05-08
source: 十年老技术开发的 AI Agent 探索之路
---

# Goal-Driven 自治系统

> Task-Driven 解决执行问题，Goal-Driven 解决迭代问题。前者让系统能跑，后者让系统能持续向前。

---

## 核心命题

Task-Driven 模式下，人仍然是系统的瓶颈——所有任务都需要人持续供给。Goal-Driven 的跃迁是：**让系统围绕目标自主向前走，而不再等人派活。**

---

## Task-Driven vs Goal-Driven

| 维度 | Task-Driven | Goal-Driven |
|------|-------------|-------------|
| 人的角色 | 项目经理 + 执行监督 | 目标设定者 / 审核者 |
| Agent 的角色 | 执行器 | 自主推进者 |
| 决策中心 | 在人脑子里 | 在目标 + 边界 + 系统状态里 |
| 主要成本 | 人持续编排 | 前期建模和约束设计 |
| 适用场景 | 简单、一次性任务 | 长期、复杂、持续推进任务 |

> **24h 在线，不等于 24h 迭代。**

---

## Goal-Driven 的 5 个前提

只有在这 5 个前提成立时，自主推进才是资产；否则只会把错误放大得更快。

1. **目标必须清晰**——不是模糊愿望，而是可推进、可判断的目标表达
2. **边界必须清晰**——哪些能做，哪些不能做，资源上限是什么
3. **状态必须可见**——当前做到哪一步，卡在哪，为什么卡
4. **过程必须留痕**——否则无法知道为什么成功，也无法知道为什么失败
5. **权限必须可控**——能调用哪些工具，能写到哪里，谁来兜底

> **Goal-Driven 不是更放权，而是更强约束下的有限自治。**

---

## 共享状态：STATE.yaml

多步骤、多角色协作时，主 Agent 会变成新瓶颈（上下文重、通信慢、单点故障）。解法是用共享状态协调：

```yaml
# STATE.yaml — 共享任务面板
goal: "优化搜索模块响应速度，P95 从 800ms 降到 200ms"
owner: "joefu"
constraints:
  - "不修改已有 API 契约"
  - "每日 API 成本不超过 $5"

agents:
  profiler:
    status: "completed"
    summary: "瓶颈在 DB 全表扫描和缺少缓存层"
  backend_dev:
    status: "in_progress"
    current_step: "为热点查询添加 Redis 缓存"
  test_runner:
    status: "pending"
    depends_on: ["backend_dev"]
```

**协调原则：**
- 每个 Agent 自己读取状态、写回进度
- 主会话只负责高层目标和验收
- 主会话负责方向，不负责搬运；系统负责推进，不靠人传话

---

## 6 步落地路径

先让一次执行可复盘，再让它可重复，再让它可规模化，最后让它可有限自主。

| 步骤 | 做什么 | 核心产出 |
|------|--------|---------|
| 1 | 写清楚 spec | 要做什么、不做什么、怎么算完成 |
| 2 | 执行过程留痕 | Prompt/状态/输出/错误全记录 |
| 3 | 补 observability 和 eval | 知道为什么成功、为什么失败 |
| 4 | 高频动作沉淀为 Skill | 模板 + 规则 + 代码 |
| 5 | 引入调度和并发 | 调度层 + 轮询 + 失败切换 |
| 6 | 最后才尝试 Goal-Driven | 目标表达 + 治理边界 + 共享状态 |

> Goal-Driven 必须建立在成熟的 Task-Driven 基础上。一个连任务都执行不稳定、没有留痕、没有 Skill 沉淀的系统，不可能真的进入目标驱动。

---

## Related Guides

- [[concept_sdd规格驱动开发]] — Step 1-2 的方法论基础
- [[concept_skill_engineering]] — Step 4 的 Skill 沉淀
- [[concept_agent自学习闭环]] — 系统如何"学会下一次做得更好"

---

## 相关页面

- [[concept_agent开发基础]] — Agent 系统五层地基
- [[concept_multiagent综述]] — 多 Agent 协作视角
- [[concept_ai系统设计原则]] — 何时需要多 Agent（状态空间隔离）
- [[concept_harness_engineering]] — 治理边界设计
- [[guide_ainative研发方法论]] — 文档驱动的执行方法
- [[AI/Articles/十年老技术开发的 AI Agent 探索之路]] — 原始素材
