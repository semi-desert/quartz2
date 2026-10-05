---
dg-publish: true
title: To Try
tags:
  - research
  - project
created: "2026-10-05"
updated: "2026-10-05"
---

上级：[[my_research|My Research]]　相关：[[os_mcu|OS - MCU]]、[[ai_learning_method|AI 辅助学习方法]]

看到的、想动手试的项目。每条写清：是什么、和我手上的事有什么关系、成本、风险。

# 想实践

| 项目 | 类别 | 和我的关系 | 成本 | 状态 |
|---|---|---|---|---|
| ESP32-C3 广告拦截器 | 硬件 / homelab | 手上有 ESP32，可做主 DNS 的备份 | 约 5 美元 | 想试 |
| image-blaster | AI / 3D | DFCine 3D 预演可参考它的流程设计 | 每个场景约 2—5 美元 API 费 | 想研究 |
| Opus 5.5 视频提示词库 | AI / 视频 | DFCine 视频、讲义动画可借用提示词 | Opus 5.5 需 Pro 以上 | 想研究 |

# ESP32-C3 广告拦截器（esp32-c3-adblock）

来源：视频号「算力炼丹炉」转 X @Psalteric，2026-10-05 收录。

- 是什么：埃及开发者 ZedAxis（GitHub 用户 M-Abozaid）做的 DNS 广告拦截器，跑在约 5 美元的 ESP32-C3 SuperMini 上。MIT 协议。
- 做法：插在路由器 USB 口取电，通过 Wi-Fi 联网；把它设成局域网 DNS，命中黑名单就拦，没命中就转发给上游 DNS。有网页控制台，可看拦截记录、加域名。
- 关键技巧：每个域名用 FNV-1a 算成 40 位哈希（5 字节），排序后写进 Flash，查询时二分查找，不占内存。理论上能放约 53.7 万个域名，约 50 KB 内存，拦截响应约 10 毫秒。
- 局限
	- 要留约 1.3 MB 给 OTA 升级，实际黑名单常用约 25 万条。
	- 没有 DHCP、没有完整查询历史，作者自己也是把它当 Pi-hole 的备份用。
	- 设备开了 DoH / DoT（加密 DNS）会绕过它。
	- 40 位哈希有极小概率误拦正常域名。
- 我这边要注意：我的 ESP32 MicroPython 板用 esptool / mpremote 会被弄死。这个项目是另刷固件，最好用一块新的 C3 板，不碰现有板子。
- 地址：https://github.com/M-Abozaid/esp32-c3-adblock

# image-blaster：一张图生成可探索的 3D 世界

来源：X @liambraus（西班牙语推文，Grok 翻译），2026-10-05 收录。推文说「整个行业失去了意义」，属于夸张。

- 是什么：一套给 Claude Code 用的 skill（MIT），不是新模型。把图片放进 `input/`，对 Claude 说「blast it」，它按步骤调别家模型：
	- World Labs Marble 1.1：生成环境（高斯泼溅 .spz + 碰撞网格 + 全景图）；
	- 混元 3D（腾讯，经 FAL）：每个物体单独生成网格 .glb / .obj，默认 5 万面带 PBR；
	- nano-banana（或 gpt-image-2）：把物体从原图里抹掉，做空场景；
	- ElevenLabs：环境循环音 + 每个物体的碰撞音效。
- 产出能直接进 Unity、Unreal、Godot、Blender，也自带一个 React 查看器（Spark 渲染泼溅、Rapier 物理）。
- 成本：仓库免费，模型收费。有报道推算一个拆出 5 件物体的场景约 4—5 美元；需要 World Labs 和 FAL 两把付费 key。
- 时间线：仓库 2026-04 建、05 发布、05-15 后停更；10 月因推文再次走红。另有报道称 AMD 9 月宣布收购 World Labs（未核实），Marble API 后续不确定。
- 局限：单张图只有一个视角，看不见的一面靠模型猜；抹除物体后可能留黑洞；网格拓扑、UV 没为量产准备，适合灰盒、预演、概念。
- 对 DFCine 有用的设计（值得抄的是流程，不是模型）
	- 拆图规则：「一个人能拎起或推动的」才算独立物体，其余留在环境里；同样的物体只记一个加数量。
	- 列完清单先停下等人确认，确认后才调用付费接口。
	- 花钱的请求先把 request id 写进隐藏文件再轮询，中断后续等，不重复扣费。
	- 文件名即状态（`N-slug.ext`），扫目录就知道进度，不另维护状态表。
	- 给世界模型的文字描述里也要删掉已拿走的物体（文字空场景），否则会长回来。
- 地址：https://github.com/neilsonnn/image-blaster

# Opus 5.5 视频：301 条特效提示词开源

来源：视频号「算力炼丹炉」转 X @yihui_indie，2026-10-05 收录。

- 是什么：作者花约 3000 美元，用 Claude Opus 5.5 复刻了 301 个热门特效视频，开源全部提示词和工作流。
- 原理：Opus 5.5 不直接生成视频，而是写代码（HTML Canvas、SVG、Three.js、Remotion、HyperFrames）画出每一帧，再用无头浏览器渲染、ffmpeg 合成 MP4。改哪里就改一行代码重新渲染，不是重新抽卡。
- 适合：动效、科普讲解、产品发布片、数据可视化。不适合：真人实拍质感。
- 相关仓库
	- yihui-dev/awesome-opus5-5-videos：本条主角，按动效、讲解、3D、游戏分类
	- Li-Evan/awesome-opus-5.5-video-prompts：334 条提示词，逐字核对原帖，附中文译文和「导演工具箱」
	- heygen-com/hyperframes、remotion-dev/skills：让 Agent 用代码出视频的渲染器
- 用法：Claude Code 里选 Opus 5.5，装渲染器（`npx skills add heygen-com/hyperframes`，需 Node 22+、Chrome、ffmpeg），挑一条提示词换成自己的主题；先让它每个场景出一张静帧，确认后再渲染。
- 版权：提示词和作品归原作者，商用前要确认授权。
- 我可以用在：奥数讲义的动画演示（如勾股定理、鸡兔同笼）、DFCine 宣传片。

# 我的理解

- （待补充）

# 待查

- [ ] 淘宝 ESP32-C3 SuperMini 价格；路由器能否给局域网下发自定义 DNS
- [ ] World Labs 被收购后 Marble API 是否继续开放
- [ ] 挑 1 条 Opus 视频提示词，用 HyperFrames 做一个 15 秒奥数动画试试
