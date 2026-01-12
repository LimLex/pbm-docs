# 从物理备份恢复

--8<-- "restore-intro.md"

## 注意事项

1. 禁用时间点恢复。恢复和时间点恢复 oplog 切片是不兼容的操作，不能同时运行。 

    ```bash
    pbm config --set pitr.enabled=false
    ```

2. 备份和恢复数据的 Percona Server for MongoDB 版本必须在同一主要版本内。
3. 确保集群中的所有节点都健康（即报告 PRIMARY 或 SECONDARY）。每个 `pbm-agent` 需要能够连接到其本地节点并运行查询以执行恢复。
4. 对于 PBM 2.1.0 之前的版本，不支持具有仲裁节点的部署的物理恢复。

## 开始之前

1. 关闭所有 `mongos` 节点，因为恢复进行时数据库将不可用。 
2. 关闭所有 `pmm-agent` 和其他可以对数据库执行写入操作的客户端。这是确保恢复后数据一致性所必需的。
3. 手动停止仲裁节点，因为这些节点上没有 `pbm-agent` 来自动执行此操作。
4. 检查 `pbm-agent.service` 和 `mongod.service` 进程的 `systemctl` 重启策略未设置为 `always` 或 `on-success`：

    ```bash
    sudo systemctl show mongod.service | grep Restart
    sudo systemctl show pbm-agent.service | grep Restart
    ```

    ??? example "示例输出"

        ```{.text .no-copy}
        Restart=no
        RestartUSec=100ms
        ```
    
    在物理恢复期间，数据库不得自动重启，因为这由 `pbm-agent` 控制。
   
## 恢复数据库

1. 列出备份 

    ```bash
    pbm list
    ```

2. 进行恢复

    ```bash
    pbm restore <backup_name>
    ```

    在物理恢复期间，`pbm-agent` 进程停止 `mongod` 节点，清理数据目录，并将数据从存储复制到每个节点。在此过程中，数据库会重启几次。 

    您可以使用 `pbm describe-restore` 命令[跟踪恢复进度](restore-progress.md)。不要运行任何其他命令，因为它们可能会中断恢复流程并导致数据库问题。

    当恢复在所有节点上成功时，恢复状态为 `Done`。如果它在某些节点上失败，状态为 `partlyDone`，但您仍然可以启动集群。失败的节点将通过初始同步接收数据。对于任一状态，请继续执行[恢复后步骤](#post-restore-steps)。在[部分完成的物理恢复](../troubleshoot/restore-partial.md) 章节中了解有关部分完成恢复的更多信息。 

### 恢复后步骤

恢复完成后，执行以下操作：

1. 删除任何仲裁节点上数据目录的内容
2. 重启所有 `mongod` 节点

    !!! note

        集群重启后，您可能会在 `mongod` 日志中看到以下消息：

        ```{.text .no-copy}
        "s":"I",  "c":"CONTROL",  "id":20712,   "ctx":"LogicalSessionCacheReap","msg":"Sessions collection is not set up; waiting until next sessions reap interval","attr":{"error":"NamespaceNotFound: config.system.sessions does not exist"}}}}
        ```

        这是数据库启动时定期检查的预期行为。在恢复期间，`config.system.sessions` 集合被删除，但 Percona Server for MongoDB 最终会重新创建它。这是正常过程。您无需执行任何操作。

3. 重启所有 `pbm-agents`

4. 运行以下命令以与存储重新同步备份列表：

    ```bash
    pbm config --force-resync -w
    ``` 

5. 启动平衡器并启动 `mongos` 节点。

6. 我们建议创建新备份作为未来恢复的新基础。
7. 如果需要，[启用时间点恢复](../features/point-in-time-recovery.md#enable-point-in-time-recovery)。
     

## 定义 `mongod` 二进制文件位置

!!! admonition "版本添加：[2.0.4](../release-notes/2.0.4.md)"

在物理恢复期间，Percona Backup for MongoDB 会多次重启数据库。默认情况下，它使用 `$PATH` 变量中 `mongod` 二进制文件的位置来访问数据库。如果您定义了 `mongod` 二进制文件的自定义路径，请让 Percona Backup for MongoDB 知道它： 

=== ":octicons-file-code-24: 配置文件"

    ```yaml
    restore:
        mongodLocation: /path/to/mongod
    ```

=== ":material-console: 命令行"

    ```bash
    pbm config --set restore.mongodLocation=/path/to/mongod/
    ```

如果您在集群/副本集的每个节点上有不同的 `mongod` 二进制文件路径，请使用 `mongodLocationMap` 选项为每个节点指定您的自定义路径。

```yaml
restore:
    mongodLocationMap:
       "node01:27017": /path/to/mongod
       "node03:27017": /another/path/to/mongod
```

在 Docker 中运行时，将 Percona Backup for MongoDB 文件与 MongoDB 二进制文件一起包含。有关更多信息，请参阅[在 Docker 容器中运行 Percona Backup for MongoDB](https://docs.percona.com/percona-backup-mongodb/install/docker.html)。

### 并行数据下载

!!! admonition "版本添加：[2.1.0](../release-notes/2.1.0.md)"

Percona Backup for MongoDB 在物理恢复期间并发从 S3 存储下载数据块。在 *Andrew Pogrebnoi* 的 [在 PBM 中加速 MongoDB 恢复](https://www.percona.com/blog/speeding-up-database-restores-in-pbm) 博客文章中了解更多关于基准测试结果的信息。

工作原理如下：

在物理恢复期间，Percona Backup for MongoDB 启动工作线程。默认情况下，工作线程数量等于 CPU 核心数。每个工作线程都分配了一个内存缓冲区。缓冲区被分割为数据块大小的跨度。工作线程获取跨度以下载数据块并将其存储到缓冲区中。当缓冲区已满时，工作线程等待空闲跨度以继续下载。   

您可以根据硬件资源和数据库负载微调并行下载。编辑 PBM 配置文件并指定以下设置：

```yaml
restore:
   numDownloadWorkers: <int>
   maxDownloadBufferMb: <int>
   downloadChunkMb: 32
```

* `numDownloadWorkers` - 从存储下载数据的工作线程数。默认情况下，它等于 CPU 核心数
* `maxDownloadBufferMb` - 用于存储下载的数据块以进行解压缩和排序的内存缓冲区的最大大小。计算为 `numDownloadWorkers * downloadChunkMb * 16`
* `downloadChunkMb` 是要下载的数据块的大小（默认情况下，32 MB）


## 下一步

[时间点恢复](../usage/pitr-physical.md){.md-button}

## 有用的链接 

* [查看恢复进度](../usage/restore-progress.md)
* [恢复到新环境](../features/restore-new-env.md)
* [恢复到不同名称的集群](../features/restore-remapping.md)




  



