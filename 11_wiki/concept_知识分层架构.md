---
title: 知识分层架构 — 五层存储 × 五种类型 × 三级成熟度
type: concept
tags: [knowledge-management, architecture, maturity, harness-engineering, team]
created: 2026-04-28
updated: 2026-04-28
---

# 知识分层架构 — 五层存储 × 五种类型 × 三级成熟度

> 来源：腾讯技术工程，AI Team 工程交付编排系统实践（2026-04-27）  
> 背景：[[concept_harness_engineering]] 的知识管理具体实现方案

知识体系的三个正交维度：

| 维度 | 回答的问题 | 值域 |
|------|---------|------|
| 存储层（在哪） | 知识存在哪里？ | Layer 0-P / 0-T / 1 / 2 / 3 |
| 知识类型（是什么） | 知识描述的是什么？ | model / decision / guideline / pitfall / process |
| 成熟度（多可信） | 知识经过多少验证？ | draft → verified → proven |

---

## 五层存储架构

```
┌──────────────────────────────────────────────────────────┐
│                      五层知识存储                         │
├──────────┬──────────────────────────────┬────────────────┤
│ Layer 0-P│ 个人偏好 (~/.ai-team/)       │ 纯本地，不共享  │
│ Layer 0-T│ 团队约定 (team-conventions/) │ 团队级，Git 共享│
│ Layer 1  │ 技术知识 (tech-wiki/)        │ 团队级，跨项目  │
│ Layer 2  │ 业务知识 (biz-wiki/{domain}/)│ 团队级，按领域  │
│ Layer 3  │ 项目知识 (docs/knowledge/)   │ 项目级，随项目走│
└──────────┴──────────────────────────────┴────────────────┘
```

**各层说明：**

- **Layer 0-P（个人偏好）**：缩进风格、函数式/OOP 偏好——纯个人，不强制给团队
- **Layer 0-T（团队约定）**：代码规范、Commit 规范、Review 标准——团队层面的"宪法"，相对稳定
- **Layer 1（技术知识）**：跨项目通用技术经验——"Spring Boot 多租户拦截器设计模式"、"Optional 依赖传递陷阱"
- **Layer 2（业务知识）**：特定领域的领域模型、业务规则、业务流程——"广告审核：提交→机审→人审→上线"
- **Layer 3（项目知识）**：仅当前项目有意义的上下文——"本项目数据库用 TencentDB for MySQL 8.0"

**关键设计：知识可以"向上提升"**

```
Layer 3（项目内，所有类型初始 maturity = draft）
  ├→ 是否项目特有？ → 是：留在 Layer 3
  ├→ 是否通用技术？ → 是：提升到 Layer 1（tech-wiki）
  └→ 是否通用业务？ → 是：提升到 Layer 2（biz-wiki）
```

---

## 五种知识类型（MECE）

| 类型 | 定义 | 示例 |
|------|------|------|
| **model** | 实体定义、数据结构、关系图 | "广告计划包含预算/出价/投放时段三个核心字段" |
| **decision** | 技术选型、架构决策及理由 | "选择事件驱动而非 RPC 同步，因为广告状态变更需要解耦" |
| **guideline** | 推荐做法 / 禁止做法 | recommend: "公共模块变更后的兼容性检查清单" |
| **pitfall** | 已知风险、故障模式、排查步骤 | "广告预算扣减在高并发下会超扣" |
| **process** | 业务流程、状态机、操作步骤 | "广告审核：提交→机审→人审→上线" |

每条知识只属于一个类型，来源信息记录在元数据中用于溯源。

---

## 三级成熟度 + 自动衰减

### 成熟度升级路径

```
draft（新提取，单一来源）
  ↓ 在 1 个工作流中被成功引用
verified（单项目验证）
  ↓ 在 ≥2 个不同项目中被验证
proven（成熟/可信赖）
```

晋升条件：
- `draft → verified`：1 人验证 + 在 1 个工作流中成功引用
- `verified → proven`：≥2 人验证 + ≥2 个不同项目中被验证

### 自动衰减机制

知识长期不被引用会自动降级，防止过时知识误导 Agent：

| 触发条件 | 衰减动作 |
|---------|---------|
| proven 条目 12 个月未被引用 | 降级为 verified |
| verified 条目 6 个月未被引用 | 降级为 draft |
| draft 条目持续未引用 + Lint 标记 | 归档，移出活跃索引 |

> 这个设计借鉴自 [[concept_wiki系统]] 中的 Karpathy LLM Wiki Lint 操作。详见 [[guide_lint操作]]。

---

## 三级渐进式索引（按需消费）

