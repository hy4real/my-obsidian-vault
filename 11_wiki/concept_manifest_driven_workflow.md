---
title: Manifest 驱动工作流 — 任务状态外部化的工程模式
type: concept
tags: [manifest, workflow, agent, state-management, orchestration, provenance]
created: 2026-05-03
updated: 2026-05-03
source: 深度解析：Codex Pet Skill（lencx, 2026-05-02）
---

# Manifest 驱动工作流 — 任务状态外部化的工程模式

> 凡是需要跨步骤、跨代理、跨失败恢复的任务，都应该有类似的外部状态机，而不是只依赖 prompt 里的计划。

---

## 核心定义

**Manifest 驱动工作流**：把任务的依赖关系、输入输出、来源信息、派生规则、完成状态持久化为结构化文件（manifest），让任务可编排、可恢复、可 diff、可被其他脚本重新验证。

关键假设：**对话上下文会压缩、会被遗忘、会被新的推理覆盖；manifest 文件不会。**

---

## 为什么需要 Manifest

| 问题 | 只靠 prompt | 有 manifest |
|------|-----------|------------|
| 中途断开 | 丢失进度 | 读取 manifest 恢复 |
| 并行任务 | 依赖关系不清楚 | `depends_on` 字段显式声明 |
| 失败修复 | 重跑整个流程 | 定位失败 job，局部修复 |
| 来源审计 | "模型说完成了" | `source_sha256` 可验证 |
| 跨代理协作 | 各自记忆混乱 | 共享 manifest，统一状态 |

---

## Manifest 结构设计

以 Codex hatch-pet 的 `imagegen-jobs.json` 为例：

```
{
  "schema_version": 1,
  "created_at": "ISO-8601",
  "run_dir": "工作目录",
  "primary_generation_skill": "$imagegen",
  "jobs": [
    {
      "id": "base",              // 唯一标识
      "kind": "row-strip",       // 任务类型
      "status": "pending",       // pending | complete
      "prompt_file": "路径",     // 使用的 prompt
      "input_images": [...],     // 参考输入
      "output_path": "路径",     // 产物位置
      "depends_on": [],          // 依赖的前置 job
      "parallelizable_after": [],// 何时可并行
      "source_sha256": "hash",   // 来源校验
      "output_sha256": "hash",   // 产物校验
      "completed_at": "时间"     // 完成时间
    }
  ]
}
```

### 关键字段解析

**任务编排字段**：
- `depends_on`：显式声明依赖，构建任务 DAG
- `parallelizable_after`：标记可并行执行的起点
- `status`：持久化状态（注意：`blocked` 是派生视图，不是持久化状态）

**来源溯源字段**：
- `source_path`：生成来源
- `source_provenance`：来源类型记录
- `source_sha256` / `output_sha256`：hash 校验，防止伪造

**约束字段**：
- `generation_skill`：用什么工具生成
- `requires_grounded_generation`：是否需要参考图像
- `allow_prompt_only_generation`：是否允许纯 prompt
- `mirror_policy`：特殊派生规则（如水平镜像）

---

## 子代理写权限隔离

Manifest 驱动工作流的核心安全模式：

**子代理（Worker Plane）可以**：
- 读取 row prompt
- 调用生成工具
- 检查候选产物
- 返回选择结果和 QA note

**子代理不可以**：
- 修改 manifest
- 拷贝文件到产物目录
- 执行来源记录脚本
- 镜像派生产物
- 执行 finalize 和 package

**父代理（Control Plane）独占**：
- manifest 写入
- 来源记录（provenance）
- 产物打包
- 最终提交

> **规则**：并行任务可以分发，truth commit 必须集中。

这避免了并行写冲突、来源混乱和 provenance 污染。参见 [[concept_multiagent综述]] 中的并行分工模式。

---

## Manifest 的工程属性

### 可恢复性
每个 job 有独立状态，断点后读取 manifest 知道从哪继续。

### 可验证性
hash 校验保证产物未被篡改；finalize 时必须校验所有 `source_sha256`。

### 可 diff 性
manifest 是文本文件，可以 `git diff` 看任务状态变化。

### 可委托性
其他脚本或 Agent 可以读取 manifest 判断任务进度，无需对话上下文。

---

## 与传统状态管理的对比

| 方式 | 状态位置 | 恢复能力 | 并发安全 | 审计能力 |
|------|---------|---------|---------|---------|
| 对话上下文 | 内存 | 丢失 | 无 | 无 |
| 数据库 | 远程 | 需要连接 | 需要锁 | 有 |
| Manifest 文件 | 本地文件系统 | 原子读写 | 父代理独占写 | hash + provenance |

Manifest 适合：本地 Agent 任务编排、资产生成流水线、需要离线恢复的长任务。
数据库适合：多人协作、高并发、需要事务保证的场景。

---

## 可迁移的应用场景

Manifest 驱动模式不仅适用于图像生成，任何需要跨步骤、跨代理、跨失败恢复的任务都适用：

- **代码生成**：编译 manifest → 各模块生成状态 → 编译结果 → 测试结果
- **知识库构建**：ingest manifest → 各文件处理状态 → QA 结果 → 修复记录
- **文档生成**：各章节生成状态 → 交叉引用验证 → 一致性检查
- **测试执行**：测试用例状态 → 覆盖率 → 失败记录 → 修复追踪

---

## Related Concepts

- [[concept_skill_engineering]] — Manifest 是 Skill 工程的核心控制结构
- [[concept_harness_engineering]] — Manifest 驱动是 Harness 思维的具体实现：状态外部化
- [[concept_asset_pipeline_and_qa]] — Manifest 记录的产物如何经过流水线处理
- [[concept_multiagent综述]] — 子代理写权限隔离的设计模式
- [[concept_ai系统设计原则]] — 状态管理在四层信息架构中的位置
- [[concept_agent认知循环]] — 外部状态如何支撑 Agent 认知循环

---

## References

- [hatch-pet skill — imagegen-jobs.json](https://github.com/openai/skills/tree/main/skills/.curated/hatch-pet)
