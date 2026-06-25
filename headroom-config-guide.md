# Headroom 配置与模式决策指南

> 基于 headroomlabs/headroom 源码（v0.x）分析整理
> 适用场景：Headroom 作为压缩代理，上游接中转站 / DeepSeek / Anthropic 等

---

## 一、核心架构一句话

```
下游客户端 ──→ Headroom（压缩 + 透传 Key）──→ TARGET_API_URL（上游/中转站）
```

- Headroom **只换 URL，不动 Key**。客户端传什么 Authorization，上游就收到什么
- 支持 `/v1/messages`（Anthropic）、`/v1/chat/completions`、`/v1/responses`（OpenAI/Codex）、`/v1beta/models/*`（Gemini），**自动路由**

---

## 二、完整环境变量清单

### 2.1 上游路由

| 变量 | 默认值 | 说明 |
|---|---|---|
| `ANTHROPIC_TARGET_API_URL` | `https://api.anthropic.com` | Claude/Anthropic 格式请求路由目标 |
| `OPENAI_TARGET_API_URL` | `https://api.openai.com` | OpenAI 格式请求路由目标 |
| `GEMINI_TARGET_API_URL` | Google 默认 | Gemini 格式请求路由目标（检测 `x-goog-api-key`） |
| `VERTEX_TARGET_API_URL` | `us-central1-aiplatform.googleapis.com` | Vertex AI |
| `BEDROCK_TARGET_API_URL` | AWS 默认 | Bedrock |
| `HEADROOM_BACKEND` | `anthropic` | 后端模式：`anthropic` / `anyllm` / `litellm-*` |

### 2.2 网络绑定

| 变量 | 默认值 | 说明 |
|---|---|---|
| `HEADROOM_HOST` | `127.0.0.1` | 监听地址（容器设 `0.0.0.0`） |
| `HEADROOM_PORT` | `8787` | 代理端口 |
| `HEADROOM_WORKERS` | `1` | Uvicorn worker 进程数 |
| `HEADROOM_MAX_CONNECTIONS` | `500` | 最大上游 HTTP 连接数 |
| `HEADROOM_HTTP2` | `true` | 启用 HTTP/2 |

### 2.3 压缩控制

| 变量 | 默认值 | 说明 |
|---|---|---|
| `HEADROOM_MODE` | `token` | **核心模式**：`token`（激进压缩）或 `cache`（仅压最新轮） |
| `HEADROOM_TARGET_RATIO` | 自动 | 目标压缩率，如 `0.5` = 保留 50%，不设则自行判断 |
| `HEADROOM_DISABLE_KOMPRESS` | `false` | 全局禁用压缩 |
| `HEADROOM_COMPRESS_USER_MESSAGES` | 不设（不压用户消息） | 设为 `1` 也压缩用户消息（默认只压 assistant） |

### 2.4 输出裁剪（质量无风险）

| 变量 | 默认值 | 说明 |
|---|---|---|
| `HEADROOM_OUTPUT_SHAPER` | 不启用 | `1` 启用输出侧裁剪 |
| `HEADROOM_VERBOSITY_LEVEL` | `2` | 冗长级别 0-4（见下文） |
| `HEADROOM_EFFORT_ROUTER` | `1` | 机械轮次降智开关 |
| `HEADROOM_MECHANICAL_EFFORT` | `low` | 机械轮次目标 effort |

### 2.5 预算与安全

| 变量 | 默认值 | 说明 |
|---|---|---|
| `HEADROOM_BUDGET` | 不设 | 每日预算上限（美元），超限返回 429 |
| `HEADROOM_BUDGET_PERIOD` | `daily` | 预算周期：`daily` / `weekly` / `monthly` |

### 2.6 缓存后端

| 变量 | 默认值 | 说明 |
|---|---|---|
| `HEADROOM_CCR_BACKEND` | `sqlite` | 设为 `memory` 切回内存 |
| `HEADROOM_CCR_SQLITE_PATH` | `~/.headroom/ccr_store.db` | SQLite 数据库路径 |

