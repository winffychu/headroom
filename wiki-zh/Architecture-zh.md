# 架构说明

## 数据流

```
 你的代理 / 应用
   (Claude Code, Cursor, Codex, LangChain, Agno, Strands, 你自己的代码…)
        │   prompts · 工具输出 · 日志 · RAG 结果 · 文件
        ▼
    ┌────────────────────────────────────────────────────┐
    │  Headroom   （本地运行 — 数据留在本地）              │
    │  ────────────────────────────────────────────────  │
    │  CacheAligner  →  ContentRouter  →  CCR            │
    │                    ├─ SmartCrusher   (JSON)        │
    │                    ├─ CodeCompressor (AST)         │
    │                    └─ Kompress-base  (文本, HF)    │
    │                                                    │
    │  跨代理记忆  ·  headroom learn  ·  MCP             │
    └────────────────────────────────────────────────────┘
        │   压缩后的 prompt  +  检索工具
        ▼
 LLM Provider  (Anthropic · OpenAI · Bedrock · …)
```

## 核心组件

### ContentRouter（内容路由器）
检测输入内容的类型，自动选择最佳压缩算法。

### SmartCrusher（智能压缩）
通用 JSON 压缩器，处理字典数组、嵌套对象、混合类型。

### CodeCompressor（代码压缩器）
AST 感知的代码压缩器，支持 Python、JS、Go、Rust、Java、C++。

### Kompress-base（基础压缩模型）
基于 HuggingFace 的专用模型，在代理交互轨迹上训练。

### CacheAligner（缓存对齐器）
稳定 prompt 前缀，使 Anthropic/OpenAI 的 KV 缓存真正命中，而非因内容变化导致缓存失效。

### CCR（可逆压缩）
在本地缓存原始内容，LLM 可通过 `headroom_retrieve` 按需检索。

## 请求生命周期

`Setup` → `Pre-Start` → `Post-Start` → `Input Received` → `Input Cached` → `Input Routed` → `Input Compressed` → `Input Remembered` → `Pre-Send` → `Post-Send` → `Response Received`
