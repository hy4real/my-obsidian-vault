
**分享人：王瀚洋**

---

## 概述

Autoresearch 是 Andrej Karpathy 于 2026 年 3 月发布的 AI 自动化研究系统，基于 nanochat（单 GPU LLM 训练框架）构建。核心机制：

- AI 代理读取 `program.md` 行为指令，修改 `train.py` 并提交 Git
    
- 每轮实验固定 **5 分钟训练预算**，用 `val_bpb`（越低越优）评估
    
- 改进则保留 commit，劣化则 `git reset --hard` 回滚，结果记录至 `results.tsv`
    
- 每小时约 12 次实验，过夜可达 100 次
    

### 仓库结构

 autoresearch-macos/  
 ├── program.md              # AI 行为指令（只读）  
 ├── train.py                # GPT 训练代码（✎ Agent 唯一可改文件）  
 ├── prepare.py              # 数据下载与分词器准备（只读）  
 ├── launch.sh               # 一键初始化并启动实验  
 ├── results.tsv             # 实验结果日志（✎ Agent 追加写入）  
 ├── run.log                 # 每次训练的输出日志（✎ Agent 覆盖写入）  
 ├── analysis.ipynb          # 结果可视化分析  
 ├── pyproject.toml          # Python 依赖声明  
 ├── CLAUDE.md               # launch.sh 按会话生成的补充指令  
 └── .claude/settings.local.json  # Claude Code 权限配置

### 实验循环流程

 读取 program.md + results.tsv → 对 train.py 提出改动 → git commit  
   → uv run train.py（5 分钟预算）→ 提取 val_bpb  
   → 下降？ 保留(KEEP) / 未下降？ git reset(DISCARD) → 记录 results.tsv → 下一轮

### train.py 内部结构

包含完整的模型定义、优化器和训练循环：

- **模型架构**：Transformer 解码器（GQA、RoPE、滑动窗口、ReLU²、Logit Softcap）
    
- **优化器（`MuonAdamW`）**：矩阵参数用 Muon，嵌入层用 AdamW
    
- **训练循环**：梯度累积、warmup + cosine warmdown 调度、loss 爆炸自动终止
    

### results.tsv 字段

|字段|说明|
|---|---|
|`commit`|Git hash，精确还原代码状态|
|`val_bpb`|验证集 bits per byte（核心优化目标）|
|`memory_gb`|峰值显存（GB）|
|`status`|`KEEP` / `DISCARD` / `CRASH`|
|`description`|Agent 对本次改动的描述|

### 常见问题

**`val_bpb` 是什么？** Validation bits per byte，衡量模型对验证数据的压缩效率，值越低越好。通过字节数归一化，与词表大小无关，不同架构/分词器的结果可直接对比。

**实验成功率？** 大多数失败（正常），100 次中约 10-20 次改进。AI 自动保留改进、丢弃劣化。

### 相关链接

