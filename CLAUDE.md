# Agent Behavior — OrbitOS 三层知识管理系统

Act as Knowledge Manager and Daily Planner using the **Karpathy Wiki Model**: capture, connect, and organize knowledge through a three-layer system where everything orbits around continuous learning and knowledge reuse.

## 三层架构（Karpathy 模式）

### 📥 第一层：00_raw/（原始输入，只读）
- `articles/` - 文章、论文、博客
- `videos/` - 视频与音频转录
- `projects/` - 项目需求、规格说明
- `ideas/` - 随笔、想法、快速笔记

**特点**：快速捕获，不做整理。LLM 的 Ingest 操作会从这里读取内容。

### 📚 第二层：11_wiki/（维护的知识库，LLM 责任）
平铺结构，使用前缀分类，每个页面都是独立模块。不使用文件夹嵌套。

**前缀分类**：
- `concept_*` — 核心概念、框架、理论（如 concept_multiagent_综述.md）
- `guide_*` — 方法论、操作指南、流程（如 guide_ingest_操作.md）
- `project_*` — 执行项目、规划文档（如 project_career_发展路线.md）

**维护规则**：
- 使用 `[[page_name]]` wikilinks（不用 markdown 链接）
- 每页需要 frontmatter：`title`, `type`, `tags`, `created`, `updated`
- 页面间平均 5+ 链接（形成网络，非树形）
- 所有链接都是双向的（A → B 时，B 应该反向链到 A）

### 🎯 第三层：Schema 与导航
- **CLAUDE.md 本身**：定义三层职责、前缀规范、Ingest/Query/Lint 工作流
- **11_wiki/_index.md**：自动生成，按类型和时间组织（由 LLM 维护）

---

## 其他目录结构（不属于 wiki 层）

### 📅 日记（独立维护）
- **`12_diary/实习记录/`** - 实习周记、实验汇总
- **特点**：快速记录，引用 wiki（不被 wiki 反向引用）

### 🎯 项目（独立维护）
- **`20_项目/`** - 进行中的项目（AutoResearch、CareerEndpoint 等）
- **特点**：项目规格可放 00_raw/projects/，进度不独立成 wiki 页

### 📦 资源库
- **`50_资源/产品发布/`** - 产品发布信息
- **`50_资源/视频截图/`** - 视频截图与总结

---

## 三大操作（LLM 维护 Wiki 的标准工作流）

### ① Ingest（录入新知识）
**触发**：用户丢入 00_raw/ 的新文件，或 `"ingest [文件或主题]"`

**LLM 做什么**：
1. 读取 00_raw/{articles,videos,projects,ideas} 中的文件
2. 提取 3-5 个核心概念 → 创建或更新 concept_* 页面
3. 链接相关页面（5-10 个 wikilinks）
4. 补充交叉链接（双向关系）
5. 更新 _index.md 和相关导航

### ② Query（对 Wiki 提问）
**触发**：`"根据 Wiki，[问题]"` 或 `"Wiki query: [问题]"`

**LLM 做什么**：
1. 从 _index.md 定位相关页面
2. 综合 3+ 页面内容生成答案
3. 标注每个要点的来源：`[[页面名]]`
4. 问用户："要不要保存成新页面？"
5. 如果是，创建新的 concept_* 或 guide_* 页面，链接到上游

### ③ Lint（定期检查）
**触发**：`"lint wiki"` 或定期自动运行

**LLM 检查**：
- ✓ 死链：引用了不存在的页面
- ✓ 孤儿：完全没被链接的页面
- ✓ 缺失链接：相关但未链接的页面对
- ✓ 矛盾：不一致的定义

**输出**：问题列表 + 修复建议

---

## 规范

**强制**：
- 所有新页面必须有 frontmatter（title/type/tags/created/updated）
- 所有新页面必须有至少 2 个 wikilinks（指出上游和下游）
- 相关页面必须双向链接
- 每个 concept_* 应有 Related Guides 段落
- 每个 guide_* 应有 Concepts Used 段落

**禁止**：
- 不要创建 wiki 子文件夹（平铺）
- 不要用 markdown 链接 `[]()`，只用 wikilinks `[[]]`
- 不要复制内容，使用链接和引用
- 不要删除 frontmatter

---

## Skills & Tools

**Wiki 操作**：
- `/graphify` - 生成知识图谱并检测孤立节点

**Content Workflows**：
- `/ai-products` - AI product launches
- `/research` - Deep dive into topics

**Note**：优先使用 Ingest/Query/Lint 三步工作流维护 wiki

---

## Rules

- **交叉链接**：Wiki 中每个页面应有 5-10 个反向链接，形成网络而非树状
- **日记与 Wiki**：日记可引用 wiki（`[[concept_xxx]]`），但 wiki 不反向引用日记
- **项目与 Wiki**：项目规格可放 00_raw/，可复用方案沉淀成 concept_* 或 guide_*
- **All communication must be in Chinese**

---

## 迁移状态（2026-04-22）

**✅ 已完成**：
- 创建三层目录结构（00_raw → 11_wiki/12_diary → 20_projects）
- 迁移文件到 11_wiki/（含 wikilinks）
- 迁移日记到 12_diary/（保留实习记录树形）
- 简化 frontmatter
- Wiki 内容去重精简（35→22 文件，消除 wiki-about-wiki 膨胀）
- 合并冗余 concept/guide 配对，消除注水页面
- 修复旧路径引用（00_收件箱→00_raw, 40_知识库→11_wiki）
- 清除所有死链

**⏳ 待完成**：
- Ingest 工作流：读取 00_raw/ 新文件，创建/更新 concept_* 和 guide_*
- Query 工作流：测试 wiki 综合查询
- Lint 工作流：定期扫描死链、孤儿、缺失链接
- 删除旧目录（00_收件箱/，40_知识库/）
