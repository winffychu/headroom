# Python SDK

在任意 Python 应用中使用 `compress()` 函数。

## 基本用法

```python
from headroom import compress

messages = [
    {"role": "user", "content": "大量的上下文内容..."}
]

# 压缩 messages
compressed = compress(messages, model="claude-sonnet-4-20250514")
# 返回压缩后的 messages，可直接发送给 LLM
```

## 与 Anthropic SDK 配合

```python
import anthropic
from headroom import compress

client = anthropic.Anthropic()

messages = [{"role": "user", "content": "长文本..."}]
compressed_messages = compress(messages, model="claude-sonnet-4-20250514")

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    messages=compressed_messages,
    max_tokens=1024,
)
```

## 与 OpenAI SDK 配合

```python
from openai import OpenAI
from headroom import compress

client = OpenAI()

messages = [{"role": "user", "content": "长文本..."}]
compressed_messages = compress(messages, model="gpt-4o")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=compressed_messages,
)
```

## 框架集成

| 框架 | 方式 |
|------|------|
| LangChain | `HeadroomChatModel(your_llm)` |
| Agno | `HeadroomAgnoModel(your_model)` |
| LiteLLM | `litellm.callbacks = [HeadroomCallback()]` |
| Vercel AI SDK | `headroomMiddleware()` |
| ASGI | `app.add_middleware(CompressionMiddleware)` |
