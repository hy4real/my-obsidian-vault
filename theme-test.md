---
type: theme-test
status: active
area: "[[主题设计]]"
tags:
  - theme-test
  - obsidian
  - claude-theme
created: 2026-05-11
rating: 4.5
enabled: true
aliases:
  - Claude 主题测试页
---
# Claude 主题测试页

这是一页用于检查 Obsidian 主题整体设计的测试文档。它覆盖正文、标题、链接、标签、表格、代码块、callout、引用、任务列表、数学公式、脚注和分隔线等常见组件。

## 正文层级

正文应该保持舒适的阅读宽度、稳定的行高和温和的段落间距。这里放一段稍长的文字，用来观察中文、英文与数字混排的质感：Claude theme should feel warm, quiet, readable, and precise. 例如 `inline code`、**加粗文字**、*斜体文字*、==高亮文字==、~~删除线文字~~ 都应该和正文自然融合。

### 三级标题

三级标题用于普通章节内部的小节，视觉上应该比正文明确，但不应该抢过二级标题。

#### 四级标题

四级标题通常出现在较细的结构里，适合测试标题权重、间距和字号。

##### 五级标题

五级标题用于低层级结构。

###### 六级标题

六级标题用于最弱标题层级。

## 链接与标签

内部链接：[[主题设计]]、[[不存在的测试页面]]、[[theme-test#表格测试|跳到表格测试]]

外部链接：[Obsidian 官网](https://obsidian.md)、[Claude](https://claude.ai)

标签测试： #theme-test #Claude主题 #设计/组件 #状态/待检查

## 列表与任务

- 一级无序列表项目
- 带有较长内容的项目，用来观察换行后的缩进、行距和文本密度是否稳定。
  - 二级列表项目
  - 另一个二级项目，包含 `inline code` 和 [[主题设计]]
- 最后一项

1. 第一项有序列表
2. 第二项有序列表
3. 第三项有序列表

- [x] 已完成任务
- [ ] 未完成任务
- [>] 推迟任务
- [!] 需要注意的任务

## 引用与分隔线

> 引用块应该低调、清晰，并且和 callout 有明显区分。
> 
> 第二行引用用于观察段落间距。

---

分隔线之后的正文应该仍然保持稳定间距。

## Callout 测试

> [!note] Note
> 适合普通说明。背景、边框、标题和图标应该统一但不刺眼。

> [!info] Info
> 用于信息提示。这个类型也会覆盖默认的 `todo` 视觉变量。

> [!tip] Tip
> 用于经验、技巧、建议。应与 note 明显不同，但仍在 Claude 暖色体系内。

> [!quote] Quote
> 用于摘录、引用和外部材料。它应该比普通引用块更有组件感。

> [!summary] Summary
> 用于总结、摘要、TLDR。颜色应该和 info、note、tip、quote 不完全相同。

> [!success] Success
> 用于完成、检查通过、验证成功。

> [!warning] Warning
> 用于风险提醒。应该醒目，但不要变成高饱和警告块。

> [!danger] Danger
> 用于严重错误或破坏性风险。

> [!question]- 折叠 Callout
> 这是默认折叠的内容。展开后可以检查标题、图标和内容间距。

> [!tip] 嵌套 Callout
> 外层 callout 内容。
> > [!note] 内层 Note
> > 嵌套 callout 用来检查边距、层级和背景叠加效果。

## 表格测试

| 项目 | 状态 | 数值 | 说明 |
| --- | --- | ---: | --- |
| Properties 面板 | 已优化 | 90% | 顶部信息区应是克制暖纸色，不应出现白底输入块 |
| Callout | 已优化 | 85% | 不同类型颜色应不同，组件结构统一 |
| 代码块 | 已调整 | 80% | 代码块面板保留暖色背景，语法高亮恢复旧版 |
| 链接与标签 | 已优化 | 75% | 内链、外链、标签 pill 应属于同一套暖色逻辑 |

## 代码块测试

行内代码测试：`const theme = "Claude";`、`--claude-accent: #d97757;`

```js
const palette = {
  accent: "#d97757",
  surface: "#faf9f5",
  codeString: "keep previous highlight",
};

function renderCallout(type, content) {
  if (!content) {
    return null;
  }

  return {
    type,
    content,
    createdAt: new Date().toISOString(),
  };
}

console.log(renderCallout("tip", "保持暖色体系，但不要牺牲可读性。"));
```

```css
.callout[data-callout] {
  border-width: 0.5px 0.5px 0.5px 3px;
  border-style: solid;
  border-radius: var(--claude-radius-medium);
  background-color: var(--claude-callout-bg);
}

.metadata-container input {
  background-color: transparent;
  box-shadow: none;
}
```

```python
from dataclasses import dataclass


@dataclass
class ThemeCheck:
    name: str
    passed: bool
    score: float


checks = [
    ThemeCheck("callout", True, 0.92),
    ThemeCheck("table", True, 0.88),
    ThemeCheck("properties", True, 0.9),
]

for check in checks:
    print(f"{check.name}: {check.score:.0%}")
```

```yaml
theme:
  name: Claude
  focus:
    - reading
    - callout
    - properties
    - table
  keep_previous_code_highlight: true
```

## 数学公式

行内公式：$E = mc^2$，以及 $\alpha + \beta = \gamma$。

块级公式：

$$
\int_0^1 x^2 \, dx = \frac{1}{3}
$$

## 图片与嵌入占位

下面是 Obsidian 嵌入语法的占位，用于检查链接、嵌入文字和未解析资源的样式。


![[1.png]]




![[不存在的测试笔记]]

## 脚注

这是一句带脚注的正文，用来检查脚注标记、链接颜色和跳转后的视觉层级。[^theme-note]

[^theme-note]: 这是脚注内容。它应该保持清晰，但不要比正文更醒目。

## 长段落压力测试

当一段文字比较长时，主题应该仍然保持舒适的阅读节奏。这里故意放入一段更接近日常笔记的内容：一个好的 Obsidian 主题不是只在截图里好看，它还需要在真实写作、快速扫描、反复编辑、移动端阅读和大量链接跳转时保持稳定。标题不能过重，正文不能过淡，代码不能像贴上去的外来组件，Properties 也不能变成突兀的白色表单。

## 检查清单

- [ ] 顶部 Properties 区没有白底输入块
- [ ] `note / tip / quote / summary / info` 色彩不同
- [ ] 代码块背景稳定，语法高亮保持旧版观感
- [ ] 表格表头和边框不抢正文注意力
- [ ] 内链、外链、未解析链接、标签 pill 可区分
- [ ] 亮色和暗色主题都能接受
