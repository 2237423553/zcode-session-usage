# zcode-session-usage

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-blue)](#compatibility)
[![Python](https://img.shields.io/badge/python-3.x%20stdlib%20only-green)](#compatibility)
[![ZCode](https://img.shields.io/badge/for-ZCode%20%F0%9F%A4%96-7c3aed)](https://zcode.z.ai)

**English** | [中文](#中文说明)

A [ZCode](https://zcode.z.ai) plugin that **automatically appends a live session-usage footer to every assistant reply** — the same style as [Cherry Studio](https://github.com/CherryHQ/cherry-studio)'s message stats:

```text
— 1718.6万 Tokens · 37.9 Token/秒
```

It also ships a `/usage` slash command that renders a full stats panel: input/output tokens, cache hits, model TPS, end-to-end throughput and a local cost estimate.

```text
## 📊 Token 使用 · sess_9ee05bb7…

GLM-5.3-Flash · account:zai-start-plan

| 输入 | 输出 | 模型生成 TPS |
|---|---|---|
| 4,915,016 Tokens | 26,867 Tokens | 38.8 Token/秒 |

费用（本地估算）：US$0.6127（after configuring prices.json）

| 明细 | 数值 |
|---|---|
| 模型调用 | 61 次（main_turn 61 次；重试 0 次） |
| ├ 缓存读取 | 3,822,656 Tokens |
| ├ 未缓存 | 136,623 Tokens |
| 推理 | ≈12,778 Tokens（estimated from chars） |
| 首 Token 耗时 | —（not recorded in logs） |
| 端到端吞吐 | 38.3 Token/秒 |
```

## Features

- **Automatic footer on every reply** — a `UserPromptSubmit` hook injects the live cumulative usage into the conversation; the model appends `— N万 Tokens · X.X Token/秒` to each reply. Zero manual steps.
- **`/usage` full panel** — input / output / cache read / cache write / uncached split, reasoning estimate, model generation TPS, end-to-end duration & throughput, per-model breakdown.
- **Local cost estimate** — put your model prices (US$/1M tokens) into `prices.json` and every panel shows a dollar figure.
- **100% local & offline** — reads ZCode's own model-I/O logs under `~/.zcode/cli/rollout/`. Nothing is uploaded, no API keys involved.
- **Stdlib-only Python** — one script, no dependencies.

## Install

1. In ZCode open **Plugin Marketplace → Add → Add Plugin Marketplace**
2. Paste: `2237423553/zcode-session-usage` (or the full `https://github.com/2237423553/zcode-session-usage.git` — any git host works)
3. In **Personal** find **Session Usage / 会话用量** and click **Install**
4. **Start a new task** (important: ZCode registers plugin hooks when a session starts) and send two messages — from the 2nd reply on, the footer appears automatically

> Want the full panel? Type `/usage`. Cost shows `—` until you add your model's prices to `session-usage/skills/session-usage/scripts/prices.json`.

## How it works

- On every user message the hook runs the stats script, reads the current session's model-I/O log (one JSONL line per model call: tokens, cache, duration) and injects the cumulative numbers; the model then appends the footer (numbers cover everything **before** that reply starts).
- ZCode trims model-I/O logs by default (`modelIoFullRetentionEnabled`), so "cumulative" means the retained window.
- Reasoning tokens are estimated from text length (CJK ≈ 1 token/char, other ≈ 0.25 token/char); per-token TTFT is not in the logs, hence the `—`.

## Compatibility

- Needs `python` (3.x, stdlib only) on PATH
- Windows / macOS / Linux
- ZCode plugin format: skills + commands + hooks (manifest `.zcode-plugin/plugin.json`)

## License

MIT — see [LICENSE](LICENSE).

---

## 中文说明

[English](#zcode-session-usage) | **中文**

一个 [ZCode](https://zcode.z.ai) 插件：**在每条回复末尾自动显示会话用量统计**，格式与 [Cherry Studio](https://github.com/CherryHQ/cherry-studio) 的消息统计一致：

```text
— 1718.6万 Tokens · 37.9 Token/秒
```

另附 `/usage` 命令，一键输出完整用量面板：输入/输出 Token、缓存命中、模型生成 TPS、端到端吞吐/耗时与本地费用估算。

## 功能

- **每条回复自动带页脚** —— `UserPromptSubmit` 钩子在每次发消息时把实时累计用量注入对话，模型据此在回复末尾附加 `— N万 Tokens · X.X Token/秒`，零手动操作
- **`/usage` 完整面板** —— 输入/输出、缓存读取/写入/未缓存构成、推理估算、模型生成 TPS、端到端耗时与吞吐、多模型分列
- **本地费用估算** —— 把模型单价（US$/1M tokens）填进 `prices.json`，面板即显示美元费用
- **纯本地离线** —— 只读 ZCode 自己的模型 I/O 日志（`~/.zcode/cli/rollout/`），不上传任何数据，不涉密钥
- **仅标准库** —— 一个 Python 脚本，零依赖

## 安装

1. 打开 ZCode **插件市场 → 添加 → 添加插件市场**
2. 粘贴：`2237423553/zcode-session-usage`（或完整地址 `https://github.com/2237423553/zcode-session-usage.git`，任意 git 托管均可）
3. 在 **个人** 页找到 **会话用量 / Session Usage**，点 **安装**
4. **新建一个任务**（重要：ZCode 在会话启动时注册插件钩子，老会话加载不到）发两条消息——第 2 条回复起，页脚自动出现

> 想看完整面板：输入 `/usage`。费用默认显示"—"，把模型单价填进 `session-usage/skills/session-usage/scripts/prices.json` 即可估算。

## 工作原理

- 每条用户消息时钩子运行统计脚本，读取当前会话的模型 I/O 日志（每行一次模型调用：token、缓存、耗时）并注入累计数字；模型据此附加页脚（数字截至该条回复开始前）
- ZCode 默认会裁剪模型 I/O 日志（`modelIoFullRetentionEnabled`），"累计"指日志保留窗口内的累计
- 推理 token 按文本长度估算（CJK ≈ 1 token/字符，其他 ≈ 0.25）；日志无逐 token 的首字时间，故显示"—"

## 兼容性

- 系统需有 `python`（3.x，仅标准库）在 PATH
- Windows / macOS / Linux
- 插件形态：技能 + 命令 + 钩子（清单 `.zcode-plugin/plugin.json`）

## License

MIT —— 见 [LICENSE](LICENSE)。
