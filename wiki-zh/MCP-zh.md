# MCP 集成

Headroom 可以作为 MCP（Model Context Protocol）服务器运行，为任何 MCP 客户端提供压缩工具。

## 安装 MCP

```bash
headroom mcp install
```

## 提供的工具

| 工具 | 说明 |
|------|------|
| `headroom_compress` | 压缩文本内容 |
| `headroom_retrieve` | 检索 CCR 缓存的原始内容 |
| `headroom_stats` | 查看压缩统计信息 |

## 启动 MCP 服务

```bash
headroom mcp serve
```

## 手动配置

在 MCP 客户端配置中添加：

```json
{
  "mcpServers": {
    "headroom": {
      "command": "headroom",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
```

## 状态检查

```bash
headroom mcp status
```

## 卸载

```bash
headroom mcp uninstall
```
