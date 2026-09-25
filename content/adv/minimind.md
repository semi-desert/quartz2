---
dg-publish: true
title: MiniMind（从零训练小语言模型）
tags:
  - research
  - llm
  - todo
created: "2026-09-25"
updated: "2026-09-25"
---

上级：[[my_research|My Research]]

状态：**想动手**（2026-09-25 在视频号「AI纪年」看到，记下来以后自己跑一遍）

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

# 我想怎么试（草案）

1. 先读代码：模型结构（`model/`）→ Tokenizer → 训练脚本，搞懂每一层在做什么。
2. 在 AutoDL 租一张 3090 / 4090，按 README 跑 mini 数据集：Pretrain → SFT，先得到一个能对话的模型。
3. 用 `web_demo.py` 或 OpenAI 兼容接口聊一聊，看看 64M 模型能做到什么程度。
4. 有余力再试 LoRA、DPO / GRPO，或者换成自己的语料（比如殆知阁古籍）做个小模型。

# 我的理解

- （待补充）

# 待查

- [ ] 全流程（Pretrain + SFT）实际需要多少时间和显存；本机显卡能不能跑
- [ ] mini 数据集的大小和下载方式（HuggingFace / ModelScope）
- [ ] 与 nanoGPT（Karpathy）的区别与取舍
