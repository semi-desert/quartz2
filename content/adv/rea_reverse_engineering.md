---
dg-publish: true
title: REA - Reverse Engineer Anything
tags:
  - research
  - ai
  - reverse-engineering
  - mcp
created: "2026-10-16"
updated: "2026-10-16"
---

上级：[[my_research|My Research]]　相关：[[agent_principles|Agent 与 AI 原理]]（MCP 部分）

# 研究问题

- 2026-10 爆火的 REA 项目（宣传为“AI 逆向解析任意程序”）到底是什么，原理是什么？
- 它是新的逆向技术，还是现有工具的封装？
- 能否用在自己的项目上（Windows 主力机、Electron 项目 DFCine）？

# 要点

## 基本信息（2026-10-16 读源码时）

| 项 | 值 |
|---|---|
| 仓库 | [morluto/rea](https://github.com/morluto/rea)，MIT |
| npm 包 | `rea-agents`，v6.5.0（提交 `42fc6c5`） |
| 语言 | TypeScript（Node.js ≥22.19 / ≥24.11 / ≥26） |
| 规模 | 非测试 TS 约 32 万行，测试文件约 1400 个，另有 stryker 变异测试 |
| 形态 | MCP stdio server + CLI，两者共用同一套流程 |
| 安装 | `npx rea-agents setup`（预览改动 → 确认 → 写入智能体配置并备份） |

## 原理：它是 MCP 适配层，不是新逆向技术

REA 自己不做反编译。它把现成的逆向引擎（Ghidra、IDA、Hopper、JADX、浏览器 CDP 等）包成一套统一的 MCP 工具。推理与解释仍由 AI 智能体完成。

```
AI 智能体（推理）
   │  MCP stdio（JSON-RPC）
REA 服务（工具注册 → 会话路由 → 证据账本）
   │  桥接脚本 + socket / 子进程
Ghidra/IDA · JS/Electron AST · .NET/APK · 浏览器 CDP
   │
目标程序（本地文件，不上传）
```

1. **入口**：`scripts/rea.mjs` 分两条路。`src/main.ts` 作为 MCP stdio 服务运行；`src/cli.ts` 作为一次性命令运行。两者共用 `src/application/`。
2. **引擎桥接**：在引擎内部运行一段桥接脚本，通过 socket 逐行收发 JSON。
	- Ghidra：`bridge/ghidra/ReaGhidraBridge.java`（4892 行），以 HeadlessScript 运行，打开 Unix socket 或 TCP 端口。
	- Hopper：`bridge/hopper_bridge.py`，在 Hopper 自带的 Python 环境里运行。
3. **会话路由**：`SessionProviderRouter` 把每个目标固定绑定到一个引擎；引擎不可用时直接报错，不悄悄换引擎。
4. **证据账本**（最有价值的设计）：`src/application/investigation/`。每条结论带出处（函数偏移、汇编、调用关系），并把“还没弄清楚的部分”单列为未知项，减少 AI 编造。
5. **调查流程**：附带的 skill `reverse-engineer-anything/SKILL.md`（355 行）规定“反编译 → 跨模块追踪 → 用自己的技术栈重写”三步，安装时写进 AI 客户端。

## 能分析什么

| 目标 | 依赖 |
|---|---|
| 原生二进制（伪代码、汇编、字符串、调用） | Ghidra / IDA / Hopper 三选一 |
| JavaScript / Electron（模块、IPC、preload、source map） | 只要 Node.js |
| .NET 程序集（元数据、CIL） | 无 |
| Android APK | 无界面 JADX + 完整 JDK |
| 网页 | Chrome 系浏览器 |
| WASM、SQLite、EVM 字节码、抓包、固件、逻辑分析仪录制等 | 多数仅 Linux 验证 |

官方案例：DX-Ball 声像计算还原（通过 3205 个原始 x86 用例）、Notion 剪贴板 IPC 链路追踪、东方 TH04 弹幕角度计算还原。

## 评估

优点：

- 工程投入大，测试严格。
- 覆盖目标类型多，一套接口打通。
- 全部本地运行；但工具返回的反编译结果会发给 AI 的模型服务商。
- 证据 + 未知项的输出契约，适合核查。

限制：

- **Windows 原生分析是实验性的**：只支持 Ghidra 12.1.x + JDK 21 分析 PE（见 `docs/windows-ghidra-p0.md`）。进程录制、固件、抓包、崩溃分析等多为 Linux/macOS。
- **接口变化快**：已到 6.x，并有 6.2 迁移指南。
- **工具数量多**：源码里粗数约 147 个工具名（待核实），全部注册会占较多上下文 token。
- **法律风险**：仅限分析自己拥有或获授权的软件，见仓库 `DISCLAIMER.md`。商业工具（IDA、Hopper、JEB）须合法授权。

宣传里的“软件时代到头了”是营销说法。项目文档自己写明：不还原原始源码，不整体克隆应用。

## 与自己项目的关系

- 最直接的用途：静态分析 Electron 应用，无需逆向引擎。可先拿 DFCine 自己打包的 asar 试，看它能还原多少模块、IPC、preload 关系：
	```bash
	npx -y rea-agents@latest analyze-javascript-application "/absolute/path/to/app.asar" --json
	```
- 分析 exe / dll：需自行装 Ghidra 12.1.x + JDK 21（免管理员，体积较大）。
- `npx rea-agents setup` 会改智能体配置，先用 `--dry-run --json` 看改动计划。

# 我的理解



# 待查

- [ ] 对 DFCine 打包产物跑一次 `analyze-javascript-application`，评估还原质量
- [ ] 实际注册后的工具数量与 token 占用（待核实）
- [ ] Windows 上 Ghidra 12.1.x + JDK 21 免管理员安装是否顺利
- [ ] 证据账本 / 未知项的数据结构，是否值得借鉴到自己的 Agent 工具设计

# 资料

- 仓库：<https://github.com/morluto/rea>
- 官网与案例：<https://rea.tools/>
- 中文介绍：[博客园 - 逆向工程(REA)](https://www.cnblogs.com/tzhuwb/p/23291713)
- 本地参考克隆：`C:/src/_ref/rea`（浅克隆，`42fc6c5`）
