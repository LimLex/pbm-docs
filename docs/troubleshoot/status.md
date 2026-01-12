# Percona Backup for MongoDB 状态

!!! admonition "版本添加：[1.4.0](../release-notes/1.4.0.md)"

您可以使用 [`pbm status`](../reference/pbm-commands.md#pbm-status) 命令检查在 MongoDB 环境中运行的 Percona Backup for MongoDB 的状态。

```bash
pbm status
```

输出提供以下信息：

* 您的 MongoDB 部署和在其中运行的 `pbm-agents`：每个代理连接到哪个 `mongod` 节点、它运行的 Percona Backup for MongoDB 版本以及代理的状态

* 当前正在运行的备份/恢复（如果有）

* 存储在远程备份存储中的备份：备份名称、类型、完成时间、大小和状态（成功、进行中、失败）

* [时间点恢复](../features/point-in-time-recovery.md) 状态（启用或禁用）

* 时间点恢复的有效时间范围和数据大小

这简化了故障排除，因为所有信息都在一个地方提供。

??? example "示例输出"

    ```{.bash .no-copy}
    pbm status    

    Cluster:
    ========
    config:
      - config/localhost:27027 [P]: pbm-agent [v2.10.0] OK
      - config/localhost:27028 [S]: pbm-agent [v2.10.0] OK
      - config/localhost:27029 [S]: pbm-agent [v2.10.0] OK
    rs1:
      - rs1/localhost:27018 [P]: pbm-agent [v2.10.0] OK
      - rs1/localhost:27019 [S]: pbm-agent [v2.10.0] OK
      - rs1/localhost:27020 [S]: pbm-agent [v2.10.0] OK
    rs2:
      - rs2/localhost:28018 [P]: pbm-agent [v2.10.0] OK
      - rs2/localhost:28019 [S]: pbm-agent [v2.10.0] OK
      - rs2/localhost:28020 [S]: pbm-agent [v2.10.0] OK    

    PITR incremental backup:
    ========================
    Status [OFF]    

    Currently running:
    ==================
    (none)    

    Backups:
    ========
    S3 us-east-1 https://storage.googleapis.com/backup-test
       Snapshots:
        2025-06-03T09:55:47Z 0.00B <physical> ongoing [running: running / 2025-06-03T09:55:50]
        2025-03-16T10:36:52Z 491.98KB <physical> success [restore_to_time: 2025-03-16T10:37:13]
        2025-03-15T12:59:47Z 284.06KB <physical> success [restore_to_time: 2025-03-15T13:00:08]
        2025-03-11T16:23:55Z 284.82KB <physical> success [restore_to_time: 2025-03-11T16:24:16]
        2025-03-11T16:22:35Z 284.04KB <physical> success [restore_to_time: 2025-03-11T16:22:56]
        2025-03-11T16:21:15Z 283.36KB <physical> success [restore_to_time: 2025-03-11T16:21:36]
        2025-03-11T16:19:54Z 281.73KB <physical> success [restore_to_time: 2025-03-11T16:20:15]
        2025-03-11T16:19:00Z 281.73KB <physical> success [restore_to_time: 2025-03-11T16:19:21]
        2025-03-11T15:30:38Z 287.07KB <physical> success [restore_to_time: 2025-03-11T15:30:59]
      PITR chunks [1.10MB]:
        2025-03-16T10:37:13 - 2025-03-16T10:43:26 44.17KB
    ```

## `pbm-agent` 日志

!!! admonition "版本添加：[1.4.0](../release-notes/1.4.0.md)"

要排查特定事件或节点的问题，请使用 [`pbm logs`](../reference/pbm-commands.md#pbm-logs) 命令。它提供环境中所有 `pbm-agent` 进程的日志。 

`pbm logs` 有一组过滤器，用于细化特定事件的日志，如 `backup`、`restore`、`pitr` 或特定节点，并管理日志详细级别。例如，要查看具有 Debug 详细级别的特定备份的日志，请按如下方式运行 `pbm logs` 命令：

```bash
pbm logs --severity=D --event=backup/2020-10-15T17:42:54Z
```

要了解有关可用过滤器和使用示例的更多信息，请参阅[查看备份日志](../usage/logs.md)。

## 备份进度跟踪

如果您有大型逻辑备份，您可以在创建它的 `pbm-agent` 的日志中跟踪备份进度。每分钟追加一行，显示当前集合的已复制字节数与总大小。

启动备份：

```bash
pbm backup
```

检查备份进度：

1. 检查哪个 `pbm-agent` 创建备份：

    ```bash
    pbm logs
    ```
