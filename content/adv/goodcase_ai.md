---
dg-publish: true
title: GoodCase.ai - AI 提示语案例复测站
tags:
  - research
  - ai
  - prompt-engineering
  - ai-video
created: "2026-11-19"
updated: "2026-11-19"
---

上级：[[my_research|My Research]]　相关：[[agent_principles|Agent 与 AI 原理]]（Agent Skill 部分）、[[ai_learning_method|AI 辅助学习方法]]、[[to_try|想实践清单]]

# 研究问题

- 2026-08 上线的 goodcase.ai 到底是什么，和已有的提示词合集站（PromptHero、Civitai、各类「提示词大全」）差别在哪？
- 它宣称的「用重跑验证提示语真伪」在方法上站得住吗，哪些是营销、哪些是真做了？
- 对本地的 AI 视频 / 图像工作流（ComfyUI、Seedance 类闭源 API）有没有可用的东西？

# 要点

## 基本信息（2026-11-19 核对）

| 项 | 值 |
|---|---|
| 站点 | <https://goodcase.ai>，中英双语，另有子站 dsh.goodcase.ai |
| 出品人 | 卡尔（@aiwarts，「卡尔的 AI 沃茨」），6 年大模型算法研发出身（字节 / 阿里 / 百度） |
| 开源仓 | [LearnPrompt/goodcase](https://github.com/LearnPrompt/goodcase)，MIT（代码）/ CC BY 4.0（策展元数据），2026-08-17 建仓，13 star |
| 主仓 | [LearnPrompt/awesome-seedance](https://github.com/LearnPrompt/awesome-seedance)，约 2.6k star，Seedance 视频口径 |
| 技术栈 | Next.js 站点 + Supabase；公开 API `/api/public/cases`，无需 Key |
| 规模（sitemap 实测） | 案例约 2270 条（作者自述「2000 多个」）、创作者 217 位、skill 页 118 个 |
| 复测（站内实测） | 1303 次复测、覆盖 1070 个案例 |
| 收入模式 | 站点本身免费无付费墙；产出是作者的内容流量入口（关联 AI HOT、carlwow.com） |

## 一句话定性

它本质上是一个**只收带完整提示语的 X 爆款案例库，用「换个模型重跑一遍」当质量信号**——把「这条 prompt 是真的还是截图」这个判断，从点赞数换成了可对照的实验记录，再把通过复测的案例提炼成可安装的 Agent Skill。

## 宣称核查

| 宣称 | 判定 | 依据 |
|---|---|---|
| 「怎么收集 Case、怎么复测、怎么算出榜单，整个过程都开源」 | ❌ 不实 | 首页原话。但 README 明写「线上站点的数据供给、审核后台与自动采集属于内部系统，不在本仓库里」；`src/` 只有 `app`/`components`/`generated`/`i18n`/`lib`/`proxy.ts`，是纯 Next.js 前端，无采集器、无审核后台、无打分管道 |
| 「112 个可安装 Skills」 | ⚠️ 口径问题 | `skills/generated` 下确有 112 个目录，但按 `-by-<hash>` 去重后只有 **33 个 base skill**，21 个带创作者变体（`pov-vlog-presence` 一家 14 个变体、`editorial-portrait-composition` 11 个） |
| 「第一个把视频提示词批量拿到第二个模型上重跑、成败都公开的库」 | ❓ 无法证伪 | 没找到更早的同类；这条既是最强卖点也最难核实，属自我宣称 |
| 「平台逆向重构的 prompt 一条都不收」 | ✅ 有制度 | [`/standards`](https://goodcase.ai/standards) 写明 2026-08-05 起对全部内容生效；公开 API 的 `provenance.method = human-reviewed-source-match` 字段可查 |
| 「月活破十万」 | ❓ 无法核实 | 作者 2026-11 文中自述，无第三方数据 |
| 案例「完整提示语」 | ✅ 已核实 | 抽检案例页给全文，变量位用 `{argument name=... default=...}` 标出，并附原帖核对链接 |
| 复测 1303 次 | ✅ 站点公开可数 | [/retests](https://goodcase.ai/retests) 页首屏统计 |

## 机制：三条管线

**① 收录闸门**（[`/standards`](https://goodcase.ai/standards)）。四要素缺一不可：作者、完整提示语、发布时间、可核验互动数据。热度用相对分位而非绝对值：新案例热度分必须 ≥ 同平台已发布案例的中位数，同平台样本不足 20 条转人工。证据分级 L1（作者/提示语/来源/媒体四项核实）/ L2（L1 + 一次独立复测），已发布案例不存在 L0。判重按「同链接 / 同提示语 / 同作者高度相似」三选一，但同一 prompt 被不同作者各自跑出作品算「复现豁免」——反而加分。

**② 复测**。把同一段提示语换一个模型重跑，跑好的和跑崩的都公开，评分取 AI 裁判打分均值。覆盖的核心问题正是「这条 prompt 是 prompt 强，还是那个模型强」。

**③ 提炼 Skill**。通过复测的案例按类型聚成 Skill。抽检 `editorial-portrait-composition/SKILL.md`，结构是：Required evidence gate（每次调用必须先读 `references/cases.md`、选且只选一个 anchor Case、命名它、不许平均多个风格）→ Preserve / Replace / Avoid 三段参考契约 → 工作流 → 输出契约 → 自检清单 → 署名与安全条款（不许声称被原作者授权、不许把接近复制说成原创）。

## 复测方法的真实边界（关键）

页面上的「跨模型复测」听着像模型矩阵，实际不是。陪审团分布（2026-11-19）：

| 复跑执行模型 | 次数 | 覆盖案例 | 最近 |
|---|---|---|---|
| Grok Imagine | 597 | 594 | 2026-11-14 |
| MiniMax H3 | 348 | 336 | 2026-09-07 |
| GPT（编程裁判） | 212 | 136 | 2026-11-14 |
| GPT Image 2 | 117 | 110 | 2026-11-17 |
| Claude | 29 | 28 | 2026-11-06 |

两个模型占 72%，且作者原文说明过选型逻辑：首轮从 720P 档位挑价格最接近的三个（Seedance 2.0 Fast 0.6 元/秒、MiniMax H3 0.5 元/秒、Wan 3.0 0.6 元/秒），选「表现最稳定且综合成本最划算」的那个。**所以复现率里混了两个变量：提示词的跨模型鲁棒性 + 那个便宜模型自身的能力上限。** 复现成功只证明「这条 prompt 不是编的」；复现失败不能反推提示词造假，可能只是 MiniMax 那段镜头拍不出来。作者自己在页面上的措辞是「我们鉴别的是提示语，不评价原作者」，分寸是对的，但读者容易把 73% 读成「三分之一是假的」。

第二处术语混淆：「陪审团」在站内同时指**复跑执行模型**（`/retests` §02 那张表）和**打分的 AI 裁判**（「分数是多个 AI 裁判各自打分的均值」）。稳定分是后者的均值，不是那 5 个模型各打一分。未见人类基准校准（未验证）。

其余硬限制：案例源以 X 为主，抖音/小红书/B站的中文创作者覆盖薄；被复测的模型全是闭源 API，**没有 Flux / Qwen-Image / SDXL 等本地开源模型的复测**；模型迭代快，提示语过期速度也快，作者自己把这列为焦虑源。

## 可迁移的提示词手法（正文里抽出来的）

这些是 2000 条案例蒸馏出来的结论，比「提示词大全」里的形容词堆叠有用：

- **真实感来自相机缺陷，不是来自 realistic**。指名具体器材年代（mini DV / 16mm / VHS / iPhone 16 Pro），把它的毛病写成要求：手抖、对焦来回找、曝光呼吸、构图漂移、变焦不匀、偶尔切掉半张脸。一个具体型号带整套光学特征。
- **冲突约束会互相抵消**。同一段 prompt 里既要「手持真实感」又要「4K 电影级打光」，最后落在塑料感上——现实中没有设备同时做到这两件事。写提示语像写需求，别让约束打架。
- **打斗写接触点，不写形容词**。「激烈搏斗」跑出来是对着空气打；「直拳打中下巴，格挡后反击体踢」才有效果。5–10 秒不超过 4 段动作，超过就糊。
- **指名流派能调动生物力学**。写「泰拳扫踢」和「跆拳道横踢」，模型给出的是大弧线 vs 髋部直线爆发两种不同的力量感。
- **大特写会暴露 AI 脸**。需要情绪对白时宁可拉到中景或半身；男友视角明确禁止脸部填满画面。
- **数字变焦不能当转场**。要一镜到底就补一句禁止数字变焦和隐形剪辑。
- **台词每句控制在八个词以内**，长了抢画面还拖垮口型。

# 我的理解



# 待查

- [ ] 稳定分的裁判是谁、几票、是否有任何人类基准（页面只写「多个 AI 裁判」，未见明细）
- [ ] 抽 2–3 条案例，用本地 ComfyUI（Flux / Qwen-Image）跑同一段 prompt，看结构是否跨得过去
- [ ] `editorial-portrait-composition` 的 Preserve / Replace / Avoid 契约，能否直接搬进自己的技能模板
- [ ] 作者上一层的「AI HOT」与 goodcase 的数据是否同一套采集管线（未验证）
- [ ] 站点内容是否会随模型下架而失效，有没有存档机制

# 资料

- 站点：<https://goodcase.ai/>｜收录标准：<https://goodcase.ai/standards>｜复测专题：<https://goodcase.ai/retests>
- 开源仓：[LearnPrompt/goodcase](https://github.com/LearnPrompt/goodcase)（站点 UI + 112 个 skill 目录 + 样例数据）
- 视频口径主仓：[LearnPrompt/awesome-seedance](https://github.com/LearnPrompt/awesome-seedance)
- 作者自述方法与成本：[这个封装了我3年实测经验的AI提示词案例网站](https://new.qq.com/rain/a/20260813A04T5U00)（十万积分横测、三模型同价档选型）
- 作者自述复测与 Skill 提炼：[我把463个AI视频做成了Skill和提示语模版](https://dy.163.com/article/L7E7BCFH055659FP.html)（两万抽卡费、四模型打分、蒸馏过程）
- 第三方评述：[Awesome Seedance 工作流页](https://vibecontent.net/workflows/awesome-seedance)（给出 cons：策展绑定作者自有站点、计数变化快、闭源模型提示语易过期）
