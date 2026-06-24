# 配置参考

## 环境变量

### 代理配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `HEADROOM_HOST` | 127.0.0.1 | 绑定地址 |
| `HEADROOM_PORT` | 8787 | 监听端口 |
| `HEADROOM_MODE` | token | 运行模式 |
| `HEADROOM_BUDGET` | 无 | 每日预算（USD） |
| `HEADROOM_BUDGET_PERIOD` | daily | 预算周期 |

### 上游路由

| 变量 | 说明 |
|------|------|
| `OPENAI_TARGET_API_URL` | OpenAI 上游端点 |
| `ANTHROPIC_TARGET_API_URL` | Anthropic 上游端点 |
| `GEMINI_TARGET_API_URL` | Gemini 上游端点 |
| `VERTEX_TARGET_API_URL` | Vertex AI 上游端点 |
| `BEDROCK_TARGET_API_URL` | Bedrock 上游端点 |

### API Key

| 变量 | 说明 |
|------|------|
| `OPENAI_API_KEY` | OpenAI API Key |
| `ANTHROPIC_API_KEY` | Anthropic API Key |
| `OPENROUTER_API_KEY` | OpenRouter API Key |

### 输出优化

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `HEADROOM_OUTPUT_SHAPER` | 0 | 开启输出 Token 减少 |
| `HEADROOM_OUTPUT_HOLDOUT` | 0 | 对照组比例（0.1 = 10%） |

### 其他

| 变量 | 说明 |
|------|------|
| `HEADROOM_UPDATE_CHECK` | 更新检查（设为 off 关闭） |
| `HEADROOM_EMBEDDER_RUNTIME` | 嵌入器运行时（如 pytorch_mps） |
| `HEADROOM_CONTEXT_TOOL` | 上下文工具选择（lean-ctx） |

## 认证模式

Headroom 自动检测请求的认证模式来决定压缩策略：

| 模式 | 检测方式 | 策略 |
|------|----------|------|
| PAYG | `Bearer sk-ant-api*`、`Bearer sk-*`、`x-api-key` | 激进压缩 |
| OAuth | `Bearer sk-ant-oat-*`、JWT、AWS SigV4 | 仅无损压缩 |
| Subscription | UA 前缀 `claude-code/`、`cursor/` 等 | 隐身模式 |
