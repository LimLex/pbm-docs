# 从任意开始时间重放 oplog

您可以在任何备份之上重放特定时期的 [oplog](../reference/glossary.md#oplog)：逻辑、物理、存储级别快照（如 [EBS 快照](../reference/glossary.md#ebs-snapshot)）。您可以在没有强制基础备份快照的情况下保存 oplog 切片。此行为由 [`pitr.oplogOnly`](../reference/pitr-options.md) 配置参数控制：

```yaml
pitr:
   oplogOnly: true
```

通过使用 [`pbm oplog-replay`](../reference/pbm-commands.md#pbm-oplog-replay) 命令在备份快照之上重放这些 oplog 切片，您可以手动将分片集群和非分片副本集从任何工具（不仅仅是 Percona Backup for MongoDB）创建的备份恢复到特定时间点。此外，您还可以减少创建冗余基础备份快照的时间、存储空间和管理工作。

!!! warning

    请谨慎使用 oplog 重放功能，仅在您确定要重放 oplog 的开始时间时使用。从任何备份恢复时，oplog 重放不保证数据一致性。但是，对于使用 Percona Backup for MongoDB 创建的备份，它不太容易出错。

## 指定 oplog 重放时间的方法

PBM 使用 MongoDB 的时间戳格式进行 oplog 重放，提供操作级别的分辨率。每个 oplog 操作由 `(epoch, ordinal)` 标识，其中 `epoch` 是 Unix 时间（以秒为单位），`ordinal` 区分同一秒内的多个操作。指定的操作始终包含在重放中。

您可以通过两种方式定义 oplog 重放停止点：

1. **通过 ISO 时间戳**：  
   将结束时间指定为 ISO 时间戳（例如，`2025-01-02T15:00:00`）。当您想要包含直到指定秒发生的所有操作时，请使用此方法。

2. **通过 MongoDB 时间戳元组**：  
   将停止点指定为 `epoch,ordinal`（例如，`1764576382,20`）。PBM 包括直到该确切操作的所有操作。当您需要精确控制在一秒内包含哪些特定操作时，请使用此方法。 

## 物理备份的 Oplog 重放

!!! note ""

    从版本 2.2.0 开始，在使用 Percona Backup for MongoDB 创建的物理备份之上进行 oplog 重放是[时间点恢复](pitr-physical.md) 的一部分自动完成的。 

本节介绍如何在使用 Percona Backup for MongoDB 版本 2.1.0 及更早版本创建的物理备份之上**手动**重放 oplog。

在您[恢复物理备份](restore.md) 后，执行以下操作：

1. 如果启用了时间点恢复，请停止它以释放锁。

2. 运行 `pbm status` 或 `pbm list` 命令以查找可用于重放的 oplog 块。

3. 运行 `pbm oplog-replay` 命令并指定 `--start` 和 `--end` 标志。请参阅[如何指定时间](#ways-to-specify-time-for-oplog-replay)。

    === "使用时间戳"

        ```bash
        pbm oplog-replay --start="{{year}}-01-02T15:00:00" --end="{{year}}-01-03T15:00:00"
        ```
    
    === "使用 `epoch,ordinal`"

        为了精确控制在一秒内包含哪些确切操作，请将 `--start` 和 `--end` 标志的值指定为 `epoch,ordinal` 元组。

        ```bash
        pbm oplog-replay --end "1764576382,100"
        ``` 

4. oplog 重放后，创建新备份并启用时间点恢复 oplog 切片。

## 存储级别快照的 Oplog 重放

创建备份时，Percona Backup for MongoDB 会停止时间点恢复。这样做是为了在恢复后保持数据一致性。

存储级别快照在启用时间点恢复的情况下保存。因此，从此类备份恢复数据库后，时间点恢复会自动启用并开始 oplog 切片。这些新的 oplog 切片可能与备份期间保存的现有 oplog 冲突。要在此类情况下重放 oplog，请在恢复后执行以下操作：


1. 禁用时间点恢复。
2. 删除可能已创建的 oplog 切片。
3. 从存储重新同步数据。
4. 运行 `pbm oplog-replay` 命令并指定带有时间戳的 `--start` 和 `--end` 标志。

    ```bash
    pbm oplog-replay --start="2022-01-02T15:00:00" --end="2022-01-03T15:00:00"
    ```

5. oplog 重放后，创建新备份并启用时间点恢复 oplog 切片。

[已知限制](../features/known-limitations.md#oplog-replay-from-arbitrary-start-time){.md-button}


