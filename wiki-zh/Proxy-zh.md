# 代理模式

Headroom 代理是一个**独立的 HTTP 服务器**，适合非 Python 应用或仅支持配置 BASE_URL 的工具（Claude Code、Cursor、Copilot CLI 等）。

## 启动

```bash
headroom proxy --port 8787
```

### 常用选项

| 选项 | 默认值 | 说明 |
|------|--------|------|
| `--host` | 127.0.0.1 | 绑定地址 |
| `--port` | 8787 | 监听端口 |
| `--mode` | token | `token`（最大压缩）或 `cache`（缓存友好） |
| `--anthropic-api-url` | https://api.anthropic.com | Anthropic 上游地址 |
| `--openai-api-url` | https://api.openai.com | OpenAI 上游地址 |
| `--code-aware` | true | AST 代码压缩 |
| `--budget` | 无 | 每日预算上限（USD） |

### 环境变量

```bash
export HEADROOM_HOST=0.0.0.0
export HEADROOM_PORT=8787
export HEADROOM_BUDGET=100.0
export HEADROOM_MODE=token

# 自定义上游端点
export OPENAI_TARGET_API_URL=https://custom.endpoint.com/v1
export ANTHROPIC_TARGET_API_URL=https://litellm.company.internal
```

## 集成示例

### OpenAI SDK

```python
from openai import OpenAI
client = OpenAI(
    base_url="http://localhost:8787/v1",
    api_key="your-api-key",  # 透传至上游
)
```

### Claude Code

```bash
ANTHROPIC_BASE_URL=http://localhost:8787 claude
```

### Cursor

设置 `OPENAI_BASE_URL=http://localhost:8787/v1`

## 云 Provider

```bash
# AWS Bedrock
headroom proxy --backend bedrock --region us-east-1

# Google Vertex AI
headroom proxy --backend vertex_ai --region us-central1

# Azure OpenAI
headroom proxy --backend azure

# OpenRouter
OPENROUTER_API_KEY=sk-or-... headroom proxy --backend openrouter
```

## 生产部署

```bash
# Gunicorn
gunicorn headroom.proxy.server:app \
  --workers 4 \
  --bind 0.0.0.0:8787 \
  --worker-class uvicorn.workers.UvicornWorker
```
