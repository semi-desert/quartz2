---
dg-publish: true
title: Agent and AI Principles
tags:
  - research
  - ai
  - learning
created: "2026-10-05"
updated: "2026-10-05"
---

上级：[[my_research|My Research]]　相关：[[ai_learning_method|AI 辅助学习方法]]、[[minimind|MiniMind]]

# 研究问题

- 大模型（LLM）本质上在做什么？为什么「预测下一个词」能做出推理和写代码？
- 一个只会「收请求、回文字」的 API，怎样变成能联网、改代码、调 MCP 的 Agent？
- Agent 做事的边界在哪：什么时候会出错、为什么会幻觉、为什么很费 token？

# 要点

## 学习阶梯（5 级）

| 级 | 掌握什么 | 常见错误 | 练习 | 过关标准 |
|---|---|---|---|---|
| 1 会用 | 知道模型、提示词、上下文、token 是什么；会写清楚的提示词 | 以为模型「记得」上次对话；以为它在查数据库 | 同一问题换 3 种问法，比较答案 | 能说清「API 无状态」是什么意思 |
| 2 懂模型 | 预训练 → 微调 → 强化学习三阶段；分词；Transformer 和注意力的直觉；幻觉的来源 | 把模型当搜索引擎；以为它逐字理解 | 用 Tiktokenizer 看中文怎么被切成 token | 能用 3 句话给别人讲清 LLM 怎么训练出来 |
| 3 懂 Agent | 工具调用循环（tool_use → 执行 → tool_result → 再请求）；系统提示、技能、记忆、上下文压缩 | 以为工具在模型里面执行 | 读一次 WorkBuddy 的工具调用过程，逐步标出谁在执行 | 能画出 Agent 循环图，并标出每种能力由谁提供 |
| 4 能动手 | 用 API 写最小 Agent；写 MCP 服务器；写 Skill | 工具描述写得含糊，模型选错工具 | 50 行 Python Agent + 一个 MCP 工具 | 自己写的 Agent 能连续调 2 个工具完成一个任务 |
| 5 能做项目 | 工作流 vs Agent 的取舍；评估、成本、权限与安全；多 Agent | 什么都上 Agent，成本失控；不设权限边界 | 给 ai-starter 或 DFCine 做一个真实的 Agent 功能 | 项目跑通，并能说出它的成本、失败模式和防护措施 |

## 20 小时计划（10 次 × 2 小时）

| 次 | 目标 | 练习 | 复盘问题 |
|---|---|---|---|
| 1 | LLM 是什么：下一个 token 预测 | 看 Karpathy 视频 0:00—0:42；用 Tiktokenizer 切一段中文 | 为什么模型数不清单词里有几个字母？ |
| 2 | 训练三阶段 | 看 Karpathy 视频 0:59—1:20、2:07—2:27 | 预训练和微调分别给了模型什么？ |
| 3 | Transformer 和注意力的直觉 | 看 3Blue1Brown 第 5、6 章 | 注意力在一句话里「看」的是什么？ |
| 4 | 幻觉、工具、工作记忆 | 看 Karpathy 视频 1:20—1:47 | 为什么把资料放进上下文能减少幻觉？ |
| 5 | Agent 循环 | 读 Claude 文档「Tool use」；对照 WorkBuddy 的一次真实调用 | stop_reason 为 tool_use 时，下一步谁做什么？ |
| 6 | 动手：最小 Agent | Python 写一个只有 read_file 工具的 Agent，约 50 行 | 去掉工具描述会发生什么？ |
| 7 | 工作流 vs Agent | 读 Anthropic《Building effective agents》 | 奥数讲义批量生成该用工作流还是 Agent？为什么？ |
| 8 | MCP | 用 Python SDK 写一个 MCP 服务器，接进 WorkBuddy | MCP 工具和内置工具对模型有区别吗？ |
| 9 | 技能、记忆、上下文管理 | 拆解 `~/.workbuddy/skills` 里一个 skill 的触发与加载 | 为什么技能只先放描述，用时才加载全文？ |
| 10 | 成本与安全 | 估算一次 20 步任务的 token；列出 Agent 的权限风险 | 为什么调用越多越贵？怎样防提示注入？ |