---

## 三、核心模式：`token` vs `cache`

### 3.1 源码揭示的唯一差异

```diff
+ token:  压缩全部历史 + 当前轮 assistant 消息
- cache:  仅压缩当前轮（最新一条 assistant 消息）
```

其他所有功能完全一致。

### 3.2 对比

| 维度 | `token` | `cache` |
|---|---|---|
| **压缩范围** | 全部历史 + 当前轮 | 仅当前轮 |
| **历史消息** | 被压缩，再重新冻结 | 完全不动 |
| **输入 token 节省** | 50-70% | 5-15%（仅最后一轮） |
| **输出 token 节省** | + shaper 可达 30-40% | 一样 |
| **质量风险** | ⚠️ 长链对话可能丢失细节 | ✅ 零风险 |
| **Provider 缓存命中** | ❌ 压缩后前缀变化，命中差 | ✅ 前缀不变，完美命中 |
| **适合谁** | 中转站/按量计费 | 官方直连（有缓存折扣）或追求质量 |

### 3.3 量化对比（10 轮对话）

| | 无 Headroom | `cache` | `token` | `token` + ratio=0.5 |
|---|---|---|---|---|
| 每轮输入 | 10K tok | 9.3K tok | 3K tok | 5K tok |
| 每轮输出 | 5K tok | 3K tok | 3K tok | 3K tok |
| 10轮总 tok | 150K | 123K (省 18%) | 60K (省 60%) | 80K (省 47%) |
| 质量风险 | — | ✅ 无 | ⚠️ 有 | ✅ 较低 |

---

## 四、OUTPUT_SHAPER 详解

### 4.1 HEADROOM_VERBOSITY_LEVEL（冗长级别）

| 级别 | 效果 | 质量影响 |
|---|---|---|
| `0` | 不注入任何指令（= 关） | — |
| `1` | "跳过开场白和结语，直接说正事" | ✅ 信息不丢 |
| `2`(默认) | + "不要复现代码/文件内容，引用路径和行号" | ⚠️ 偶尔缺失解释 |
| `3` | + "只给结论，不解释理由" | ⚠️ 调试时可能不够 |
| `4` | 极限精简："碎片化回答，不客套，不总结" | ⚠️ 过度精简 |

**实现方式**：往 system prompt 尾部追加一段 `<headroom_output_shaping>` 指令块，**不动历史 messages**。

**机制**：每次请求都重新注入，不影响之前或之后的请求。

### 4.2 HEADROOM_MECHANICAL_EFFORT（机械轮次降智）

自动检测当前请求类型：

| 检测结果 | 对应行为 |
|---|---|
| **新提问**（最后一条是新文本） | 保持原 effort，全力回答 |
| **机械轮次**（最后是 tool_result 无报错） | 降低 `output_config.effort` 到指定值（默认 `low`） |
| **报错轮次**（tool_result 有 is_error） | 保持原 effort，不降智 |

**实现方式**：修改当前请求 body 的 `output_config.effort` 字段，**不动历史**。

---

## 五、两套最优配置方案

### 方案 A：cache 模式（最安全，推荐首选）

```yaml
HEADROOM_MODE: "cache"
HEADROOM_OUTPUT_SHAPER: "1"
HEADROOM_VERBOSITY_LEVEL: "2"
HEADROOM_MECHANICAL_EFFORT: "low"
```

- 质量风险：**零**
- 节省：**~15-20%**（仅最后一轮压缩 + 输出裁剪）
- 适用：所有场景首选，如果省钱不够再考虑 token

### 方案 B：token 模式（最大省钱）

```yaml
HEADROOM_MODE: "token"
HEADROOM_TARGET_RATIO: "0.5"       # 关键：保守压缩，保留 50%
HEADROOM_OUTPUT_SHAPER: "1"
HEADROOM_VERBOSITY_LEVEL: "2"
HEADROOM_MECHANICAL_EFFORT: "low"
```

