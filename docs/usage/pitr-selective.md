# 从选定数据库和集合进行时间点恢复

!!! important

    仅支持副本集。
    适用于逻辑备份。

1. 开始之前：

    1. 阅读[选择性备份和恢复的已知限制](../features/known-limitations.md#selective-backups-and-restores)。
    2. 检查您是否[已创建完整备份](backup-selective.md)，因为它作为时间点恢复的基础。任何选择性备份都会被忽略。

2. 要将所需数据库或集合恢复到某个时间点，请按如下方式运行 ``pbm restore`` 命令：

    ```bash
    pbm restore --base-snapshot <backup_name> --time <timestamp> \
    --ns <db.collection>
    ```

    您可以将选择性备份指定为时间点恢复的基础快照。在这种情况下，Percona Backup for MongoDB 仅将此备份中包含的命名空间（数据库或集合）恢复到指定时间。    

    或者，您可以使用完整备份快照并从中将所需的命名空间（数据库或集合）恢复到特定时间。将它们指定为 `pbm restore` 命令的逗号分隔列表。    

    启动时间点恢复时，Percona Backup for MongoDB 使用提供的基础快照，恢复指定的命名空间，并在其上重放 oplog 直到指定时间。如果未提供基础快照，Percona Backup for MongoDB 使用最新的完整备份快照。
