<div align="center"><pre>
  ██╗  ██╗███████╗ █████╗ ██████╗ ██████╗  ██████╗  ██████╗ ███╗   ███╗
  ██║  ██║██╔════╝██╔══██╗██╔══██╗██╔══██╗██╔═══██╗██╔═══██╗████╗ ████║
  ███████║█████╗  ███████║██║  ██║██████╔╝██║   ██║██║   ██║██╔████╔██║
  ██╔══██║██╔══╝  ██╔══██║██║  ██║██╔══██╗██║   ██║██║   ██║██║╚██╔╝██║
  ██║  ██║███████║██║  ██║██████╔╝██║  ██║╚██████╔╝╚██████╔╝██║ ╚═╝ ██║
  ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝╚═════╝ ╚═╝  ╚═╝ ╚═════╝  ╚═════╝ ╚═╝     ╚═╝
                  AI 代理的上下文压缩层
</pre></div>

<p align="center"><strong>减少 60–95% token · 库 · 代理 · MCP · 6 种算法 · 本地优先 · 可逆</strong></p>

<p align="center">
  <a href="https://github.com/chopratejas/headroom/actions/workflows/ci.yml"><img src="https://github.com/chopratejas/headroom/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
  <a href="https://app.codecov.io/gh/chopratejas/headroom"><img src="https://codecov.io/gh/chopratejas/headroom/graph/badge.svg" alt="codecov"></a>
  <a href="https://pypi.org/project/headroom-ai/"><img src="https://img.shields.io/pypi/v/headroom-ai.svg" alt="PyPI"></a>
  <a href="https://www.npmjs.com/package/headroom-ai"><img src="https://img.shields.io/npm/v/headroom-ai.svg" alt="npm"></a>
  <a href="https://huggingface.co/chopratejas/kompress-v2-base"><img src="https://img.shields.io/badge/model-Kompress--v2--base-yellow.svg" alt="Model: Kompress-v2-base"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-Apache%202.0-blue.svg" alt="License: Apache 2.0"></a>
  <a href="https://headroom-docs.vercel.app/docs"><img src="https://img.shields.io/badge/docs-online-blue.svg" alt="Docs"></a>
</p>

<p align="center">
  <a href="https://headroom-docs.vercel.app/docs">文档</a> ·
  <a href="#快速开始-60-秒">安装</a> ·
  <a href="#效果证明">效果</a> ·
  <a href="#代理兼容性矩阵">代理</a> ·
  <a href="https://discord.gg/yRmaUNpsPJ">Discord</a> ·
  <a href="llms.txt">llms.txt</a> ·
  <a href="ENTERPRISE.md">企业版</a>
</p>

<p align="center"><sub>
  <b>AI 代理 / LLM：</b> 阅读 <a href="llms.txt"><code>/llms.txt</code></a>，或获取 <a href="https://headroom-docs.vercel.app/llms.txt">实时索引</a> / <a href="https://headroom-docs.vercel.app/llms-full.txt">完整文档</a>。
</sub></p>

---

<p align="center"><a href="https://trendshift.io/repositories/20881" target="_blank"><img src="https://trendshift.io/api/badge/repositories/20881" alt="chopratejas%2Fheadroom | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a></p>

Headroom 压缩 AI 代理读取的一切内容——工具输出、日志、RAG 块、文件和对话历史——在到达 LLM 之前完成压缩。答案不变，Token 只需零头。

<p align="center">
  <img src="HeadroomDemo-Fast.gif" alt="Headroom 运行演示" width="820">
  <br/><sub>实况：10,144 → 1,260 tokens — 同样找到 FATAL 错误。</sub>
</p>

## 它能做什么

