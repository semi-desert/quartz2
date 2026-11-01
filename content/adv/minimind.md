---
dg-publish: true
title: MiniMind（从零训练小语言模型）
tags:
  - research
  - llm
  - todo
created: "2026-09-25"
updated: "2026-11-01"
---

上级：[[my_research|My Research]]　相关：[[ai_learning_method|AI 辅助学习方法]]

状态：**已定计划**（2026-11-01）。2026-09-25 在视频号「AI纪年」看到；按零基础起步，用本机 RTX 2080 16GB 训练，学习计划见下文。

# 是什么

- 仓库：<https://github.com/jingyaogong/minimind>（Apache-2.0，约 6.25 万星，2026-09-22 仍在更新）
- 用原生 PyTorch 从零训练一个超小语言模型，**不是调用大模型，也不是只做 LoRA 微调**，而是把造大模型的整套流程从头到尾用可读代码走一遍。
- 作者口径：约 3 块钱 GPU 租用成本 + 2 小时。「2 小时」指 **SFT 阶段在单张 3090 上跑完 1 个 epoch**，不是全流程。
- 孪生项目：[MiniMind-V](https://github.com/jingyaogong/minimind-v)（能看图的视觉版）、MiniMind-O（多模态 Omni）。

# 覆盖的链路

| 阶段 | 内容 |
|---|---|
| 分词器 | Tokenizer 训练代码（BPE + ByteLevel），带 `<tool_call>`、`<think>` 等标记 |
| 模型结构 | Dense + MoE，主线对齐 Qwen3 / Qwen3-MoE |
| 预训练 | Pretrain，数据集已开源 |
| 微调 | 全量 SFT、LoRA（不依赖 peft，从零实现） |
| 对齐 / 强化 | DPO、PPO / GRPO / CISPO、Agentic RL（多轮 Tool Use） |
| 其他 | 模型蒸馏、自适应思考、OpenAI 兼容 API 服务端、Streamlit 聊天 WebUI |

# 版本与参数量

| 版本 | 参数量 | 发布 |
|---|---|---|
| minimind-3 | 64M | 2026-04-01 |
| minimind-3-moe | 198M（激活 64M） | 2026-04-01 |
| minimind2 / minimind2-moe | 104M / 145M | 2025-04-26 |
| minimind-v1-small | 26M | 2024-08-28（已下线） |

注：视频里说的「25.8M、GPT-3 的 1/2700」对应旧版 v1-small，当前主线是 64M。

# 本机能训多大（2026-11-01 分析）

硬件：RTX 2080 16GB（Turing，算力 7.5）、Ryzen 9 5900X、128GB 内存；C 盘是 SSD，D 盘是 4TB 机械盘。

| 规模 | 按 20×参数量的 token 训饱，本机耗时（估算，未验证） | 定位 |
|---|---|---|
| 26M | 约 2 小时 | 入门实验 |
| 64M（MiniMind-3） | 约 11 小时 | 主线，跑通全流程 |
| 约 200M | 5–7 天 | 毕业作品，挂机一周 |
| 561M（nanochat d20） | 约 5 周 | 本机不做 |

- 显存能装下约 0.8B（混合精度 AdamW 每参数 16–18 字节），但真正的限制是算力：一周约 6e18 FLOPs，按 Chinchilla 配比，最优规模约 220M。
- 第二个上限是数据：`pretrain_t2t.jsonl`（10GB）估计约 2B token（未验证），只够训饱约 100M 的模型。所以毕业作品要过 2 遍数据，或者混入自己的语料。
- Turing 不支持 bf16 和 FlashAttention 2。训练脚本必须加 `--dtype float16`（默认是 bfloat16），用 GradScaler。PyTorch cu130 版本仍支持 sm_75。
- 不需要上云。云预算（不超过 200 元）只留作可选项，提交前先报预估。

# 暂停与续训（读源码确认，2026-11-01）

MiniMind 原生支持断点续训，不用自己从零写：

- 每 `--save_interval` 步（默认 1000）保存一次 `checkpoints/<名>_<维度>_resume.pth`，内容包括模型、优化器、GradScaler、epoch、step。写入方式是先写 `.tmp`，再 `os.replace`，中途断电也不会写坏旧档。
- 续训时加 `--from_resume 1`。`SkipBatchSampler` 用同一个随机种子重新排数据，跳过已训练的批次，数据顺序和中断前一致。
- 缺点：直接 Ctrl+C 会丢掉上次存档之后的进度，最多 1000 步。改进办法是加一个「停止文件」检查：检测到文件就立即存档并退出（打算做的小补丁）。
- 存档 64M 模型每次约 1GB，200M 约 3GB。数据集、存档和 HF datasets 缓存（`HF_DATASETS_CACHE`）都放 D 盘，不写 SSD。

# 学习阶梯（零基础起步）

| 级 | 掌握什么 | 过关标准 |
|---|---|---|
| L1 | 导数、梯度下降、反向传播 | 从空文件写出 micrograd |
| L2 | 语言模型 = 预测下一个 token；交叉熵、embedding | 写出 bigram 和 MLP 语言模型 |
| L3 | 注意力、Transformer 块、分词器 | 手写一个小 GPT，在本机训出能续写的模型 |
| L4 | 完整链路：预训练 → SFT → DPO / GRPO → 蒸馏 | 在 MiniMind 上每步都跑过，能讲清每步的损失函数 |
| L5 | 规模定律、算力预算、自己的语料和模型 | 独立完成毕业作品 |

起点：L0（学过，但已经忘光）。

# 学习计划（每次 2 小时，共 28 次，约 56 小时，加上 GPU 挂机时间）

每次结束后做 1 道「考官」题（一次一题，由易到难），没过的写进「待查」。

**A. 原理地基（10 次，L0 → L3，只用小模型，GPU 几乎不用）**

1. 3B1B 第 1–2 章：神经网络、梯度下降。练习：用 numpy 手写线性回归的梯度下降。复盘：学习率太大会发生什么？
2. 3B1B 第 3–4 章 + micrograd 前半：`Value` 类和链式法则。练习：手算一个两层表达式的梯度，再用代码对答案。
3. micrograd 后半：训练一个小 MLP。练习：不看视频，从空文件重写 micrograd。
4. PyTorch 张量基础（Raschka 附录 A）+ makemore 1：bigram。复盘：交叉熵和负对数似然是什么关系？
5. makemore 2：MLP 和 embedding。练习：改 embedding 维度，看 loss 怎么变。
6. makemore 3：激活值、梯度、初始化、归一化。复盘：为什么需要残差和 LayerNorm？
7. 3B1B 第 5–6 章 + Let's build GPT 前半：手写自注意力。
8. Let's build GPT 后半：多头、FFN、残差。练习：在本机用中文语料训一个小 GPT，10 分钟。
9. 分词器：Karpathy tokenizer 一讲 + minbpe。练习：用花园笔记训一个 BPE，对比词表大小和压缩率。
10. 总复盘：对照 MiniMind 的 `model/model_minimind.py`，标出它和课堂 GPT 的差异（RMSNorm、RoPE、SwiGLU、GQA）。费曼：讲清一个 token 从输入到输出 logits 经过了什么。

**B. 本机环境和续训演练（2 次）**

11. 把 MiniMind 克隆到 D 盘，建 venv，装 torch cu130，下载 mini 数据集。用 `--dtype float16` 跑 64M 预训练 10 分钟，记下 tokens/s 和显存。
12. 打「停止文件」补丁，并把训练注册到 ai-starter 面板（启动按钮固定带 `--from_resume 1`）。演练三次中断和续训，确认 loss 曲线能接上。用实测速度校准上面的耗时表。

**C. 跑通 64M 全流程（5 次，GPU 共约 6 小时，可以分几次挂机）**

13. 预训练：`pretrain_t2t_mini`（1.2GB）。看 loss 曲线，读 `train_pretrain.py`。
14. SFT：`sft_t2t_mini`。读懂 chat template，以及为什么只对 assistant 段计算损失（`generate_labels`）。
15. 评估和聊天：`eval_llm.py`、`web_demo.py`。体会 64M 能做到什么、做不到什么。
16. LoRA：读懂 `model_lora.py` 的低秩分解，用一个小领域数据（如古诗）微调。
17. 规模定律实验：训 5 个 5M–50M 的模型，每个 20–60 分钟，画 loss 和算力的关系，拟合幂律。

**D. 对齐与强化学习（4 次）**

18. DPO：推导损失函数，用 `dpo.jsonl` 训练，对比 SFT 前后的回答。
19. 奖励模型和 PPO：理解 actor、critic、reference 三个网络。显存是单网络的 1.5–2 倍。
20. GRPO / RLVR：组内相对优势、可验证奖励（数学题），以及 `<think>` 思考模式。
21. 复盘：RLHF、DPO、GRPO 的异同。费曼：讲清 RL 为什么能让模型学会「思考」。

**E. 蒸馏（3 次）**

22. 黑盒蒸馏：本机用开源的 Qwen3-1.7B 生成几千条带 `<think>` 的中文问答，用它们 SFT 一个 64M 学生。不用闭源模型的输出（服务条款禁止）。
23. 白盒蒸馏：`train_distillation.py`，损失 = α·CE + (1−α)·T²·KL。代码只截断 logits，不做词表映射，所以**老师必须和学生用同一个 6400 词表**：用 MiniMind-3-MoE（198M）官方权重或自己训的大模型当老师，不能直接用 Qwen。
24. 对比实验：只用 CE，和 CE 加 KL，比较两者的 loss 和回答质量。复盘：老师和学生差距太大时，蒸馏为什么会变差？

**F. 推理与部署（2 次）**

25. KV cache、温度和 top-p 采样、量化。
26. `serve_openai_api.py`：把自己的模型接成 OpenAI 兼容接口。

**G. 毕业作品（2 次 + 挂机约一周）**

27. 设计约 200M 的 dense 模型（如 hidden 1024 × 16 层，参数量以脚本输出为准）。用 `pretrain_t2t`（10GB）过 2 遍，或混入自己的语料（如殆知阁古籍）。用实测速度算总时长；超过 7 天就降到约 150M，或只过 1 遍。
28. 晚上和不用电脑时挂机，随时暂停、续训。训完做 SFT，再和 64M 版本对比。

# 我的理解

- （待补充）

# 待查

- [x] 全流程（Pretrain + SFT）实际需要多少时间和显存；本机显卡能不能跑 → 能跑，见上文（时长待第 11 次实测校准）
- [x] mini 数据集的大小 → pretrain mini 1.2GB、sft mini 1.6GB；完整版 10GB / 14GB
- [ ] mini 数据集的下载方式（HuggingFace / ModelScope），国内哪个快
- [x] 与 nanoGPT（Karpathy）的区别与取舍 → nanoGPT / Zero to Hero 用来学原理（阶段 A），MiniMind 用来跑全链路（阶段 C–G），nanochat 作规模参考
- [ ] MiniMind 的注意力实现在 fp16 下会不会溢出，是否走 PyTorch SDPA（第 11 次读代码确认）
- [ ] `pretrain_t2t.jsonl` 实际 token 数（用分词器统计）

# 资料

学习资源（每个都核实过存在）：

1. 3Blue1Brown 深度学习系列（第 1–7 章，B 站有官方双语版，例如第 6 章注意力 <https://www.bilibili.com/video/BV1TZ421j7Ke/>）：建立直觉，零基础先看。
2. Karpathy《Neural Networks: Zero to Hero》，中英字幕合集 <https://bilibili.com/video/BV1mqrTBvEaf>，代码 <https://github.com/karpathy/nn-zero-to-hero>：阶段 A 的主线，边看边写代码。
3. 塞巴斯蒂安·拉施卡《从零构建大模型》，人民邮电出版社，ISBN 978-7-115-66600-0，代码 <https://github.com/rasbt/LLMs-from-scratch>：配合视频查漏，附录 A 讲 PyTorch，附录 E 讲 LoRA。
4. Karpathy microgpt（2026-02，用 200 行纯 Python 训练 GPT）<https://karpathy.github.io/>：阶段 A 结束时自测。
5. MiniMind <https://github.com/jingyaogong/minimind>：阶段 B–G 的主线。

参考：

- nanochat d20：561M 参数、11.2B token，8×H100 跑约 4 小时、约 100 美元（<https://github.com/karpathy/nanochat>）。
- PyTorch CUDA 支持范围：cu130 覆盖 Turing (7.5) 到 Blackwell（dev-discuss.pytorch.org，2026-04 和 2026-09 的公告）。
- MiniMind 源码：`trainer/train_pretrain.py`、`trainer/trainer_utils.py`（`lm_checkpoint`、`SkipBatchSampler`）、`trainer/train_distillation.py`（2026-11-01 读的 master 分支）。
