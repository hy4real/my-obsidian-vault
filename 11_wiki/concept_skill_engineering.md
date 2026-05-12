---
title: Skill 工程范式 — 从 Prompt 玩具到可执行协议
type: concept
tags: [skill-engineering, agent, codex, workflow, prompt, production]
created: 2026-05-03
updated: 2026-05-03
source: 深度解析：Codex Pet Skill（lencx, 2026-05-02）
---

# Skill 工程范式 — 从 Prompt 玩具到可执行协议

> 真正的 Skill 不是"让模型换个语气说话"，而是把一个领域里隐性的经验、边界、工具链、失败处理、验收标准和执行流程，压缩成 Agent 可以稳定调用的可执行协议。

---

## 核心定义

**Skill Engineering**：将领域知识编译为 Agent 可执行的生产协议，包含触发条件、任务编排、确定性脚本、QA 标准、修复策略和产物规范。区别于简单的 prompt wrapper，Skill 工程把不可控的模型能力关进可控的工程边界里。

---

## Skill 与 Prompt Wrapper 的本质区别

| 维度 | Prompt Wrapper | Skill 工程 |
|------|---------------|-----------|
| 定义方式 | 几段角色扮演 instructions | 产物协议 + 任务图 + 验收标准 |
| 执行模型 | 单轮"像 XX 一样思考" | 多阶段流水线，含失败修复 |
| 状态管理 | 藏在上下文里 | 外部化成 manifest 文件 |
| 质量保证 | 无 / "看起来不错" | 确定性 QA + 语义检查 |
| 失败处理 | 重新 prompt | 最小范围局部修复 |
| 产物定义 | 模型输出即最终产物 | 模型输出只是生产材料 |

---

## Codex Skill 机制

OpenAI 的 Skill 定义：一个 Skill 是包含 `SKILL.md`、可选 `scripts/`、`references/`、`assets/`、`agents/openai.yaml` 的目录。

**渐进式披露（Progressive Disclosure）**：Codex 一开始只把 skill 的 name、description、路径放入上下文，只有当决定使用时才读取完整 `SKILL.md`——避免技能库把上下文撑爆。

**三层角色分工**：
- **SKILL.md** — 操作系统级 workflow contract（谁能生成、谁能写 manifest、何时停止、哪些行为禁止）
- **scripts/** — 确定性编译器（抽帧、拼 atlas、透明验证、hash、package）
- **references/** — 协议文档（动画状态定义、资产格式、验收标准）

这三层的分离是关键：模型处理语义，脚本处理确定性，references 定义契约。

---

## hatch-pet 的 Skill 工程范式

OpenAI 的 `hatch-pet` skill（生成 Codex 电子宠物动画资产）表面上是玩具，实质上展示了成熟的 Skill 工程：

**架构边界**：
- **非确定性部分** → `$imagegen`（系统级图像生成 skill）负责
- **确定性部分** → hatch-pet scripts 负责（抽帧、拼 atlas、验证、打包）

**反作弊边界**（禁止模型"假装完成生产"）：
- 不能手改 `imagegen-jobs.json` 标记完成
- 不能复制本地文件冒充生成结果
- 不能写 helper script 填充输出
- 必须用 `record_imagegen_result.py` 记录真实生成输出

**核心链路**：
```
intent → request → job manifest → generated candidates
→ recorded provenance → compiled artifact → QA result
→ targeted repair → packaged asset
```

这条链路的普适性：把用户模糊的创意请求，转换成有输入规范、任务图、来源证明、确定性编译、QA、修复和最终打包的资产流水线。

---

## Skill 的设计原则

### 1. description 是路由入口
不是营销文案，是 route spec。必须精准说明何时触发、产出什么、约束什么。

### 2. 约束先行，执行后置
先定义产物是什么，再定义生成过程如何受控。美术风格不是审美偏好，而是工程约束（192x208 小尺寸下高细节会糊成噪声）。

### 3. manifest 是控制中心
把任务依赖、输入输出、来源、派生规则、完成状态写成可检查结构。详见 [[concept_manifest_driven_workflow]]。

### 4. 并行分发，提交集中
子代理可以并行生成，但 manifest 写入、资产打包、最终提交必须由父代理独占。详见 [[concept_multiagent综述]]。

### 5. 局部修复优于整体重跑
失败后定位最小失败范围，归档旧输出，追加 repair note，重开对应 job。详见 [[concept_asset_pipeline_and_qa]]。

---

## Skill vs 传统 Workflow 工具

| 维度 | n8n / Zapier | Skill 工程 |
|------|-------------|-----------|
| 优势 | 确定性链路、节点清晰 | 动态决策、局部修复、多代理协作 |
| 劣势 | 上下文理解弱、维护巨型流程图 | 需要精心设计 manifest 和约束 |
| 适用 | 简单触发器 + 固定流程 | 复杂任务 + 需要理解上下文 |

传统 workflow 擅长确定性链路；一旦任务需要动态决策、局部修复、多代理协作，就会变成越来越难维护的巨型流程图。Skill 的优势：保持模块化，让 Agent 根据当前任务状态选择工具、展开步骤、调用脚本、分发子任务、回收结果，并在失败后按协议修复。

---

## Codex /goal 命令

Codex v0.128.0 引入目标驱动执行：`/goal <objective>` 允许设置明确目标，agent 完成一轮后自动注入提示引导选择下一个动作。goal 的需求映射到可验证证据（文件变更、测试结果、PR 状态），只能通过更新 goal 标记完成。内置 Ralph loop++——围绕目标持续执行、检查证据、推进任务，直到完成或 token 预算耗尽。

---

## Agent 工程启发

1. **模型是 creative worker，不是 trusted committer** — 模型输出只是生产材料，必须经过记录、编译、QA、打包才成为最终资产
2. **关键状态不能藏在上下文里** — 必须外部化成 manifest（对话上下文会压缩、遗忘，文件不会）
3. **prompt 可以提升单次质量，协议才能提升系统稳定性**

---

## Related Concepts

- [[concept_harness_engineering]] — Skill 工程是 Harness 思维在具体领域的应用：模型只负责选择下一步，状态、依赖、边界由外部结构保存
- [[concept_manifest_driven_workflow]] — Skill 的核心控制结构：manifest 驱动的任务编排
- [[concept_asset_pipeline_and_qa]] — Skill 的产物管理：从生成到打包的完整流水线
- [[concept_multiagent综述]] — Skill 中子代理并行的设计模式
- [[concept_ai系统设计原则]] — 四层信息架构如何支撑 Skill 设计
- [[guide_ainative研发方法论]] — Skill 工程是 AI Native 研发方法的具体体现
- [[project_ai工程框架]] — Skill 工程在整体 AI 工程框架中的位置

---

## References

- [hatch-pet skill 源码](https://github.com/openai/skills/tree/main/skills/.curated/hatch-pet)
- [OpenAI Skills](https://github.com/openai/skills)
- [codex-rs/goals](https://github.com/openai/codex/blob/rust-v0.128.0/codex-rs/core/templates/goals)