**问题**：Agent 启动时推送大量知识会导致上下文膨胀（Harness 要解决的核心问题之一）。

**解法**：Agent 通过三层索引主动按需查阅，而非被动接收推送。

| 层级 | 文件 | 大小 | 作用 |
|------|------|------|------|
| Layer A：全景目录 | `knowledge-catalog.md` | ~50 行 | "知识库有什么？"——分类统计 + 阶段推荐路径 |
| Layer B：分类清单 | 各目录下 `catalog.md` | ~100-300 行 | "这个分类有哪些条目？"——每条一行摘要（ID + 标题 + 成熟度 + 标签） |
| Layer C：完整条目 | `TK-*.md` / `BK-*.md` | ~50-200 行 | "这条知识说了什么？"——完整内容 + 背景 + 适用场景 |

渐进查询流程：
```
Step 1: 读全景目录（~50 行，零成本）→ 定位推荐的 catalog.md 路径
Step 2: 读分类清单（~300 行，低成本）→ 按 tags/phases 过滤相关条目
Step 3: 读完整条目（按需，每条 50-200 行）→ 获取完整知识内容
Step 4: 读原始产物（深入，可选）→ 沿 source_references 追溯推导过程
```

效果：用 ~50 行了解全貌，用 ~300 行定位相关条目，只在真正需要时读完整内容——对比"一次性推送 50 条完整知识"（5000-10000 行），上下文效率提升一个数量级。

---

## 知识引用追踪闭环

Agent 查询知识后，在产物中记录引用：

```json
{
  "knowledgeReferences": [
    { "id": "TK-SB-003", "title": "分页查询延迟关联优化", "usedIn": "复用评级 Step 2" }
  ]
}
```

ARCHIVE 阶段读取所有 `knowledgeReferences`，批量更新 `evidence.last_referenced` 字段。被引用的知识 maturity 自动提升，长期未引用的自动衰减。

---

## 团队知识库：共建共享架构

**独立 Git 仓库**（不寄生于任何业务项目）：

```
team-knowledge.git
├── knowledge-catalog.md     ← 全景目录（Agent 查询入口）
├── team-conventions/        ← Layer 0-T
├── tech-wiki/               ← Layer 1
├── biz-wiki/{domain}/       ← Layer 2
├── project-profiles/        ← 项目画像
└── contributions/pending/   ← 贡献暂存区
```

**三种角色：**

| 角色 | 权限 | 适用人群 |
|------|------|---------|
| maintainer | 裁决内容冲突、审批 proven 提升、管理成员 | 团队负责人、资深工程师 |
| contributor | 通过工作流自动贡献（创建/验证/标记矛盾） | 正式团队成员 |
| reader | 只消费知识（查询/注入），不贡献 | 新成员试用期 |

**冲突解决策略：**

| 冲突类型 | 处理方式 |
|---------|---------|
| 纯新增（不同条目） | 自动合并，两条都保留 |
| 证据追加（同条目验证） | 自动合并，evidence 数组合并去重 |
| 成熟度提升 | 自动合并 |
| 内容矛盾 | 写入 conflicts/，通知 maintainer 裁决 |
| 成熟度冲突（一升一降） | 保留较低成熟度 + 标记 contradiction |

设计理念：大多数情况可以自动处理，只有真正的内容矛盾才需要人工介入，让共建过程尽可能低摩擦。

---

## 与 Karpathy Wiki 的关系

本架构是 Karpathy LLM-Wiki 模式在**团队工程交付场景**的具体化：

| 维度 | Karpathy Wiki（[[concept_wiki系统]]） | 知识分层架构 |
|------|--------------------------------------|------------|
| 使用场景 | 个人知识管理 | 团队工程交付 |
| 知识分层 | 三层（Raw/Wiki/Schema） | 五层（0-P/0-T/1/2/3） |
| 消费方式 | 人阅读 + LLM 维护 | Agent 按需查询（三级索引） |
| 生命周期 | Lint 检查矛盾/孤儿 | +自动衰减+引用追踪 |
| 共建机制 | 单人 | 多人 + Git + 角色权限 |

---

## Related Guides

- [[concept_harness_engineering]] — 本架构的理论基础：为什么知识比工作流更重要
- [[concept_wiki系统]] — Karpathy LLM-Wiki：Ingest/Query/Lint 三大操作
- [[guide_lint操作]] — 知识健康检查，含自动衰减触发条件
- [[guide_ainative研发方法论]] — propose→apply→archive 工作流，与 INIT→消费→ARCHIVE 同理
- [[concept_ai系统设计原则]] — 系统设计中的信息分层原则
- [[project_ai工程框架]] — AI 工程整体框架
