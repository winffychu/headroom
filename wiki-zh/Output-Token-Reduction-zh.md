# 输出 Token 减少

除了压缩输入（发送给模型的 prompt），Headroom 还能减少模型**写回**的输出 Token。

## 为什么需要？

- Opus 级别模型输出成本是输入的 5 倍
- 大量输出是浪费的："好的，让我…"开场白、重复代码、常规步骤的深度思考

## 开启方式

```bash
export HEADROOM_OUTPUT_SHAPER=1
headroom proxy --port 8787
```

## 工作原理

### 简洁度引导
在系统 prompt 末尾附加一条简短提示（"简洁，不要复述上下文"），不影响 prompt 缓存。

### 努力度路由
当某轮只是模型在工具结果后继续执行（如读取文件、通过测试），调低思考努力度。新问题和错误保持全力。

## 自动学习

```bash
headroom learn --verbosity            # 预览（干运行）
headroom learn --verbosity --apply    # 保存设置
```

## 查看节省

```bash
headroom output-savings
# 输出类似：Reduction: 31.7% (95% CI 27.7% … 35.7%)
```

设置 `HEADROOM_OUTPUT_HOLDOUT=0.1` 保留 10% 对话为对照组，获得实测数据而非估算。