- [Mac 版本仓库](https://github.com/miolini/autoresearch-macos)（可从此找到原仓库）
    
- [Cursor](https://cursor.com) | [Claude Code](https://code.claude.com) | [Claude Code 权限配置](https://code.claude.com/docs/en/permissions#bash)
    

---

## macOS 实验

### 环境与配置

|项目|规格|
|---|---|
|设备|MacBook Pro 2021|
|芯片|Apple M1 Pro|
|内存|16 GB|
|Claude 订阅|Pro Plan|
|模型|Sonnet 4.6（`/model Sonnet 4.6`）|
|推理强度|medium（`/effort medium`）|

选择 Sonnet + medium 而非 Opus + high：每轮决策逻辑简单（读结果→提改动→跑训练），**迭代频率的总体收益远大于单次推理深度的边际收益**。

### 权限配置

Claude Code 默认会对 `git add`、`git reset` 等操作弹出权限确认，即使在 prompt 中要求全自动也无法完全避免。

**初始方案**：在 `settings.local.json` 中逐条白名单（`Bash(git add:*)`、`Bash(uv run:*)` 等），可覆盖大部分操作，但白名单仅对当前会话生效，跨终端或跨会话的命令仍会触发确认。

**最终方案**：两步配置实现完全无人干预——

1. Claude Code 设置界面：Default permission mode → **auto-accept edits**
    
2. 全局配置：
    

 // ~/.claude/settings.json  
 { "permissions": { "defaultMode": "bypassPermissions" } }

### 运行方式

**方式一：`launch.sh` 一键启动**

 bash launch.sh mar18  # 自动创建分支、初始化 results.tsv、生成 CLAUDE.md、启动 Claude Code

**方式二：手动启动**

 claude  
 # 输入以下 prompt：  
 Hi have a look at program.md and let's kick off a new experiment! Let's do the setup first.  
 And don't ask questions. Just keep running fully autonomous from here on.  
 All reasonable Git operations are permitted, excluding repository deletion.

### 实验结果

35 次实验，5 次保留、30 次丢弃（保留率 14%）。`val_bpb` 从基线 1.5894 降至 1.4455（下降 9.1%），始终未突破 1.0。主要瓶颈：MPS 后端算子支持不完整、16 GB 内存限制超参搜索空间、5 分钟预算内可训练 token 数偏少。

以下选取 9 个代表性实验，展示 Agent 的五类决策模式：

|#|改动内容|Val BPB|结果|类型|
|---|---|---|---|---|
|1|Baseline（d4, dim=256, 11.5M）|1.5894|保留|起点|
|3|batch 2^{16}→2^{15}|1.5614|保留|系统性改进|
|4|batch 2^{15}→2^{14}|1.4604|保留|系统性改进|
|5|batch 2^{14}→2^{13}|1.4537|保留|系统性改进|
|7|matrix_lr 0.04→0.06|1.4680|丢弃|常规失败（LR 过高）|
|8|batch 2^{13}→2^{12}|1.5197|丢弃|过犹不及|
|9|GELU 替换 ReLU²|1.4775|丢弃|常规失败（激活函数）|
|11|final_lr_frac 0.0→0.1|1.4455|保留|精细调优（最终最优）|
|24|移除 value embeddings|1.5271|丢弃|关键组件验证|

**关键发现**：batch size 是效果最显著的改动方向（2^{16}→2^{13} 持续下降，2^{12} 反弹止损）。学习率与激活函数调整均未改进。移除 value embeddings 导致大幅恶化，反向验证其必要性。最终通过 `final_lr_frac` 微调压至 1.4455。

### 故障排除

|问题|解决方案|
|---|---|
|`command not found: uv`|关闭终端，重新打开|
|`command not found: git`|安装 Git（见上文）|
|MPS/Metal 错误|确认使用 Mac 版本而非原版|
|内存不足|GPU VRAM 不足，AI 通常自动调整|
|Claude Code 认证失败|需要付费 Claude 订阅|
|网络连接问题（下载依赖或访问外部服务失败）|中国大陆用户：配置代理或 VPN 访问 GitHub 等服务。|

---

## 总结启示

原作者 276 次实验中保留 29 次（10.5%），12 小时内将 BPB 从 0.8624 压到 0.8580，全程无人干预。

![progress](https://p.ipic.vip/quoli6.png)

核心 insight：**竞争层级上移了一层——从直接做研究，变成了构建"自动做研究的系统"**。研究本身成为可被优化的对象。

### 启示与思考

|对比维度|预训练（原版）|微调（我们的场景）|
|---|---|---|
|单次实验时长|5 分钟（需大算力）|几分钟到几小时|
|搜索空间|架构 + 优化器 + 超参|LoRA rank、学习率、数据配比、prompt 模板|
|硬件要求|8×H100|2×A100 ➕ RTX 4090|

单次成本更低、搜索空间更聚焦、硬件门槛更低——**实验循环天然适配微调场景**。

**1. 数据质量是第一杠杆，且可有限度地纳入实验循环。** 原作者几百次超参改动中，效果最大的单一变量是换数据集（FineWeb-edu → ClimbMix）。微调场景下数据量更少（几万~几十万条），每条数据的质量权重更高，数据筛选、清洗、去重的投入产出比远高于调超参。“高质量数据”本质上是相对概念，只能通过训练结果间接度量——这恰好与 autoresearch「让结果反向定义质量」的逻辑吻合。团队已有多源数据，Agent 可测试不同来源的配比或按聚合物类别筛选子集，量化各部分的边际贡献。但数据变更的搜索空间远大于超参且缺乏连续性，每次需完整重训练。务实做法：**超参全自动高频迭代，数据实验低频穿插、人工设定候选方案后交 Agent 验证**。

**2. 单一指标不够，评估体系需要分层。** 原版用单一 `val_bpb` 做二元决策即可，但微调场景中 eval_loss 的下降不等于业务效果的提升。单一指标作为优化目标容易过拟合（Goodhart 效应），需要分层兜底：

|层级|指标|角色|
|---|---|---|
|L1 粗筛|eval_loss|全自动，每轮必跑，快速淘汰明显劣化实验|
|L2 精筛|任务指标（F1、BLEU 等）|全自动，需标注测试集，衡量真实任务能力|
|L3 终筛|LLM-as-Judge / 人工抽检|周期性执行，捕捉自动指标遗漏的质量问题|

实验循环内以 L1+L2 组合做自动决策（eval_loss 下降且任务指标不退化方可 KEEP），L3 每 N 轮人工校准。**测试集的质量决定整个评估体系的上限**。

**3. Agent 可探索人工预设之外的优化路径。** 微调的可调维度已被 LLaMA-Factory 等框架标准化，人工通常在框架定义的参数范围内搜索。但一个真实案例表明 Agent 的能力不止于此：有开发者在单张 GTX 4090 上用 autoresearch 优化 AI 预测模型，前期在预设搜索空间内提升了 24% 后陷入停滞；随后通过 `/btw` 指令告诉 Agent「不要约束在当前思路，可以从底层修改」，Agent 自主重构了模型架构，48 小时约 70 次实验后将 MSE 降低了 240 倍。这说明当人工划定的搜索空间本身就是瓶颈时，放开约束让 Agent 自由探索可能带来超出预期的收益。在提示词优化、数据采集效率、数据工程等方向上，同样值得在框架参数搜索之外，给 Agent 留出从底层重新思考的空间。

**4. 自动化实验循环是通用方法论。** Autoresearch 的核心是**「假设→实验→反馈→迭代」的自动化闭环**，不限于模型训练。团队的数据采集、知识提取、知识图谱、预测模型等每条线都存在可量化的优化目标。只要具备**明确的评估指标**和**可控的单次实验成本**，即可套用此范式。

基于以上分析，我们决定将 autoresearch 的自动化实验思路迁移到 Qwen 微调场景。原版 autoresearch 是单 Agent 单文件的简单循环；为了在多 GPU、多服务器环境下并行探索，我们引入 **ClawTeam**——Claude Code 的多 Agent 协作框架——来实现 Leader-Worker 调度模式。以下为本地演练和服务器部署方案。

---

## ClawTeam 本地演练

> 目标：在 Mac 上用小模型跑通 ClawTeam 多 Agent 协作 + 微调实验循环的完整流程，验证可行性后再迁移到服务器。

### 演练策略

受限于 macOS 硬件，演练使用 Qwen2.5-0.5B（500M 参数）替代正式模型，训练框架使用 transformers + peft（LLaMA-Factory 对 MPS 支持不完整），2 个 Worker 串行共享 MPS 设备，数据为 100 条模拟样本。

演练的核心目的不是追求训练效果，而是**验证 ClawTeam 多 Agent 协作 + 微调实验循环的完整流程**。上述选择均可在迁移到服务器时按实际环境替换，核心逻辑（实验循环、ClawTeam 调度、Agent 间协作）是通用的。

### 项目结构

 autoresearch-finetune-multi-agent/  
 ├── program.md              # Agent 指令文件（定义实验流程与搜索空间）  
 ├── train.py                # 训练脚本（macOS MPS 适配，Agent 通过配置文件控制参数）  
 ├── evaluate.py             # 评估脚本（Agent 不修改，输出 eval_loss）  
 ├── results_lr.tsv          # worker-lr 的实验记录  
 ├── results_lora.tsv        # worker-lora 的实验记录  
 ├── best_config.yaml        # 当前最优配置  
 ├── configs/                # 每次实验的配置存档  
 │   ├── exp_lr_001.yaml  
 │   ├── exp_lora_001.yaml  
 │   └── ...  
 ├── data/  
 │   ├── train/train.json    # 训练数据（Alpaca 格式）  
 │   └── eval/eval.json      # 评估数据（固定，不可修改）  
 └── outputs/                # 训练产出的 LoRA adapter

### 多 Agent 架构

本地演练采用 Leader + 2 Worker 的三 Agent 架构：

 Leader（调度者，不占 GPU）  
 ├── 每 10 分钟读取 results_lr.tsv 和 results_lora.tsv  
 ├── 汇总分析，识别最优配置  
 ├── 通过 clawteam inbox 向 worker 发送下一轮方向  
 └── 协调 GPU 使用：worker-lr 先跑，完成后通知 worker-lora  
 ​  
 Worker-lr（探索学习率）  
 ├── 固定 LoRA 配置（rank=16, alpha=32, target=q_proj,v_proj）  
 ├── 搜索空间：lr ∈ {1e-5, 2e-5, 5e-5, 1e-4, 2e-4}  
 ├── 结果写入 results_lr.tsv  
 └── 每批实验完成后通过 inbox 通知 worker-lora  
 ​  
 Worker-lora（探索 LoRA 配置）  
 ├── 固定学习率（lr=2e-5）  
 ├── 搜索空间：rank ∈ {8, 16, 32, 64}，target modules 变体  
 ├── 结果写入 results_lora.tsv  
 └── 等待 worker-lr 的 done 信号后再启动训练（共享 MPS）

> **GPU 协调机制**：macOS 只有一个 MPS 设备，两个 worker 不能同时训练。通过 ClawTeam 的 inbox 消息实现串行协调——worker-lr 完成一批实验后发送 `done` 信号，worker-lora 收到后才启动。上服务器后各 worker 独占 GPU，无需此约束。

### 启动流程

 cd ~/autoresearch-finetune-multi-agent  
 ​  
 # 1. 创建团队  
 clawteam team spawn-team qwen-multi \  
   -d "Qwen2.5-0.5B LoRA fine-tuning on macOS" -n leader  
 ​  
 # 2. 启动 Leader（Leader 自动 spawn 两个 worker）  
 clawteam spawn tmux claude \  
   --team qwen-multi --agent-name leader \  
   --task "Read program.md. You are the LEADER agent. You do NOT run experiments yourself.  
           1) Spawn worker-lr and worker-lora  
           2) Coordinate GPU usage: worker-lr runs first, then worker-lora  
           3) Every 10 minutes, read results files and send new directions"  
 ​  
 # 3. 监控  
 clawteam board attach qwen-multi   # tmux 平铺视图，实时查看所有 Agent

