# 增量物理备份

!!! admonition "版本添加：[2.0.3](../release-notes/2.0.3.md)"

## 注意事项

* :warning: 使用 Percona Backup for MongoDB [2.1.0](../release-notes/2.1.0.md) 之前版本创建的增量备份与 Percona Backup for MongoDB 2.1.0 的恢复不兼容。这是因为现在存储在备份中的元数据文件集已更改。使用早期 PBM 版本创建的备份中缺少这些文件，但使用 PBM 2.1.0 进行恢复时需要这些文件。

    我们建议在升级到 Percona Backup for MongoDB 2.1.0 后创建新的增量基础备份并从中开始增量备份链

* 增量备份实现基于 [`$backupCursor` :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/backup-cursor.html) 聚合阶段，该阶段仅在 Percona Server for MongoDB 中可用。因此，您必须在部署中运行 Percona Server for MongoDB 才能使用增量物理备份。
* 增量备份支持从以下版本开始的 Percona Server for MongoDB：[4.2.24-24 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/4.2/release_notes/4.2.24-24.html)、[4.4.18 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/4.4/release_notes/4.4.18-18.html)、[5.0.14-12 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/5.0/release_notes/5.0.14-12.html)、[6.0.3-2 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/6.0/release_notes/6.0.3-2.html) 及更高版本。 
* 由于打开 `$backupCursor` 时 [WiredTger 在日志结构化合并 (LSM) 树中的限制 :octicons-link-external-16:](https://source.wiredtiger.com/develop/backup.html#backup_incremental-block) 行为，如果在数据库中配置了 LSM 树，则增量备份不可用。

大型数据集的所有者可能需要频繁备份数据。每次都进行完整物理备份在存储空间方面成本高昂。增量物理备份在这种情况下很有用，使您能够优化备份策略并降低存储成本。

在增量备份期间，Percona Backup for MongoDB 仅保存上次备份后更改的数据。这导致更快的备份/恢复性能。由于增量备份的大小与完整备份相比更小，您还可以节省云部署中的存储和传输成本。

```mermaid
graph LR
  A[完整物理 ] --> B([增量 1 ]);
  B --> C([增量 2 ]);
  C --> |.....| D([增量 n ]);
```

## 实现细节

1. Percona Backup for MongoDB 仅在创建增量基础备份的节点上跟踪备份历史。这意味着后续增量备份必须始终在该节点上运行。为了实现这一点，Percona Backup for MongoDB 尝试在同一节点上安排备份。

    如果具有增量基础备份的节点关闭或不可用，您必须在另一个节点上重新开始增量备份链。

2. 调整备份的节点优先级会干扰上述默认行为。如果您仅为优先级列出节点的子集，剩余节点会收到默认优先级 1，增量备份可能不是从创建基础备份的节点获取的。 

    为避免这种情况，为您希望从中进行增量备份的节点设置最高优先级。这指示 Percona Backup for MongoDB 从同一节点开始备份，并将保留备份链。 

    对于分片集群，为每个副本集定义最高优先级节点。 

[创建备份](../usage/backup-incremental.md){ .md-button .md-button }
[恢复备份](../usage/restore-incremental.md){ .md-button .md-button }

