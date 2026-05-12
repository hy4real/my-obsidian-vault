---
title: Agent 资产流水线与双层 QA — 从生成材料到可信产物
type: concept
tags: [asset-pipeline, qa, provenance, agent, production, verification]
created: 2026-05-03
updated: 2026-05-03
source: 深度解析：Codex Pet Skill（lencx, 2026-05-02）
---

# Agent 资产流水线与双层 QA — 从生成材料到可信产物

> 模型是 creative worker，不是 trusted committer。  
> 模型输出只是生产材料，必须经过记录、编译、QA、打包才成为最终资产。

---

## 核心定义

**资产流水线（Asset Pipeline）**：把模型的非确定性输出，经过来源记录（provenance）、确定性编译、双层 QA、局部修复、最终打包，转化为可信产物的生产流程。

**双层 QA**：把验收拆成结构正确性（脚本自动检查）和语义一致性（模型/人类视觉检查），承认两者不是一回事。

---

## 流水线全链路

```
模型生成候选
  ↓
来源记录（record → hash + provenance）
  ↓
确定性编译（抽帧 → 拼 atlas → 格式转换）
  ↓
结构 QA（尺寸、格式、alpha、帧数、空格）
  ↓
语义 QA（身份一致性、状态语义、视觉质量）
  ↓
局部修复（最小失败范围，非整体重跑）
  ↓
最终打包（验证 → 包装 → 写入元数据）
```

每一步都是不可跳过的——跳过任何一步，产物就不可信。

---

## 来源溯源（Provenance）

### 核心问题
"模型给了我一张图" ≠ "这张图是可追溯的资产"。

### 解决方案
每个生成产物必须记录：
- `source_path`：生成来源路径
- `source_provenance`：来源类型（$imagegen / mirror / fallback）
- `source_sha256`：来源文件 hash
- `output_sha256`：产物文件 hash
- `completed_at`：完成时间
- `metadata`：附加信息

### 反作弊约束
- 不能手改 manifest 标记完成
- 不能复制本地文件冒充生成结果
- 必须用专用脚本记录真实生成输出
- finalize 时校验 hash 一致性

> 从 agent 工程角度看：不信任模型"说完成了"，只信任文件、hash、manifest、QA 结果和可检查产物。

---

## 确定性编译器

模型处理语义（"这只宠物应该是什么样子"），脚本处理确定性（"把图片裁剪成 192x208 的格子"）。

**编译步骤示例**（hatch-pet）：
1. `extract_strip_frames.py` — 从 row strip 抽帧，移除 chroma key
2. `inspect_frames.py` — 检查每帧尺寸、透明像素、边界
3. `compose_atlas.py` — 组装 spritesheet（8列×9行，1536×1872）
4. `validate_atlas.py` — 验证最终 atlas 工程正确性
5. `package_custom_pet.py` — 打包到应用可读格式

**关键原则**：这些步骤不能由模型"假装完成"，必须是真实执行的确定性脚本。

---

## 双层 QA 设计

### 第一层：结构正确性（脚本自动检查）

| 检查项 | 说明 |
|-------|------|
| 尺寸 | 必须是 1536×1872 |
| 格式 | PNG 或 WebP |
| Alpha 通道 | 除非显式允许，必须有 |
| 已用格稀疏度 | 太稀疏则报错 |
| 未用格透明度 | 必须完全透明 |
| 近似全不透明 | 视为背景没抠干净 |

### 第二层：语义一致性（模型/人类检查）

| 检查项 | 说明 |
|-------|------|
| 身份一致性 | 不能换物种、换体型、换脸、换标记 |
| 状态语义 | review 状态是否"像 review" |
| 调色板一致性 | 各行动画保持统一配色 |
| 道具一致性 | 配饰、手持物不漂移 |

### 为什么必须分两层

> `validate_atlas.py` 能证明尺寸、透明、帧数、空格、alpha；但它不能证明"这还是同一只宠物"。

**类比**：schema validation 只能证明数据形状正确，不能证明语义正确。很多 Agent 系统只做 schema validation，然后误以为任务完成；但真实生产里，schema correct 只是底线，semantic correct 才是用户真正关心的结果。

---

## 局部修复策略

### 原则
**失败的是某一行，就修这一行，不要重做整张 sheet。**

### 修复流程
1. `queue_pet_repairs.py` — 读取 QA 失败记录，重新打开失败 row job
2. 归档旧输出（保留对比）
3. 追加 repair note（说明失败原因和修复目标）
4. 只重新生成失败的 row
5. 重新 finalize

### 何时升级为全量重跑
只有身份或布局广泛损坏时，才做 full atlas regeneration。默认走最小范围修复。

### 工程价值
图像生成高成本、不稳定——局部修复比整体重跑更可控、更便宜、漂移更小。

这个思想可迁移到任何 Agent 任务：
- 代码生成：失败的模块重写，不重写整个项目
- 知识库：有问题的页面修复，不重建整个 wiki
- 测试：失败的用例重跑，不重跑整个测试套件

---

## 产物最终形态

```bash
${CODEX_HOME:-$HOME/.codex}/pets/<pet-name>/
├── pet.json          # 元数据：id, displayName, description, spritesheetPath
└── spritesheet.webp  # Codex app 按固定背景位置读取的精灵图
```

**产物定义在先**：不是"画一只好看的宠物"，而是"产出一个 Codex app 能加载的 animated pet package"。美术只是其中一个环节，真正的目标是符合协议的 artifact。

---

## 资产流水线的普适模式

```
非确定性生成（模型）     → creative worker
确定性编译（脚本）       → asset compiler
来源记录（provenance）   → audit trail
结构 QA（自动化）       → schema validation
语义 QA（模型/人）      → semantic validation
局部修复（定向重跑）     → targeted repair
最终打包（校验+包装）   → trusted artifact
```

这个模式适用于任何需要把模型输出转化为可信产物的场景。

---

## Related Concepts

- [[concept_skill_engineering]] — 资产流水线是 Skill 工程的产物管理层
- [[concept_manifest_driven_workflow]] — 流水线的编排和状态由 manifest 驱动
- [[concept_harness_engineering]] — "模型是 creative worker，不是 trusted committer"是 Harness 思维的核心体现
- [[concept_ai系统设计原则]] — 结构 QA 对应四层机制中的"上下文"层，语义 QA 对应"外部记忆"层
- [[concept_multiagent综述]] — 流水线中的并行生成与写权限隔离

---

## References

- [hatch-pet skill 源码](https://github.com/openai/skills/tree/main/skills/.curated/hatch-pet)
