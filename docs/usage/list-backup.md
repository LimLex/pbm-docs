# 列出备份

使用 `pbm list` 命令查看所有已完成的备份。 

```bash
pbm list
```

输出提供以下信息：

* 备份名称 
* 备份类型：[逻辑](../features/logical.md)、[物理](../features/physical.md)、[选择性](../features/selective-backup.md)、[增量](../features/incremental-backup.md)。从版本 1.7.0 开始可用 
* 恢复后分片集群/非共享副本集将返回到的时间。从版本 1.4.0 开始可用
* 如果启用了[时间点恢复](../features/point-in-time-recovery.md)，其状态和恢复的有效时间范围

??? example "示例输出"

    ```{.text .no-copy}
    Backup snapshots:
      2025-03-10T10:44:52Z <logical> [restore_to_time: 2025-03-10T10:44:56]
      2025-03-10T10:49:20Z <physical> [restore_to_time: 2025-03-10T10:49:23]
      2025-03-10T10:50:22Z <incremental> [restore_to_time: 2025-03-10T10:50:25]
      2025-03-10T10:51:02Z <incremental> [restore_to_time: 2025-03-10T10:51:04]
      2025-03-10T10:57:47Z <incremental> [restore_to_time: 2025-03-10T10:57:49]
      2025-03-10T11:04:25Z <incremental> [restore_to_time: 2025-03-10T11:04:27]
      2025-03-10T11:05:03Z <logical, selective> [restore_to_time: 2025-03-10T11:05:07]
    ```

## 恢复到时间

在逻辑备份中，完成时间几乎与备份完成时间一致。为了定义完成时间，Percona Backup for MongoDB 等待备份快照在所有集群节点上完成。然后它捕获从备份开始时间到该时间的 oplog。

在物理备份中，完成时间仅在备份开始时间后几秒。通过保持 `$backupCursor` 打开，保证检查点数据在备份期间不会更改，Percona Backup for MongoDB 可以提前定义完成时间。


## 有用的链接

* [查看备份的详细信息](describe-backup.md)
* [恢复到时间点](pitr-physical.md)