## 5 个资源与 7 天路径

| # | 资源 | 适合 | 难度 | 怎么用 | 可跳过 |
|---|---|---|---|---|---|
| 1 | Karpathy《Deep Dive into LLMs like ChatGPT》（YouTube，3 小时 31 分，有中文字幕搬运） | 零基础到中级 | ★★ | 按章节分段看，配合 Tiktokenizer | 0:31—0:59 GPT-2、Llama 推理演示；2:27 后 DeepSeek、AlphaGo 细节 |
| 2 | 3Blue1Brown 神经网络系列第 5—7 章（Transformer、注意力、模型如何存储事实） | 想要直觉的人 | ★★★ | 只看动画建立直觉，不推公式 | 第 1—4 章（反向传播）初次可跳 |
| 3 | Claude 文档「Tool use with Claude」（platform.claude.com） | 想懂 Agent 循环 | ★★ | 照着 get_weather 例子跑一遍 | 服务端工具、计费表 |
| 4 | Anthropic《Building effective agents》 | 想做项目 | ★★ | 重点读「工作流 vs Agent」和 5 种模式 | 附录 1 客服案例 |
| 5 | MCP Python SDK 文档（py.sdk.modelcontextprotocol.io） | 想写工具 | ★★ | 用快速入门的 `add` 例子起步 | 传输方式细节、v1 迁移指南 |

| 天 | 内容 | 资源 |
|---|---|---|
| 1 | 预训练、分词、神经网络输入输出 | 1 |
| 2 | 后训练、幻觉、工具、「模型需要 token 来思考」 | 1 |
| 3 | Transformer 与注意力直觉 | 2 |
| 4 | 强化学习、RLHF，回到 Karpathy 总结 | 1 |
| 5 | Agent 循环，写最小 Agent | 3 |
| 6 | 工作流和 Agent 的 5 种模式，用自己的项目对照 | 4 |
| 7 | 写一个 MCP 服务器接进 WorkBuddy | 5 |

## 已讲过的核心结论

- API 只做一件事：收请求、回内容；无状态，每次都要重发全部历史。
- 联网、改文件、跑命令、MCP、技能、记忆都由 Agent 外壳（WorkBuddy）执行；模型负责规划、选工具、填参数、生成内容。读图是模型原生能力。
- 工具调用循环：模型输出 tool_use → 外壳执行 → 结果作为 tool_result 加入历史 → 再请求 → 直到模型只输出文字。
- 技能按需加载：平时只放名字和描述，用时才读全文，省 token。
- MCP 是「工具清单」的通用接口标准：任何程序按协议实现服务器，所有支持 MCP 的 Agent 都能接入。

# 我的理解

- （待补充）

# 待查

- [x] 起点测定（2026-10-06）：第 1 级通过（8/10）；第 2 级未过（4/10），从第 2 级开始
- [ ] 薄弱点：基础模型只会「续写文档」，不会回答问题；变成助手靠后训练（对话数据微调 + 强化学习），不是靠 Agent 外壳
- [ ] 薄弱点：幻觉不是「选中第二概率的答案」，而是模型没有「不知道」的默认选项，按句式自信续写；参数里的知识是模糊记忆，上下文里的材料才可照抄
- [ ] 考官进度（2026-10-06 暂停）：第 1 题 8/10、第 2 题 4/10、第 3 题 8/10，第 2 级有条件通过；下次从第 4 题继续——「推送到 GitHub」中间几次 API 请求、各由谁做什么；git push 报错后谁决定下一步
- [ ] 第 6 次练习：最小 Agent 代码放哪个仓库（候选 `C:\src\ai-starter` 或新建）

# 资料

- Karpathy, Deep Dive into LLMs like ChatGPT：https://www.youtube.com/watch?v=7xTGNNLPyMI
- 3Blue1Brown, Neural networks：https://www.youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi
- Claude 文档 Tool use：https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
- Anthropic, Building effective agents：https://www.anthropic.com/engineering/building-effective-agents
- MCP Python SDK：https://py.sdk.modelcontextprotocol.io/
- Tiktokenizer：https://tiktokenizer.vercel.app/
- Transformer 3D 可视化：https://bbycroft.net/llm