- **库** — 在 Python 或 TypeScript 中内联使用 `compress(messages)`
- **代理** — `headroom proxy --port 8787`，零代码改动，任何语言
- **编码代理包装** — `headroom wrap claude|codex|cursor|aider|copilot|opencode` 一行命令
- **MCP 服务器** — 为任何 MCP 客户端提供 `headroom_compress`、`headroom_retrieve`、`headroom_stats`
- **跨代理记忆** — 在 Claude、Codex、Gemini 之间共享存储，自动去重
- **`headroom learn`** — 挖掘失败会话，自动将修正写入 `CLAUDE.md` / `AGENTS.md`
- **输出 Token 减少** — 裁剪模型写回的内容（不仅是发送的内容）：去除客套话/重复代码，对常规步骤跳过深度"思考"。参见[输出 Token 减少](#output-token-reduction-cut-what-the-model-writes-back)
- **可逆（CCR）** — 原始内容缓存在本地，可按需检索

## 工作原理（30 秒）

```
 你的代理 / 应用
   (Claude Code, Cursor, Codex, LangChain, Agno, Strands, 你自己的代码…)
        │   prompts · 工具输出 · 日志 · RAG 结果 · 文件
        ▼
    ┌────────────────────────────────────────────────────┐
    │  Headroom   （本地运行 — 你的数据留在本地）          │
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

- **ContentRouter** — 检测内容类型，选择正确的压缩器
- **SmartCrusher / CodeCompressor / Kompress-base** — 压缩 JSON、AST 或散文
- **CacheAligner** — 稳定前缀，使 Provider KV 缓存真正命中
- **CCR** — 在本地存储原始内容；LLM 需要时调用 `headroom_retrieve`

→ [架构](https://headroom-docs.vercel.app/docs/architecture) · [CCR 可逆压缩](https://headroom-docs.vercel.app/docs/ccr) · [Kompress-v2-base 模型卡](https://huggingface.co/chopratejas/kompress-v2-base)

## 快速开始（60 秒）

```bash
# 1 — 安装
pip install "headroom-ai[all]"          # Python
npm install headroom-ai                 # Node / TypeScript

# 2 — 选择你的模式
headroom wrap claude                    # 包装编码代理
headroom proxy --port 8787              # 即插即用代理，零代码改动
# 或：from headroom import compress      # 内联库

# 3 — 查看节省效果
headroom perf
headroom dashboard                      # 实时节省看板（需要代理正在运行）
```

可选扩展：`[proxy]`、`[mcp]`、`[ml]`、`[code]`、`[memory]`、`[relevance]`、`[image]`、`[agno]`、`[langchain]`、`[evals]`、`[pytorch-mps]`（Apple GPU 内存嵌入器卸载 — 设置 `HEADROOM_EMBEDDER_RUNTIME=pytorch_mps`）。需要 **Python 3.10+**。

## 效果证明

**真实代理工作负载的节省效果：**

| 工作负载                      | 压缩前 | 压缩后 | 节省 |
|-------------------------------|-------:|-------:|--------:|
| 代码搜索（100 条结果）        | 17,765 |  1,408 | **92%** |
| SRE 故障排查                  | 65,694 |  5,118 | **92%** |
| GitHub Issue 分类             | 54,174 | 14,761 | **73%** |
| 代码库探索                    | 78,502 | 41,254 | **47%** |

**标准基准测试准确度保持：**

| 基准测试  | 分类 | 样本数 | 基线 | Headroom | 差异 |
|-----------|----------|----:|---------:|---------:|------------|
| GSM8K     | 数学 | 100 | 0.870 | 0.870 | **±0.000** |
| TruthfulQA| 事实性 | 100 | 0.530 | 0.560 | **+0.030** |
| SQuAD v2  | 问答 | 100 | — | **97%** | 19% 压缩率 |
| BFCL      | 工具 | 100 | — | **97%** | 32% 压缩率 |

复现：`python -m headroom.evals suite --tier 1` · [完整基准与方法论](https://headroom-docs.vercel.app/docs/benchmarks)

## 输出 Token 减少（裁剪模型写回的内容）

以上所有内容都在缩小你**发送**的 prompt。但你同样需要为模型**写回**的每个 token 付费——在 Opus 级别模型上，输出成本是输入的 5 倍。很多输出是浪费的："好的，让我…"这类开场白、重复你刚刚展示的代码，以及读取文件等常规步骤上的深度"思考"。

Headroom 也可以从代理层面裁剪这些，无需修改代码：

- **简洁度引导** — 在系统 prompt 末尾附加一条简短提示（"简洁，不要复述上下文"），这样你的 prompt 缓存仍然命中。
- **努力度路由** — 当某轮只是模型在工具结果后继续执行（如读取文件、通过测试），它会调低模型的思考努力度。新问题和错误则保持全力输出。

开启方式：

```bash
export HEADROOM_OUTPUT_SHAPER=1     # 默认关闭
headroom proxy --port 8787
```

> **代理已经在运行？** 这些开关会在每个请求上*实时*读取，因此如果 `headroom wrap` **复用**（而非启动）了一个已有代理，它不会看到你在启动后 export 的值——环境变量在启动时已快照。现在 `headroom wrap` 会通过回环 `POST /admin/runtime-env` 热同步当前设置到运行中的代理，使它们**无需重启**立即生效（无冷启动、无请求丢弃、无缓存丢失）。在 `wrap` 之前设置即可。在共享代理上，这些覆盖是全局的——最后显式设置的生效。

**学习适合你的简洁度。** 人们不会说他们想要多简洁——他们会展示（打断长回复，或在读完之前就切换到下一步）。`headroom learn --verbosity` 读取你的历史会话，自动选择合适级别：

```bash
headroom learn --verbosity            # 预览发现结果（干运行）
headroom learn --verbosity --apply    # 保存设置；代理从此使用
```

**查看你节省了多少输出 Token。** 输出节省是*反事实的*——我们从未看到模型原本会写什么——因此 Headroom 报告一个诚实的**带有置信区间的估算值**，而非编造的数字：

```bash
headroom output-savings
# 减少: 31.7%  (95% CI 27.7% … 35.7%)   [估算值]
```

想要*实测*而非估算值？保留 10% 的对话不做塑造作为对照组：`export HEADROOM_OUTPUT_HOLDOUT=0.1`。看板会在输入压缩旁边显示一张**输出 Token 节省**卡片，标注 `实测` 或 `估算` 及置信区间。

→ 完整说明（含测量方法论）：[`docs/proposals/output-token-reduction.md`](docs/proposals/output-token-reduction.md)

<a href="https://www.star-history.com/?repos=chopratejas%2Fheadroom&type=date&legend=top-left">
 <picture>
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=chopratejas/headroom&type=date&legend=top-left" />
 </picture>
</a>

## 代理兼容性矩阵

| 代理         | `headroom wrap` | 说明                            |
|--------------|:---------------:|----------------------------------|
| Claude Code  | ✅              | `--memory` · `--code-graph`      |
| Codex        | ✅              | 与 Claude 共享记忆               |
| Cursor       | ✅              | 打印配置 — 粘贴一次即可          |
| Aider        | ✅              | 启动代理 + 启动                  |
| Copilot CLI  | ✅              | 启动代理 + 启动                  |
| OpenClaw     | ✅              | 作为 ContextEngine 插件安装      |
| OpenCode     | ✅              | 注入配置 · 启动代理 + 启动      |
| Cortex Code  | ✅              | 60–65% 节省 · 库模式            |

任何 OpenAI 兼容客户端都能通过 `headroom proxy` 使用。MCP 原生：`headroom mcp install`。

### GitHub Copilot CLI 订阅模式

Headroom 可以将 GitHub Copilot CLI 订阅流量路由到本地代理：

```bash
headroom copilot-auth login
headroom wrap copilot --subscription -- --model gpt-4o
```

这让 Headroom 拦截 OpenAI 兼容的 Copilot CLI 请求，在转发到 GitHub Copilot 托管 API 之前应用相同的代理压缩管道。该包装器将 Headroom 的可复用 GitHub OAuth token 交换为 Copilot 的短期 API token，并在启动时打印上游端点为 `COPILOT_PROVIDER_API_URL=...`。

`headroom copilot-auth login` 存储一个 Headroom 专属的 Copilot OAuth token。这避免了依赖可以读取 Copilot 账户元数据但仍可能被 Copilot token 交换端点拒绝的通用 GitHub 或 Copilot CLI token。

对于 GitHub Enterprise Server 或自定义域名的 Copilot 部署，在启动前设置部署域名：

```bash
export GITHUB_COPILOT_ENTERPRISE_DOMAIN=ghe.example.com
```

对于 GitHub.com Enterprise Cloud URL（如 `github.com/enterprises/your-enterprise`），无需设置企业域名覆盖。Headroom 使用 GitHub 的正常 token 交换端点和登录账号所公告的 Copilot API 端点。

平台支持说明：通过 Copilot CLI Keychain 存储的 macOS 认证复用已通过冒烟测试。Windows Credential Manager、Linux Secret Service / `secret-tool` 以及 Docker/CI token 注入路径已实现或已计划作为认证发现路径，但在被视为完全验证之前仍需真实 OS 验证。对于 Docker 和 CI，建议传递显式的 `GITHUB_COPILOT_TOKEN` 或 `GITHUB_COPILOT_GITHUB_TOKEN`，而非依赖主机 keychain 访问。

## 何时使用 · 何时跳过

**非常适合你如果……**
- 日常使用 AI 编码代理，希望不修改代码就能节省 token
- 在多个代理之间工作，需要共享记忆
- 需要可逆压缩——在配置的 TTL 内可通过 CCR 检索原始内容

**跳过它如果……**
- 只使用单一提供商的原生压缩功能，不需要跨代理记忆
- 在沙盒环境中工作，无法运行本地进程

<details>
<summary><b>集成 — 将 Headroom 接入任何技术栈</b></summary>

| 你的场景                   | 接入方式                                                     |
|---------------------------|--------------------------------------------------------------|
| 任意 Python 应用          | `compress(messages, model=…)`                                |
| 任意 TypeScript 应用      | `await compress(messages, { model })`                        |
| Anthropic / OpenAI SDK    | `withHeadroom(new Anthropic())` · `withHeadroom(new OpenAI())` |
| Vercel AI SDK             | `wrapLanguageModel({ model, middleware: headroomMiddleware() })` |
| LiteLLM                   | `litellm.callbacks = [HeadroomCallback()]`                   |
| LangChain                 | `HeadroomChatModel(your_llm)`                                |
| Agno                      | `HeadroomAgnoModel(your_model)`                              |
| Strands                   | [Strands 指南](https://headroom-docs.vercel.app/docs/strands) |
| ASGI 应用                 | `app.add_middleware(CompressionMiddleware)`                  |
| 多代理                     | `SharedContext().put / .get`                                 |
| MCP 客户端                | `headroom mcp install`                                       |

</details>

<details>
<summary><b>内部组件</b></summary>

- **SmartCrusher** — 通用 JSON：字典数组、嵌套对象、混合类型
- **CodeCompressor** — AST 感知的 Python、JS、Go、Rust、Java、C++
- **Kompress-base** — 我们的 HuggingFace 模型，基于代理轨迹训练
- **图像压缩** — 通过训练的 ML 路由器实现 40–90% 缩减
- **CacheAligner** — 稳定前缀，使 Anthropic/OpenAI KV 缓存真正命中
- **IntelligentContext** — 基于评分的上下文适配，学习重要性
- **CCR** — 可逆压缩；LLM 按需检索原始内容
- **跨代理记忆** — 共享存储、代理来源追踪、自动去重
- **SharedContext** — 跨多代理工作流的压缩上下文传递
- **`headroom learn`** — 基于插件的失败挖掘，支持 Claude、Codex、Gemini

</details>

<details>
<summary><b>管道内部</b></summary>

Headroom 在 `compress()`、SDK 和代理之间暴露一个稳定的请求生命周期：

`Setup` → `Pre-Start` → `Post-Start` → `Input Received` → `Input Cached` → `Input Routed` → `Input Compressed` → `Input Remembered` → `Pre-Send` → `Post-Send` → `Response Received`

- **Transforms** 执行实际工作：CacheAligner、ContentRouter、SmartCrusher、CodeCompressor、Kompress-base、IntelligentContext / RollingWindow
- **Pipeline extensions** 通过 `on_pipeline_event(...)` 观察或定制生命周期阶段
- **Compression hooks** 作为额外的扩展接缝与标准生命周期并置
- **Proxy extensions** 仍然是 ASGI 中间件、路由和启动策略的服务器/应用集成接缝

Provider 和工具特定行为位于 `headroom/providers/` 下，使核心编排专注于生命周期、排序和策略。

- **CLI/工具切片**：`headroom/providers/claude`、`copilot`、`codex`、`openclaw`
- **Provider 运行时切片**：`headroom/providers/claude`、`gemini`，加上 `headroom/providers/registry.py` 中的共享后端/运行时分发
- **核心文件保持编排优先**：`wrap.py`、`client.py`、`cli/proxy.py` 和 `proxy/server.py` 代理 Provider 特定的环境塑造、API 目标标准化、后端选择和传输分发

</details>

## 安装

```bash
pip install "headroom-ai[all]"          # Python，包含所有功能
npm install headroom-ai                 # TypeScript / Node
docker pull ghcr.io/chopratejas/headroom:latest
```

可选扩展：`[proxy]`、`[mcp]`、`[ml]`（Kompress-base）、`[code]`、`[memory]`、`[relevance]`、`[image]`、`[agno]`、`[langchain]`、`[evals]`、`[pytorch-mps]`（Apple GPU 内存嵌入器卸载 — 设置 `HEADROOM_EMBEDDER_RUNTIME=pytorch_mps`）。需要 **Python 3.10+**。

使用 `pipx`？显式选择一个受支持的 Python 解释器：

```bash
pipx install --python python3.13 "headroom-ai[all]"
```

→ [安装指南](https://headroom-docs.vercel.app/docs/installation) — Docker 标签、持久化服务、PowerShell、devcontainers

### 更新

```bash
headroom update          # 检测 pip / pipx / uv tool 并在原地升级
headroom update --check  # 报告最新版本而不升级
headroom update --pre    # 包含预发布版本
```

`headroom update` 会判断 Headroom 的安装方式（pip/venv、`pip --user`、pipx、uv tool）并执行对应的升级，支持 macOS、Linux 和 Windows。对于 git checkout、可编辑安装、Docker 镜像和外部管理的系统 Python（PEP 668），它会打印出正确的手动步骤而非猜测。

代理启动时也会显示一行"有可用更新"的提示。它每天最多检查一次 PyPI，在后台进行，从不阻塞。通过 `HEADROOM_UPDATE_CHECK=off` 退出（在 `--stateless` 模式和 CI 中也会跳过）。

### 企业 / SSL 审查环境

如果 `pip install "headroom-ai[all]"` 因 `CERTIFICATE_VERIFY_FAILED`（`unable to get local issuer certificate`）而失败，说明你的网络使用了 **SSL 审查**——一个使用公司签发 CA 的 MITM 代理。构建后端（`maturin`）会通过你的 TLS 栈不信任的连接下载 `rustup`。**先安装 Rust** 以避免构建时获取它：

```bash
# macOS / Linux
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh && rustup default stable
# Windows
winget install Rustlang.Rustup && rustup default stable
```

重启 shell，然后 `pip install "headroom-ai[all]"`。如果可用，预编译 wheel 可以完全避免 Rust 构建：`pip install --only-binary headroom-ai headroom-ai`。

两个运行时资源通过 TLS 获取；如果它们被阻断，通过 `REQUESTS_CA_BUNDLE` / `SSL_CERT_FILE` / `CURL_CA_BUNDLE` 信任你的企业 CA：

- **`cdn.pyke.io`** — Rust 核心的 ONNX Runtime。或者通过 `ORT_STRATEGY=system` 和 `ORT_LIB_LOCATION=/path/to/onnxruntime` 预先提供。
- **`huggingface.co`** — `kompress-base` 压缩模型。预先下载并用 `HF_HUB_OFFLINE=1` 运行，或设置 `HF_ENDPOINT` 指向受信镜像。

在禁用压缩的情况下运行（纯网关）不需要这两项资源。

## headroom learn

<p align="center">
  <img src="headroom_learn.gif" alt="headroom learn 运行演示" width="720">
</p>

`headroom learn` — 挖掘失败会话，将修正写入 `CLAUDE.md` / `AGENTS.md` / `GEMINI.md`。

## 文档

| 从这里开始                                                                  | 深入阅读                                                                          |
|-------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| [快速开始](https://headroom-docs.vercel.app/docs/quickstart)                 | [架构](https://headroom-docs.vercel.app/docs/architecture)                         |
| [代理](https://headroom-docs.vercel.app/docs/proxy)                          | [压缩原理](https://headroom-docs.vercel.app/docs/how-compression-works)            |
| [MCP 工具](https://headroom-docs.vercel.app/docs/mcp)                        | [CCR — 可逆压缩](https://headroom-docs.vercel.app/docs/ccr)                        |
| [记忆系统](https://headroom-docs.vercel.app/docs/memory)                     | [缓存优化](https://headroom-docs.vercel.app/docs/cache-optimization)               |
| [失败学习](https://headroom-docs.vercel.app/docs/failure-learning)           | [基准测试](https://headroom-docs.vercel.app/docs/benchmarks)                       |
| [配置](https://headroom-docs.vercel.app/docs/configuration)                  | [局限性](https://headroom-docs.vercel.app/docs/limitations)                        |

## 对比

Headroom **本地运行**、覆盖**每一种**内容类型、兼容所有主流框架，并且是**可逆的**。

|                                                                               | 范围                                            | 部署方式                            | 本地 | 可逆 |
|-------------------------------------------------------------------------------|------------------------------------------------|------------------------------------|:-----:|:----------:|
| **Headroom**                                                                  | 所有上下文 — 工具、RAG、日志、文件、历史记录     | 代理 · 库 · 中间件 · MCP           | 是   | 是        |
| [RTK](https://github.com/rtk-ai/rtk)                                         | CLI 命令输出                                   | CLI 包装器                         | 是   | 否         |
| [lean-ctx](https://github.com/yvgude/lean-ctx)                               | CLI 命令、MCP 工具、编辑器规则                  | CLI 包装器 · MCP                    | 是   | 否         |
| [Compresr](https://compresr.ai)、[Token Co.](https://thetokencompany.ai)     | 发送到其 API 的文本                             | 托管 API 调用                      | 否   | 否         |
| OpenAI Compaction                                                             | 对话历史                                       | Provider 原生                      | 否   | 否         |

> **致谢。** Headroom 附带优秀的 [RTK](https://github.com/rtk-ai/rtk) 二进制工具用于 shell 输出重写——`git show --short`、限定范围的 `ls`、汇总安装器。衷心感谢 RTK 团队；他们的工具是我们技术栈的一等公民，Headroom 压缩其下游的所有内容。Headroom 也可以使用 [lean-ctx](https://github.com/yvgude/lean-ctx) 作为选定的 CLI 上下文工具；在运行 `headroom wrap ...` 前设置 `HEADROOM_CONTEXT_TOOL=lean-ctx`。

## 贡献

```bash
git clone https://github.com/chopratejas/headroom.git && cd headroom
uv sync --extra dev && uv run pytest
```

开发容器在 `.devcontainer/` 中（默认 + `memory-stack` 含 Qdrant 和 Neo4j）。参见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 社区

- **[Discord](https://discord.gg/yRmaUNpsPJ)** — 问题、反馈、经验分享
- **[HuggingFace 上的 Kompress-v2-base](https://huggingface.co/chopratejas/kompress-v2-base)** — 文本压缩背后的模型

## 许可证

Apache 2.0 — 参见 [LICENSE](LICENSE)。
