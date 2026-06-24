# 常见问题

## Headroom 是否验证 API Key？

不。Headroom 代理是一个透明代理，不验证客户端传入的 API Key。它将 Authorization header 原样透传到上游 LLM Provider。上游 Provider（OpenAI、Anthropic 等）才是真正验证 Key 的地方。

## 如何保证安全？

- Headroom 默认只监听 `127.0.0.1`（本地）
- 将真实 API Key 放在 Headroom 进程的环境变量中，客户端使用占位 Key
- 生产环境建议前置 Nginx 做认证
- 使用 `--budget` 设置每日预算上限

## 支持哪些 Provider？

支持 100+ LLM Provider，通过 LiteLLM 路由：
- OpenAI、Anthropic、Google Gemini
- AWS Bedrock、Azure OpenAI、Vertex AI
- OpenRouter（400+ 模型）
- 任何 OpenAI 兼容端点

## 压缩会降低回答质量吗？

基准测试显示准确度无下降甚至略有提升：
- GSM8K：±0.000
- TruthfulQA：+0.030
- SQuAD v2：97% 准确率（19% 压缩率）

## 什么是 CCR？

CCR（可逆压缩）在本地缓存原始内容，LLM 可以通过 `headroom_retrieve` 工具按需检索。这确保了关键信息不会丢失。

## 如何更新？

```bash
headroom update          # 自动检测安装方式并升级
headroom update --check  # 检查新版本
```

## 支持哪些代理？

Claude Code、Codex、Cursor、Aider、Copilot CLI、OpenClaw、OpenCode、Cortex Code。任何 OpenAI 兼容客户端也可通过代理模式使用。
