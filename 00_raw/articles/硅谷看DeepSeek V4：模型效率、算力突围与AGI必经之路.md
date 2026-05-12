---
title: "硅谷看DeepSeek V4：模型效率、算力突围与AGI必经之路"
tags: [AI, DeepSeek, 大模型, 硅谷, 芯片, token-efficiency, AGI, 开源]
source: https://www.bilibili.com/video/BV1Kq9SBSE7N
author: 硅谷101
date: 2026-04-29
duration: 1:10:59
type: 视频笔记
platform: bilibili
transcript_source: asr
---

# 硅谷看DeepSeek V4：模型效率、算力突围与AGI必经之路

> [!info] 视频信息
> 硅谷101 / 1:10:59 / 2026-04-29 / [链接](https://www.bilibili.com/video/BV1Kq9SBSE7N)

## 核心观点

DeepSeek V4 的意义不在于又出了一个强模型，而是揭示了大模型竞争正从单点 benchmark 转变为系统竞争：模型架构、token efficiency、芯片适配、商业化、开源生态已成为同一场战争的不同战场。token efficiency 是达到 AGI 的必备之路——没有效率，AGI 就只能是个 demo；有了效率，AGI 才能成为真正的产品和基础设施。

## 嘉宾介绍

- **肖志斌**：资深芯片架构师，ZFLOW AI 创始人兼 CEO，华美半导体协会前主席
- **Jenny Xiao**：OpenAI 前研究员，Leonis Capital 合伙人

## DeepSeek V4 的三大核心技术

DeepSeek 论文重点强调的三个技术突破，共同实现了 token efficiency 的新高度：

### 1. CSA + HCA：混合注意力机制

解决长上下文推理中 attention 与 KV cache 的成本问题（原本是平方关系）。

- **CSA（Compressed Sparse Attention）**：将多个 token 的 KV cache 压缩为 compressed KV engine，再通过 sparse attention 选择最相关的部分做相关性检索，相当于精确搜索
- **HCA（Hierarchical Compressed Attention）**：对当前 token 之前的 context 做深度压缩后进行 dense attention，相当于全局快速检索
- **Sliding Window**：保证与最近 token 的强相关性不丢失

三者结合实现长上下文的 attention 压缩，**降低推理成本**。

### 2. MHC：Manifold Constrained Hyper Connections

升级 Transformer 的层间信息流转机制。

- 原来的 ResNet link 是单条高速通路
- Hyper Connections 做了多条高速通路
- Manifold Constrain 在多层 residual connection 上加约束，保证数值稳定性

**提升复杂生成架构的训练稳定性**。

### 3. MUON 优化器

提升 AI 模型训练的收敛速度和稳定性，可以训练更大更复杂的模型。部分模块仍使用 Adam 优化器，两者配合使用。

**提升训练的收敛效率和稳定性**。

> [!tip] 总结
> CSA+HCA 降低推理成本 → MHC 提升训练稳定性 → MUON 提升训练效率，三者叠加实现 token efficiency 新高度。

## Token Efficiency：AGI 的必经之路

### 效率即智能

- Token efficiency 不是 AGI 的对立面，而是**基础条件**
- 未来更强的模型需要更多 test-time compute、更长 reasoning、更长上下文、更复杂的工具调用
- 如果每个 token 都很贵，模型就不能长时间思考，也不能大规模服务用户
- Agent 时代 token 消耗是 Chatbot 的 10-100 倍，没有效率就无法大规模商业化

### 中美模型厂商的路线差异

| 维度 | 硅谷头部公司 | 中国模型厂商 |
|------|------------|------------|
| 优先级 | 更强模型、更好智能 | 更高效率、更低成本 |
| 资源 | 更多算力、资本、资源 | 资源约束下创新 |
| 追求 | Reasoning、多模态、Agent、安全、产品生态 | MoE 架构、上下文压缩、低成本推理、非英伟达适配 |
| 价格 | API 价格较高 | API 价格极低 |

中国厂商被资源倒逼，更早进入追求 token efficiency 的创新阶段。硅谷厂商在模型智能上仍有领先性，但也越来越重视效率。

### DeepSeek 带给硅谷的三大压力

1. **价格压力**：企业客户关心的是每个任务多少钱，而非模型参数多大。同等能力下 DeepSeek 价格极低，闭源 API 面临重新定价
2. **长上下文压力**：DeepSeek 将长上下文与低 KV cache 成本结合，其他模型公司的长上下文方案面临挑战
3. **Agent 成本压力**：推理成本降低可大幅降低 Agentic workload 的 token 消耗量

## 芯片与算力生态

### 英伟达的优势与挑战

**短期不会被取代**：英伟达的优势不仅是 GPU，更是 CUDA 生态、通信库、NVLink、InfiniBand、开发者生态、成熟供应链。

**长期会走向异构**：
- DeepSeek 降低了长上下文推理对 compute 和 KV cache 的要求，让非英伟达芯片有机会承接推理 workload
- 训练方面 GPU 仍是 default，推理方面会出现越来越多专用芯片
- Google TPU 已证明：通过全栈软件设计可以降低对英伟达的依赖

### 未来芯片分化趋势

不同 workload 对计算、存储、通信的需求完全不同，芯片将走向专用化：

| Workload | 核心需求 |
|----------|---------|
| 训练 | 大规模集群、高带宽互联、高可靠性 |
| 推理 | 低成本、低延迟、高吞吐量 |
| 长上下文 | 更大 KV cache、精准的 memory hierarchy |
| Agent | 小 batch、长时间、频繁工具调用、低延迟响应 |

**未来常态**：训练在英伟达 GPU，部署在国产/非英伟达芯片，Agent workload 在专用芯片上——数据中心推理将越来越异构。

### 国产芯片适配的五大难点

1. **算子层面**：需要 kernel 级支持（fused MoE、sparse attention、低精度等），DeepSeek 使用 Triton 编程语言做优化
2. **通信层面**：MoE 模型需要 dispatch、all-to-all 等通信支持，通信做不好会被拖垮
3. **Serving 运行时**：需要适配 continuous batching、PD 分离、KV cache 管理等
4. **训练稳定性**：大规模训练需要长时间稳定运行，对容错、checkpoint、数值一致性要求极高
5. **开发者生态**：编译器、profiler、通信库等整个 AI infra software stack 需要成熟

> [!note] 关键判断
> 推理适配会进展较快，训练适配会慢很多。非英伟达芯片需要补的不是一个点，而是整个 AI infra software stack。

## Silicon Valley 视角：Jenny Xiao 的分析

### DeepSeek 的硅谷反应

- V3/R1 是 Silicon Valley 的"DeepSeek moment"，当时很少有人相信中国能训练前沿模型
- V4 反应规模不如上次，但态度从质疑转向认可
- 被视为优秀的工程突破，尤其是效率方面
- 中国模型在"又好又便宜"方面显著领先美国基础模型实验室

### 开源模型对闭源的结构性威胁

> [!quote] Jenny Xiao
> The biggest risk of DeepSeek is that it creates a death zone / kill line for foundation model companies in the US. If you're a foundation model company and you get surpassed by an open source company, the value of your business is essentially zero.

- AI 商业模型非常二元：如果 Anthropic 不是编码最强的，谁还会用 Claude Code？收入会归零
- 开源模型每次出现，二级模型的定价能力就会被削弱
- 80% 的任务运行在小型开源模型上，只有 20% 最复杂的任务使用闭源模型
- 未来 12 个月可能出现拐点：额外智能的边际价值消失

### Anthropic vs OpenAI

**Anthropic 崛起的三个原因**：
1. Claude Code 是定义性产品，驱动了大部分收入
2. 企业信任：安全承诺 + 成熟形象
3. 对比 OpenAI 的混乱，Anthropic 更像"房间里的成年人"

**Anthropic 的优势**：专注——只聚焦安全、企业、编码。不追求视频/图像生成。

**OpenAI 的问题**：想做一切（everything platform），导致技术领先丧失。GPT-5.5 性能很好但太贵（$180/M tokens）。

**估值差异**：Anthropic 在二级市场超越 OpenAI 达到 $1 万亿，因为企业收入被视为更粘性、更高质量。

### 为什么编码是 AI 最重要的战场

- 编码是控制计算机的根本方式——视频编辑、网站设计本质上都是编码任务
- 编码可衡量、数据量大、工程师是最早采用新技术的群体
- 谁拥有编码市场，谁就可能在 AGI 竞赛中占主导

### 美国开源 vs 中国开源

- Jenny 认为美国开源模型（包括 Meta Llama）几乎没有机会与中国开源竞争
- 中国开发者非常聪明，开源所有模型（包括架构），鼓励全球开发
- 迭代速度令人惊叹

### 资本市场与 IPO

- 机构对 Anthropic 的兴趣远大于 OpenAI
- 很多机构在抛售 OpenAI 份额，每天都有人问是否要买
- OpenAI 的问题：花钱太多，同样的收入 Anthropic 资本效率高得多
- Anthropic 的哲学：从第一天起就保持克制，不过度承诺 GPU 和基础设施支出

## 个人思考

- [ ] 关注 DeepSeek V4 的开源权重发布节奏
- [ ] 跟踪非英伟达芯片（华为昇腾、Google TU）在推理场景的实际部署进展
- [ ] 评估 token efficiency 优化在实际 Agent 工作流中的应用
- [ ] 关注 Anthropic vs OpenAI IPO 的市场反应

## Related Concepts（Wiki 沉淀）

- [[concept_token_efficiency]] — Token Efficiency 作为 AGI 基础条件（CSA+HCA+MHC+MUON 技术栈）
- [[concept_ai芯片生态]] — AI 芯片生态与异构计算趋势（英伟达护城河、国产芯片五大难点）
- [[concept_开源闭源AI竞争]] — 开源 Kill Zone 效应与 Anthropic vs OpenAI 格局
- [[concept_multiagent综述]] — Agent 时代的 token 消耗为什么是系统瓶颈
- [[concept_竞争力框架]] — AI 竞争格局变化对个人职业路线的影响

## 原始转录

> [!note]-
> 转录内容过长（46507 字符），未保存。可从 [原视频](https://www.bilibili.com/video/BV1Kq9SBSE7N) 获取。
