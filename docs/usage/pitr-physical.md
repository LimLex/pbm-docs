# 从物理备份进行时间点恢复

--8<-- "pitr-preparation.md"

## 步骤

!!! admonition "版本添加：[2.2.0](../release-notes/2.2.0.md)"

您可以从完整或增量物理备份以与从逻辑备份相同的自动化方式恢复数据库。Percona Backup for MongoDB 恢复备份快照并自动在其上重放 oplog 事件直到指定时间，保证数据一致性。这有助于您防止灾难期间的数据丢失，并在管理备份和恢复时提供相同的用户体验。    

要从物理备份恢复数据库，请为 [`pbm restore`](../reference/pbm-commands.md#pbm-restore) 命令指定时间：    

```bash
pbm restore --time <timestamp> 
```    

Percona Backup for MongoDB 识别它是完整还是增量备份，并从中将数据库恢复到指定时间。     

您可以使用 `pbm describe-restore` 命令[跟踪恢复进度](restore-progress.md)。不要运行任何其他命令，因为它们可能会中断恢复流程并导致数据库问题。


!!! note    

    对于早于 2.3.0 的 PBM 版本，时间点恢复的命令如下：
        
    ```bash
    pbm restore --base-snapshot=<backup_name> --time <timestamp>
    ```

    `--base-snapshot` 标志是必需的。否则，PBM 会查找逻辑备份，即使没有逻辑备份或存在更新的物理备份。    

### 恢复后步骤

时间点恢复完成后，执行这些恢复后步骤：   

1. 重启所有 `mongod` 节点。    

2. 重启所有 `pbm-agents`。    

3. 与存储重新同步备份列表：    

    ```bash
    pbm config --force-resync
    ```    

4. 启动平衡器并启动 `mongos` 节点。    
5. 创建新备份作为未来恢复的新基础。    

6. [启用时间点例程](../features/point-in-time-recovery.md#enable-point-in-time-recovery) 以恢复保存 oplog 切片。    

??? admonition "Percona Backup for MongoDB 版本 2.1.0 及更早版本"

    对于 Percona Backup for MongoDB 版本 2.1.0 及更早版本，时间点恢复包括以下步骤：    

    * 从物理备份快照恢复。
    * 在此快照上手动重放 oplog 事件直到特定时间戳。    

    有关如何在备份上重放 oplog 事件，请参阅[物理备份的 Oplog 重放](oplog-replay.md#oplog-replay-for-physical-backups)。

### 实现细节

1. 由于物理恢复逻辑和流程，当 Percona Server for MongoDB 关闭时，PBM 在每个分片的主节点上重放 oplog 事件。数据库启动后，剩余节点在初始同步期间接收数据。
2. 对具有分片集合的部署进行时间点恢复时，PBM 仅写入现有数据，不支持创建新集合。因此，无论何时创建新的分片集合，请为其创建新备份以包含在其中。


## 有用的链接

* [恢复备份](restore.md)
* [从任意开始时间重放 oplog](oplog-replay.md)


