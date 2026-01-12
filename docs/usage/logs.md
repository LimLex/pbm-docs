# 查看备份日志

!!! admonition "版本添加：[1.4.0](../release-notes/1.4.0.md)"

您可以使用 `pbm CLI` 查看 MongoDB 环境中所有 `pbm-agents` 的日志。这减少了在排查问题时查找所需信息的时间。

!!! note 
    
    有关从物理备份恢复的日志信息在 pbm logs 中不可用。

要查看 `pbm-agent` 日志，请运行 `pbm logs` 命令并传递一个或多个标志以缩小搜索范围。

以下标志可用：

* `-t`、`--tail` - 显示日志的最后 N 行
* `-e`、`--event` - 按所有备份或特定备份过滤日志
* `-n`、`--node` - 按特定节点或副本集过滤日志
* `-s`、`--severity` - 按严重性级别过滤日志。支持以下值（从低到高）：

    * `D` - Debug
    * `I` - Info
    * `W` - Warning
    * `E` - Error
    * `F` - Fatal

* `-o`、`--output` - 以文本（默认）或 JSON 格式显示日志信息。
* `-i`、`--opid` - 按操作 ID 过滤日志

## 示例

以下是一些过滤日志的示例：

**显示所有备份的日志**

```bash
pbm logs --event=backup
```

**显示特定备份 2020-10-15T17:42:54Z 的日志的最后 100 行**

```bash
pbm logs --tail=100 --event=backup/2020-10-15T17:42:54Z
```

**仅包含来自特定副本集的错误**

```bash
pbm logs -n rs1 -s E
```

输出包括指定严重性类型的日志消息和所有更高级别。因此，当指定 `ERROR` 时，输出中会显示 `ERROR` 和 `FATAL` 消息。

## 实现细节

`pbm-agents` 将日志信息写入 [PBM 控制集合](../reference/glossary.md#pbm-control-collections) 中的 `pbmLog` 集合。每个 `pbm-agent` 还将日志信息写入 `stderr`，以便在集群或副本集中没有健康的 `mongod` 节点时可以检索它。有关如何查看单个 `pbm-agent` 日志，请参阅[如何查看 pbm-agent 日志](../install/start-pbm-agent.md#how-to-see-the-pbm-agent-log)。

!!! note

    来自 `pbmLog` 集合的日志信息以 UTC 时区显示，来自 stderr 的日志信息以服务器的时区显示。
