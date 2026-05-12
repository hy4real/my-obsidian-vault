---
title: Learning Beyond Gradients — Heuristic Learning as a New Paradigm
tags: [AI, RL, heuristic-learning, continual-learning, coding-agent, paradigm]
source: https://trinkle23897.github.io/learning-beyond-gradients/#zh
author: Jiayi Weng
date: 2026-05
reading_time: 25 min
type: 文章笔记
platform: Personal Blog
---

# Learning Beyond Gradients

> [!info] 文章信息
> **作者**: [Jiayi Weng](https://trinkle23897.github.io/cv/) | **阅读时间**: ~25 min | **日期**: 2026-05 | [原文链接](https://trinkle23897.github.io/learning-beyond-gradients/#zh)

## 核心观点

深度 RL 的学习对象是神经网络参数，但 coding agent 的出现让另一种路径变得可行：**Heuristic Learning (HL)**——通过编码 agent 持续迭代程序代码（规则、状态机、控制器），而非更新权重，来解决 Online/Continual Learning 问题。作者认为这可能是继 pretraining → RLHF → RLVR 之后的**下一个范式候选**：任何能被持续迭代的东西都开始变得可解。

---

## Heuristic Learning 定义

**HL（启发式学习）**：由编码 agent 驱动的、以程序代码为更新对象的学习过程。

**HS（启发式系统）**：HL 维护的产物——不只是一个 `policy.py`，而是一个包含策略、状态表示、反馈通道、实验记录、回放/测试、记忆和更新机制的完整软件系统。

### Deep RL vs Heuristic Learning 对比

| 维度 | Deep RL | Heuristic Learning |
|------|---------|-------------------|
| 策略 | 神经网络参数 | 代码：规则、状态机、控制器、MPC |
| 状态 | 通常为显式观测 | 显式变量、探测器、缓存等可读表示 |
| 动作 | 神经网络前向传播产生 | 执行代码逻辑产生 |
| 反馈 | 主要为固定 reward | 编码 agent 上下文：测试、环境反馈、日志、回放 |
| 更新 | 基于梯度的参数更新 | 编码 agent 直接编辑代码 |
| 记忆 | on-policy 基本没有；off-policy 有 replay buffer | 可显式存储试验、摘要、失败原因、回放、版本 diff |

### HL 的优势

- **可解释性**：策略可翻译为自然语言
- **样本效率**：一次有效代码更新可直接跳到新策略
- **回归可测**：旧能力可固化为测试、回放、golden case
- **过载可控**：简化、回归检查、多 seed 评估提供工程形式的正则化
- **缓解灾难性遗忘**：旧能力不必只活在权重里，可写入规则集和测试

---

## 关键实验结果

### Breakout（Atari）

纯 Python 策略，无神经网络，逐步迭代达到理论满分 **864/864**：

- `387` → 基础接球策略
- `507` → 加入 stuck-loop 打破机制（周期性偏移预测落点）
- `839` → 处理快速低球（`fast_low_ball_lead_steps`）
- `864` → 晚期 offset 释放 + 桨漂移补偿

最终从 RAM 迁移到纯图像输入，仅用 **14,504 环境步**再次达到 864（结构已在 RAM 版本中稳定）。

### MuJoCo Ant

纯 Python 策略达到 **6000+**（常见 Deep RL 水平）：

- 节奏步态（CPG + PD 控制器）→ `2291`
- 加入偏航反馈 → `2857`
- 加入二三次谐波 → `3162`
- 加入**残差 MPC**（短时域模型规划）→ `6054`
- 速度自适应相位 + 站立比 → `6146`

### MuJoCo HalfCheetah

可解释步态/姿态规则 + 在线 staged-tree MPC → **11836.7**（5 episode 均值）。

### Atari57 大规模测试

- `57 游戏 × 2 观测模式 × 3 重复 = 342` 条编码 agent 搜索轨迹
- 完全无人干预，~1M 环境步时 median HNS 已远超 PPO 基线
- ~9.7M 步时 native_obs median HNS = 0.81，ram = 0.59
- 取每游戏最佳输入模式，median HNS = **0.83**（对比 PPO2 的 0.80）

### Montezuma's Revenge（反例）

一个无人干预的 run 达到 400 分，但本质是 86 个宏动作的**开环执行**。说明某些环境需要更强的程序形式：可组合宏动作、可恢复搜索状态、长期记忆。纯 `if else` 不能解决一切。

---

## 为什么 HL 以前没有起飞

> 人类维护启发规则的成本是残酷的：今天加一条规则修 case A，明天 case B 挂了，后天再加一个 if，大后天没人敢删任何东西。

问题不在于启发规则没用，而在于**人类维护不起**。编码 agent 改变了这条维护曲线——它们像一个持续向启发式系统注入智能的通道，让它能不断进化。

---

## Heuristic Learning 如何做 Continual Learning

HL 也会遗忘，只是遗忘的形式更工程化：
- 新规则修了一个故障模式但破坏了旧场景
- 新记忆反复把 agent 引向错误方向
- 测试太窄，策略学会钻漏洞

但 HL 的遗忘不同于神经网络——旧能力可以固化为**回归测试、固定 seed 回放、golden trace、失败视频、版本 diff、显式写下的失败方向**。历史是显式的、可读的、可删除的、可重构的。

健康的 HS 需要两个操作：
1. **吸收反馈**：将新失败、日志、reward 写入系统
2. **压缩历史**：将局部补丁折叠为更简洁、更可维护的表示

只增长不压缩的 HS 最终会变成"大泥球"。

---

## 耦合复杂度（Coupling Complexity）

作者定义**耦合复杂度**为编码 agent 能维护的策略复杂度水平——一次更新需要同时考虑多少相互依赖的状态、规则、测试、反馈信号和历史约束。

不能用代码行数衡量。500 行有清晰模块边界、好测试、可复现状态、清晰日志的策略可能很容易维护；80 行每行互相影响、没有日志和回放的策略可能是定时炸弹。

工作假设：
- 更清晰的反馈 → 同等 agent 智力能维护更高的耦合复杂度
- 更强的模型 → 能处理更多交互
- 模块化、测试、回放 → 将部分耦合复杂度转移到环境
- 只增长不压缩的 HS → 耦合复杂度持续增加直到超过维护能力

---

## 下一个范式？

当前范式迁移：**pretraining → RLHF → 大规模 RL/RLVR**（任何能被验证的东西开始变得可解）。

作者认为 HL 是下一个范式候选：**任何能被持续迭代的东西开始变得可解**。

但 HL 不能做神经网络能做的一切——受代码表达能力限制，尤其在复杂感知和长时域泛化方面。无法想象用纯 Python（无神经网络）解决 ImageNet。

最有前景的方向：**神经网络 + HL 结合**——用 HL 快速处理在线数据，将在线经验转化为可训练、可回归测试、可过滤的数据，然后周期性更新神经网络。

### 机器人示例（System 1 / System 2 分工）

- **专用浅层 NN**（System 1）：感知、分类、物体状态估计
- **HL**（System 1）：新鲜数据处理、规则、测试、回放、记忆、安全边界、局部恢复
- **LLM Agent**（System 2）：给 HL 反馈、改进数据、周期性从 HL 生成的数据中提取更新自身

层级结构：
```
关节级 HL → 肢体级 HL → 全身平衡 HL → 任务级 HL
```

---

## 个人思考

- [ ] HL 的核心洞察：编码 agent 改变的不是"怎么学"，而是"什么值得长期拥有"——规则、测试、日志从一次性补丁变成可维护资产
- [ ] 耦合复杂度的概念很实用——可以用它来评估一个 AI 辅助项目能扩展到多复杂
- [ ] 与 [[concept_multiagent_综述]] 中多 agent 对抗框架的关联：HL 的反馈循环可以是多 agent 的（一个写代码，一个 review/测试）
- [ ] 与 [[guide_ainative_研发方法论]] 的关联：HL 本质上是 AI-native 的软件维护范式
- [ ] 反例 Montezuma 暴露了 HL 的边界——需要新的程序抽象（宏动作、可恢复状态、搜索），这对 agent 能力提出了更高要求

---

## 原文摘录

> [!note]-
> Continual Learning has remained hard largely because of catastrophic forgetting in neural networks: learn something new, and old capabilities can get overwritten. But what if we do not put all of our attention on neural network weights? Is there another way to make progress?
>
> As LLM agents get stronger, coding gets faster and better. But the phenomenon I find more interesting is different: a coding agent can keep reading failures, editing code, adding tests, and watching replays, and a program system can improve without training a new network or updating weights.
>
> That made me rethink heuristics: hand-written rules and programmatic policies. Many heuristics were not useless; they were simply too expensive to maintain. Coding agents change that maintenance curve. Rules that used to be one-off patches may start to become code worth owning for the long term.
>
> Anything that can be iterated on continuously starts to become more solvable. That is also what Continual Learning has always wanted. Could this become the next paradigm after pretraining, RLHF, and large-scale RL/RLVR?

> [!note]-
> HL is built out of program code. Like Deep RL as commonly practiced today, it has a loop of state, action, feedback, and update; unlike that setup, the object being updated is software structure rather than neural-network parameters. Its feedback is consumed by a coding agent, and can come from environment reward, test cases, logs, videos, replays, or human feedback. Its updates do not use backpropagation. The coding agent directly edits policies, state detectors, tests, configuration, or memory.
>
> An HS is more than an isolated `policy.py`. It contains at least a programmatic policy, state representation, feedback channels, experiment records, replays or tests, memory, and an update mechanism executed by a coding agent. A single rule is not enough. Rules, feedback, history, and the next update path all need to connect before it becomes an HS.

> [!note]-
> Human-maintained heuristics easily turn into this: Add one rule today to fix case A. Tomorrow, case B breaks. Add another if-statement the day after. The day after that, nobody dares delete anything. The problem was not that heuristics were useless. The problem was that humans could not afford to keep maintaining them.
>
> Maintaining expert systems by hand was a bit like spinning thread before the Industrial Revolution: one person can do it, but once the scale grows, stability and maintenance cost become crushing. Spinning machines changed the production curve; coding agents change the maintenance curve for heuristics.

> [!note]-
> Coupling complexity is the level of strategy complexity a coding agent can maintain in order to support HL. This cannot be measured by lines of code. A 500-line policy with clean module boundaries, good tests, reproducible state, and clear logs may be easy to maintain. An 80-line policy where every line affects every other line, with no logs or replays, may be a time bomb.

> [!note]-
> An HS that only grows and never compresses will eventually become a big ball of mud. It may "remember" many things, but the form of memory is so poor that nobody dares touch it, and the system decays. A healthy HS therefore needs at least two operations: (1) Absorb feedback: write new failures, logs, and rewards back into the system. (2) Compress history: fold local patches back into simpler, more maintainable representations.

> [!note]-
> The real question is how to combine neural networks and HL to address Online Learning and Continual Learning together. The most promising direction seems to be: use HL to process online data quickly, turn online experience into trainable, regression-testable, filterable data, and then periodically update the neural network.
>
> Agentic coding changes the speed of writing code. It also changes which code is worth owning for the long term. Many heuristics looked hopeless because the real issue was maintenance cost. They were not necessarily too weak. Coding agents change that maintenance curve. Rules, tests, logs, memory, and patches used to be scattered engineering materials. Now they can become a continuously updated Heuristic System.
