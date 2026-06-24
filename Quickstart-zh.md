# 快速开始（60 秒）

## 安装

```bash
# Python
pip install "headroom-ai[all]"

# 或 Node/TypeScript
npm install headroom-ai

# 或 Docker
docker pull ghcr.io/chopratejas/headroom:latest
```

可选扩展：`[proxy]`、`[mcp]`、`[ml]`、`[code]`、`[memory]`、`[relevance]`、`[image]`、`[agno]`、`[langchain]`、`[evals]`

需要 **Python 3.10+**。

## 选择模式

### 方式 1：包装编码代理（最简单）

```bash
headroom wrap claude
# 或
headroom wrap cursor
# 或
headroom wrap copilot
```

### 方式 2：启动代理（零代码改动）

```bash
headroom proxy --port 8787
```

然后设置客户端指向代理：

```bash
# Claude Code
ANTHROPIC_BASE_URL=http://localhost:8787 claude

# 任何 OpenAI 兼容客户端
OPENAI_BASE_URL=http://localhost:8787/v1
```

### 方式 3：内联库

```python
from headroom import compress

compressed = compress(messages, model="claude-sonnet-4")
```

## 验证节省

```bash
headroom perf
headroom dashboard    # 实时看板（需代理运行中）
```
