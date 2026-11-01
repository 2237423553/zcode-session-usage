# zcode-session-usage

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-blue)](#compatibility)
[![Python](https://img.shields.io/badge/python-3.x%20stdlib%20only-green)](#compatibility)
[![ZCode](https://img.shields.io/badge/for-ZCode%20%F0%9F%A4%96-7c3aed)](https://zcode.z.ai)

**English** | [中文说明](README.zh-CN.md)

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
