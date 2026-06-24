# Headroom Wiki 中文文档

欢迎来到 Headroom 中文 Wiki！Headroom 是一个上下文压缩层，可减少 AI 代理与 LLM 交互时的 Token 消耗达 60-95%。

## 快速导航

- [快速开始](Quickstart-zh) — 60 秒上手
- [代理模式](Proxy-zh) — 零代码改动，即插即用
- [Python SDK](Python-SDK-zh) — 内联压缩库
- [MCP 集成](MCP-zh) — Model Context Protocol
- [配置参考](Configuration-zh) — 环境变量与 CLI 参数
- [架构说明](Architecture-zh) — 工作原理
- [输出 Token 减少](Output-Token-Reduction-zh) — 裁剪模型写回内容
- [常见问题](FAQ-zh)

## Headroom 是什么？

Headroom 是一个**上下文压缩工具**，在 AI 代理（如 Claude Code、Cursor、Codex 等）向 LLM 发送请求之前，自动压缩其中的工具输出、日志、RAG 块、文件和对话历史。压缩后答案质量不变，但 Token 消耗大幅降低。

### 核心能力

| 能力 | 说明 |
|------|------|
| **库模式** | 在 Python/TypeScript 中内联调用 `compress(messages)` |
| **代理模式** | `headroom proxy --port 8787`，任何语言零代码改动 |
| **Agent Wrap** | `headroom wrap claude/cursor/copilot/...` 一行命令 |
| **MCP 服务器** | 为任何 MCP 客户端提供 `headroom_compress` 等工具 |
| **跨代理记忆** | Claude、Codex、Gemini 之间共享记忆，自动去重 |
| **可逆压缩** | 原始内容缓存本地，LLM 可按需检索 |
| **失败学习** | 挖掘失败会话，自动写入 CLAUDE.md / AGENTS.md |

### 支持的代理

| 代理 | `headroom wrap` | 说明 |
|------|:---------------:|------|
| Claude Code | ✅ | `--memory` · `--code-graph` |
| Codex | ✅ | 与 Claude 共享记忆 |
| Cursor | ✅ | 打印配置，粘贴一次 |
| Aider | ✅ | 启动代理 + 启动 |
| Copilot CLI | ✅ | 启动代理 + 启动 |
| OpenClaw | ✅ | ContextEngine 插件 |
| OpenCode | ✅ | 注入配置 · 启动 + 启动 |
| Cortex Code | ✅ | 60-65% 节省 · 库模式 |

任何 OpenAI 兼容客户端都可通过 `headroom proxy` 使用。