- 质量风险：**较低**（ratio=0.5 保证至少一半内容完整）
- 节省：**~50%**
- 适用：对省钱有硬需求，且能接受偶尔上下文遗漏

---

## 六、决策树

```
你的上游是？
├── Anthropic 官方直连（有缓存 90% 折扣）
│   └── cache 模式（保缓存命中，更省钱）
├── DeepSeek 官方（无缓存折扣）
│   ├── 追求质量 → cache + shaper
│   └── 追求省钱 → token + ratio=0.5 + shaper
└── 中转站（sub2api / new-api / 其他）
    ├── 追求质量 → cache + shaper（省 15-20%）
    └── 追求省钱 → token + ratio=0.5 + shaper（省 50%）

质量有问题吗？（用 cache 时）
├── 没有 → 保持 cache
└── 有 → 调高 TARGET_RATIO 或切 cache

省钱不够吗？（用 cache 时）
├── 够 → 保持
└── 不够 → 切 token + ratio=0.5，或降低 ratio
```

---

## 七、完整 docker-compose.yml

```yaml
version: "3.8"

services:
  headroom:
    image: ghcr.io/chopratejas/headroom:latest
    container_name: headroom
    restart: unless-stopped
    ports:
      - "127.0.0.1:8787:8787"

    environment:
      # ── 上游路由 ──────────────────────────────
      ANTHROPIC_TARGET_API_URL: "https://sub2api.example.com"
      OPENAI_TARGET_API_URL: "https://new-api.example.com/v1"
      GEMINI_TARGET_API_URL: ""

      # ── 网络 ──────────────────────────────────
      HEADROOM_HOST: "0.0.0.0"
      HEADROOM_PORT: "8787"

      # ── 压缩（选一套）─────────────────────────
      # 方案 A（推荐）：
      HEADROOM_MODE: "cache"
      HEADROOM_OUTPUT_SHAPER: "1"
      HEADROOM_VERBOSITY_LEVEL: "2"
      HEADROOM_MECHANICAL_EFFORT: "low"
      #
      # 方案 B（省钱）：
      # HEADROOM_MODE: "token"
      # HEADROOM_TARGET_RATIO: "0.5"
      # HEADROOM_OUTPUT_SHAPER: "1"
      # HEADROOM_VERBOSITY_LEVEL: "2"
      # HEADROOM_MECHANICAL_EFFORT: "low"

      # ── 安全兜底 ──────────────────────────────
      HEADROOM_BUDGET: "100.0"
      HEADROOM_BUDGET_PERIOD: "daily"

    volumes:
      - headroom-data:/home/nonroot/.headroom

    healthcheck:
      test: ["CMD", "curl", "--fail", "--silent", "http://127.0.0.1:8787/readyz"]
      interval: 30s
      timeout: 5s
      start_period: 20s
      retries: 3

volumes:
  headroom-data:
```

---

## 八、下游客户端配置速查

```bash
# Claude Code → Headroom → ANTHROPIC_TARGET_API_URL
ANTHROPIC_BASE_URL=http://localhost:8787 \
ANTHROPIC_API_KEY="sk-你的中转站Key" \
claude

# Codex → Headroom → OPENAI_TARGET_API_URL
OPENAI_BASE_URL=http://localhost:8787/v1 \
OPENAI_API_KEY="sk-你的中转站Key" \
codex

# Cursor / 其他 OAI 客户端
OPENAI_BASE_URL=http://localhost:8787/v1
```

---

## 九、关键注意事项

1. **Key 是纯透传**：Headroom 不修改 Authorization header，客户端传什么 Key，上游就收到什么 Key
2. **HEADROOM_OUTPUT_SHAPER 修改的是 system prompt**，不碰历史消息，不影响模型理解能力
3. **CCR SQLite 缓存 TTL=5 分钟**，仅用于避免重复压缩，不影响质量
4. **`cache` 模式仍会压缩最新轮**，不是完全不压缩
5. **容器必须绑定 `127.0.0.1`**（外部不能直连 Headroom），安全依赖本地网络隔离
