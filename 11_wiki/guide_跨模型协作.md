---
title: 跨模型工作流——Claude + Codex 协作工程
type: wiki
tags: [multi-model-workflow, claude, codex, engineering-practice]
created: 2026-04-09
updated: 2026-04-10
---
# 跨模型工作流——Claude + Codex 协作工程

## 核心理念

**多模型异步验证**：充分发挥 Claude Opus（创意规划）和 GPT（代码审查）的优势，通过 4 阶段流程确保高质量产出。

---

## 工作流全景

```
PLAN (Opus 4.6)
    ↓
QA REVIEW (Codex/GPT-5.4)
    ↓
IMPLEMENT (Opus 4.6)
    ↓
VERIFY (Codex/GPT-5.4)
```

---

## 4 个阶段详解

### STEP 1：规划阶段（Plan）
**执行者**：Claude Code + Opus 4.6 | **工具**：Plan Mode

**输入**：功能需求、技术约束、项目背景

**任务**：
- 使用 Plan Mode 进入结构化思考
- 通过 `AskUserQuestion` 与用户交互，确认需求边界
- 输出**分阶段的实现计划**（带测试关卡）
- 定义关键检查点

**产出**：`plans/{feature-name}.md`
```markdown
## Phase 1: Data Model
- [ ] Define schema
- [ ] Test: schema validation

## Phase 2: API Layer
- [ ] Create endpoints
- [ ] Test: endpoint responses

## Phase 2.5: Code Quality (Codex Finding)
- [ ] Address security concerns
- [ ] Refactor for performance
```

**关键原则**：
- 拆成 3-5 个小阶段，每个阶段控制在 **一个文件或函数级别**
- 每个阶段的最后写 **测试关卡**，成为验收标准
- 不要一口气规划整个项目，预留空间给 QA 反馈

---

### STEP 2：QA 审查（QA Review）
**执行者**：Codex CLI + GPT-5.4 | **环境**：Terminal 2

**输入**：规划文档 `plans/{feature-name}.md`、当前代码库

**任务**：
- 用 Codex CLI 逐行审视计划与代码现状
- 用 "Codex Finding" 标签插入中间检查点（Phase 2.5）
- **不重写原计划**，只补充和调整
- 检查点：
  - 安全风险（SQL 注入、认证缺陷）
  - 性能隐患（N+1 查询、内存泄漏）
  - 测试覆盖度（关键路径是否有测试）

**产出**：`plans/{feature-name}.md` (已增强)
```markdown
## Phase 2.5: Code Quality (Codex Finding)
- [ ] Sanitize input to prevent XSS
- [ ] Add rate limiting on POST endpoints
- [ ] Test: auth bypass scenarios
```

**关键原则**：
- Codex 的角色是"**守门人**"，不是"**重写者**"
- 发现问题后，返回给 Opus 处理实现
- 建立信任反馈循环，避免重复审查

---

### STEP 3：实现阶段（Implement）
**执行者**：Claude Code + Opus 4.6 | **工具**：新会话

**输入**：增强后的计划 `plans/{feature-name}.md`

**任务**：
- 按阶段逐步实现
- **测试驱动**：每个阶段先写测试用例，再写实现
- 每个测试关卡通过后才进入下一阶段
- 模块化：拆成小函数，控制单个函数 < 50 行

**产出**：代码文件 + 测试文件

**关键原则**：
- Opus 在 **小范围内** 代码质量最高，大范围容易丢细节
- 利用测试本身作为 **最好的文档**
- 发现问题时，改实现，不改测试（除非测试本身错误）

---

### STEP 4：最终验证（Verify）
**执行者**：Codex CLI + GPT-5.4 | **环境**：Terminal 2 (新会话)

**任务**：
- 综合验证实现与计划的一致性
- 检查端到端流程完整性
- 最后 10% 的隐蔽问题：
  - 并发竞态条件
  - 边界条件处理
  - 依赖版本兼容性

**产出**：验证报告（问题列表）

**关键原则**：
- 这是最后的防线，禁用自动修复
- 发现的问题返回 Opus 处理

---

## 与 Vibecoding 最佳实践的融合