### 本地演练实况

以下为 macOS 上运行 ClawTeam 多 Agent 协作的终端截图。

**Leader Agent**——读取 `program.md`，管理两个 worker 的任务分配和 GPU 协调：

![Leader Agent](https://p.ipic.vip/a5tojh.jpg)

**Worker-lr**——负责学习率搜索，固定 LoRA 配置，依次测试 lr ∈ {1e-5, 5e-5, 1e-4, 2e-4}：

![Worker-lr](https://p.ipic.vip/07z4kr.jpg)

**Worker-lora**——负责 LoRA 配置搜索，等待 worker-lr 的 done 信号后启动，依次测试 rank ∈ {8, 32, 64} 和不同 target modules：

![Worker-lora](https://p.ipic.vip/9qes70.jpg)

![worker-lora2](https://p.ipic.vip/84u9n2.jpg)

从截图可以看出三个关键行为：

1. **Leader 正确完成了调度**——spawn 了两个 worker，并通过 inbox 协调 GPU 使用顺序
    
2. **Worker 间通信正常**——worker-lr 通过 `clawteam inbox send` 向 worker-lora 发送 done 信号，worker-lora 通过 `clawteam inbox receive` 轮询等待
    
3. **ClawTeam 的 task 系统正常工作**——每个 worker 有独立的 task ID，通过 `clawteam task update` 报告进度
    

### 演练验证清单

|验证项|标准|
|---|---|
|Agent 自主实验|连续多轮无报错，results.tsv 持续增长|
|Leader 调度|能 spawn worker、读取结果、发送方向指令|
|Worker 间协调|inbox 消息传递正常，GPU 不冲突|
|实验有效性|eval_loss 有下降趋势|
|配置存档|configs/ 下每次实验有对应 yaml|

结果如下： ![multi-result](https://p.ipic.vip/nc6rjq.jpg)

---

## 服务器部署方案（待完善）

> 本地演练验证了 ClawTeam 多 Agent 协作 + 微调实验循环的可行性。以下为迁移到服务器后的部署方案。

### Phase 1：单机验证

目标模型为 **Qwen3.5-35B-A3B**（MoE 架构，总参数 35B，单次推理激活约 3B），采用 **4-bit QLoRA** 微调。

显存预算：35B 参数 4-bit 量化后约 17.5 GB，单卡剩余 ~6.5 GB 给 LoRA 适配器、梯度和激活值。

> **风险说明**：Unsloth 官方不推荐对 Qwen3.5 系列做 QLoRA 4-bit 训练，理由是 BitsAndBytes 对该架构的量化误差偏大。此方案属于激进路线，选择它是因为 35B MoE 模型在任务能力上的潜在优势值得一试。如果训练不稳定或效果不及预期，回退方案为 Qwen3.5-9B bf16 LoRA（单卡约 18 GB，官方推荐路线）。