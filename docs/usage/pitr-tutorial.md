# 从逻辑备份进行时间点恢复

--8<-- "pitr-preparation.md"

## 步骤

运行 [`pbm restore`](../reference/pbm-commands.md#pbm-restore) 并指定有效范围内的时间戳：    

```bash
pbm restore --time="2022-12-14T14:27:04"
```    

您为恢复指定的时间戳必须在 `pbm list` 输出的 PITR 部分的时间范围内。Percona Backup for MongoDB 自动选择与指定时间戳相关的逻辑、物理和增量备份中最新的备份，并将其用作恢复的基础。    

为了说明此行为，让我们使用以下 `pbm list` 输出作为示例。     

```{.text .no-copy}
$ pbm list    

  2025-03-04T13:00:58Z [restore_to_time: 2025-03-04T13:01:23]
  2025-03-05T13:00:47Z [restore_to_time: 2025-03-05T13:01:11]
  2025-03-06T08:02:44Z [restore_to_time: 2025-03-06T08:03:09]
  2025-03-06T08:03:43Z [restore_to_time: 2025-03-06T08:04:08]
  2025-03-06T08:18:17Z [restore_to_time: 2025-03-06T08:18:41] 

PITR <off>:
  2025-03-04T13:01:24 - 2025-03-05T13:00:11
  2025-03-06T08:03:10 - 2025-03-06T08:18:29
  2025-03-06T08:18:42 - 2025-03-06T08:33:09
```    

对于时间戳 `2025-03-06T08:10:10`，备份快照 `2025-03-06T08:02:44Z [restore_to_time: 2025-03-06T08:03:09]` 用作恢复的基础，因为它是最新的。    

如果您[使用 `–-base-snapshot` 选项为恢复选择备份快照](#select-a-backup-snapshot-for-the-restore)，恢复的时间戳也必须晚于所选备份。    

!!! admonition "另请参阅"    

    [恢复备份](restore.md)    

### 恢复后步骤    

恢复操作会更改 oplog 事件的时间线。因此，在恢复时间戳之后和最后一次备份之前创建的所有 oplog 切片都变得无效。恢复完成后，执行以下操作：    

1. 创建新备份作为 oplog 更新的起点：    

    ```bash
    pbm backup
    ```    

2. 重新启用时间点恢复以恢复保存 oplog 切片：    

    ```bash
    pbm config --set pitr.enabled=true
    ```

## 为恢复选择备份快照

您可以使用任何备份快照（而不仅仅是最新的）将数据库恢复到特定时间点。使用 `--base-snapshot=<backup_name>` 标志运行 `pbm restore` 命令，在其中指定所需的备份快照。

要从任何备份快照恢复，Percona Backup for MongoDB 需要连续的 oplog。创建备份快照并重新启用时间点恢复后，它会复制与备份快照一起保存的 oplog，并从最新切片的结束时间到新起点创建 oplog 切片，从而使 oplog 连续。

## 有用的链接

* [恢复备份](restore.md)
* [从任意开始时间重放 oplog](oplog-replay.md)