### 1. CLAUDE.md 先行（规划前置）
在 STEP 1 前，务必完善项目的 CLAUDE.md：
```markdown
## 技术栈
- Backend: Node.js + Express
- DB: PostgreSQL
- Testing: Jest

## 目录结构
src/
  ├── models/
  ├── routes/
  ├── middleware/
  └── tests/

## 代码规范
- 函数长度 < 50 行
- 命名：camelCase for variables, PascalCase for classes
- 错误处理：显式 try/catch，不吞错误
```

Opus 在 STEP 1 规划时会参考这些约束，减少返工。

### 2. 自然语言表达（规划阶段）
不要上来就要求写业务代码。用一段自然语言描述：
```
"我要实现一个用户认证系统，支持 JWT，有刷新令牌机制，
需要防止 CSRF，密码用 bcrypt 加密，登出时黑名单令牌。"
```

让 Opus 先输出**架构蓝图**，审完再动手。

### 3. 测试驱动（STEP 3）
**关键点**：测试是 Opus 的验收标准。
```javascript
// test/auth.test.js (先写这个)
describe('authenticate', () => {
  it('should return JWT on valid credentials', () => {});
  it('should reject invalid password', () => {});
  it('should prevent SQL injection', () => {});
});
```

Opus 看到测试用例，自动对齐实现逻辑。

### 4. Code Review 不能省（STEP 2 & 4）
Codex 的两次审查：
- **STEP 2**：计划评审，提前暴露风险
- **STEP 4**：实现评审，最后把关

两次都不应该跳过。

### 5. 上下文管理（三个会话）
- **会话 1**（STEP 1）：规划，Opus Plan Mode
- **会话 2**（STEP 3）：实现，新的 Opus 会话，干净上下文
- **Codex**（STEP 2 & 4）：专门的 Terminal，避免污染 Claude 上下文

---

## 快速检查清单

### STEP 1 完成后 ✓
- [ ] 计划分成 3-5 个小阶段
- [ ] 每个阶段有明确的测试关卡
- [ ] 关键约束（安全、性能）已标记

### STEP 2 完成后 ✓
- [ ] 没有改写原计划，只补充了 Phase 2.5
- [ ] Codex Finding 的问题清单明确
- [ ] 返回给 Opus 继续 STEP 3

### STEP 3 完成后 ✓
- [ ] 所有测试关卡都通过
- [ ] 代码遵循 CLAUDE.md 的规范
- [ ] 每个函数 < 50 行
- [ ] 没有硬编码的秘钥或配置

### STEP 4 完成后 ✓
- [ ] 没有新的 P0 问题
- [ ] P1 问题已列明并优先化
- [ ] 可以合并到 main 分支

---

## 何时用这个工作流

✅ **适用场景**：
- 新功能模块（>500 行代码）
- 安全敏感的系统（认证、支付、数据保护）
- 多人协作项目，需要明确的质量关卡
- 对性能有要求的后端服务

❌ **不适用场景**：
- 快速原型（<100 行代码）
- 单人短期项目
- 配置文件修改
- 文档更新

---

## 常见陷阱

### 陷阱 1：跳过 STEP 2
**后果**：Opus 在 STEP 3 重复审查，效率低下
**修复**：哪怕只有 15 分钟，也要让 Codex 过一遍计划

### 陷阱 2：STEP 2 改写整个计划
**后果**：Opus 看不到一致的目标，实现偏差大
**修复**：Codex 只补充中间检查点 (Phase 2.5)，不重写

### 陷阱 3：实现时不写测试
**后果**：Opus 没有验收标准，代码漂移，后期返工多
**修复**：强制 TDD：测试 → 实现 → 测试通过

### 陷阱 4：STEP 3 一个会话做完所有代码
**后果**：上下文污染，后面的函数质量下降
**修复**：大功能拆成多个会话，每个会话单个文件或 2-3 个相关函数

### 陷阱 5：忽视 CLAUDE.md
**后果**：Opus 凭空瞎写，代码风格混乱，难以维护
**修复**：把项目规范写死在 CLAUDE.md，Opus 会严格遵守

---

## 相关笔记
- [[guide_ainative研发方法论]] - AI Native方法论
- [[20_项目/AutoResearch]] - 项目应用


## See Also
- [[guide_ainative研发方法论]]
- [[concept_ai系统设计原则]]

## Backlinks

- [[guide_ingest操作]]
- [[guide_query操作]]
- [[concept_wiki系统]]
