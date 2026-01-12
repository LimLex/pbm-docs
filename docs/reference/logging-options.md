# 日志选项

以下选项用于 pbm-agent 配置。在[使用配置文件启动 pbm-agent](../manage/start-agent-with-config.md) 章节中了解更多信息。

```yaml
log:
  path: "/var/log/pbm.json"
  level: "info"
  json: true
```

### path

*类型*：字符串

日志文件的路径。如果文件不存在，则创建该文件。默认值为 `/dev/stderr`，这意味着日志写入标准错误输出。如果 PBM 由于某种错误无法将日志写入指定路径，它会回退到默认路径。

### level

*类型*：字符串

日志严重性级别。支持的级别有（从低到高）：D - Debug（默认）、I - Info、W - Warning、E - Error、F - Fatal。

输出包括指定的严重性级别和所有更高级别。

### json

*类型*：boolean

以 JSON 格式输出日志消息。如果未定义，日志以默认文本格式写入。
