# 基于快照的物理备份

!!! admonition "版本添加：[2.2.0](../release-notes/2.2.0.md)"

## 注意事项 

1. 仅支持完整备份
2. 仅在您在环境中运行 Percona Server for MongoDB 时可用，因为 PBM 使用 [`$backupCursor 和 $backupCursorExtended 聚合阶段` :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/backup-cursor.html)。 

虽然物理备份是数据目录的物理副本，但快照是存储数据文件的磁盘或卷的时间点副本。从快照恢复要快得多，并允许几乎立即访问数据，但是数据库在恢复期间不可用。基于快照的备份对于拥有 TB 级数据的大型数据集的所有者特别有用。然而，快照不保证分片集群中的完全数据一致性。

这就是 Percona Backup for MongoDB 的用武之地。它提供了进行基于快照的物理备份和恢复的接口，并确保数据一致性。因此，数据库所有者受益于提高的性能和减少的停机时间，并确保他们的数据保持一致。

基于快照的物理备份/恢复流程包括三个不同的阶段：

* 准备数据库 — 由 PBM 完成
* 复制文件 — 由用户/客户端应用程序完成
* 完成备份/恢复 — 由 PBM 完成。 

这是基于快照的备份的第一阶段，您可以手动进行。自动化的基于快照的备份计划在将来提供。

[创建备份](../usage/backup-external.md){.md-button}
[从备份恢复](../usage/restore-external.md){.md-button}


