# 创建物理备份

--8<-- "prepare-backup.md"

## 步骤

在*物理*备份期间，Percona Backup for MongoDB 将 `dbpath` 目录的内容（数据和元数据文件、索引、日志和日志文件）从每个分片和配置服务器副本集复制到备份存储。 

要启动备份，请运行以下命令：

```bash
pbm backup --type=physical
```
     
!!! warning 

    在备份游标打开期间，可以创建数据库检查点，但不能删除检查点。这可能导致文件显著增长。
    
从 [2.4.0](../release-notes/2.4.0.md) 开始，如果启用了[时间点恢复 oplog 切片](../features/point-in-time-recovery.md#oplog-slicing)，PBM 不会停止它，而是并行运行它。这确保如果创建备份快照需要太长时间（例如数小时），可以[时间点恢复](pitr-tutorial.md) 到任何时间戳。

## 下一步

[列出备份](../usage/list-backup.md){.md-button}
[进行恢复](restore-physical.md){.md-button}
[进行时间点恢复](pitr-physical.md){.md-button}

## 有用的链接

* [备份和恢复类型](../features/backup-types.md)
* [安排备份](../usage/schedule-backup.md)

