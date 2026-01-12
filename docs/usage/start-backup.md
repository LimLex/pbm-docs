# 创建逻辑备份

--8<-- "prepare-backup.md"

## 步骤

!!! warning

    不支持分片时间序列集合。如果您在部署中使用它们，将无法创建备份。 
 

要创建备份，请运行以下命令：

```bash
pbm backup --type=logical
```
     
逻辑备份是默认的，因此您可以省略 `--type` 标志。 

在*逻辑*备份期间，Percona Backup for MongoDB 将实际数据复制到备份存储。

从版本 2.0.0 开始，Percona Backup for MongoDB 以新的多文件格式存储数据，其中每个集合都有一个单独的文件。无论这是完整备份还是选择性备份，都会为所有命名空间存储 oplog。

多格式现在是默认的数据格式，因为它允许[选择性恢复](restore-selective.md)。但是请注意，您只能从使用早期版本的 Percona Backup for MongoDB 创建的备份进行完整恢复。


## 下一步

[列出备份](../usage/list-backup.md){.md-button}
[从逻辑备份恢复](restore.md){.md-button}
[执行时间点恢复](pitr-tutorial.md){.md-button}

## 有用的链接

* [跟踪备份进度](../troubleshoot/status.md#backup-progress-tracking)
* [安排备份](../usage/schedule-backup.md)

