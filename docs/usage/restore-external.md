# 从基于快照的备份恢复

--8<-- "restore-intro.md"

## 注意事项

1. 关闭所有 `mongos` 节点。如果您设置了数据库的自动重启，请禁用它。
2. 手动停止仲裁节点，因为这些节点上没有 `pbm-agent` 来自动执行此操作。
   
## 恢复数据库

### 从通过 PBM 创建的备份恢复

以下过程描述了从[通过 PBM 创建的备份](backup-external.md) 的恢复过程。也可以尝试从没有 PBM 创建的快照恢复（此功能是实验性的）。请参阅[从 PBM 外部创建的备份恢复](#restore-from-a-backup-made-outside-pbm)。

1. 要执行恢复，请运行以下命令：

    ```bash
    pbm restore --external 
    ```    

    Percona Backup for MongoDB 停止数据库，清理所有节点上的数据目录，提供恢复名称并提示您复制数据：    

    ```{.text .no-copy}
    Starting restore <restore_name> from '[external]'.................................................................................................................................Ready to copy data to the nodes data directory.
        After the copy is done, run: pbm restore-finish <restore_name> -c </path/to/pbm.conf.yaml>
        Check restore status with: pbm describe-restore <restore_name> -c </path/to/pbm.conf.yaml>
        No other pbm command is available while the restore is running!
    ``` 

2. 复制数据回来。虽然备份是从副本集的单个节点创建的，但对于恢复，您必须**将数据复制到集群中相应副本集的每个节点**。例如，给定 3 节点副本集 `rs1` 的备份，其中备份是从 `node3` 获取的，请将数据复制回 `rs1` 中的所有 3 个节点。

3. 将文件复制到节点后，使用以下命令完成恢复：    

    ```bash
    pbm restore-finish <restore_name> -c </path/to/pbm-conf.yaml>
    ```    

    在此阶段，Percona Backup for MongoDB 从备份读取元数据，准备集群/副本集启动的数据并确保其一致性。数据库恢复到元数据的 `restore_to_time` 中指定的时间戳。

    !!! note

        如果您使用文件系统作为远程备份存储，`pbm-agent` 和 `pbm` CLI 都必须对其具有相同的权限。为此，请以 `mongod` 用户身份运行 `pbm restore-finish` 命令：

        ```bash
        sudo -u mongod -s pbm restore-finish <restore_name> -c </path/to/pbm-conf.yaml> --mongodb-uri=MONGODB_URI
        ```

4. 可选。您可以通过运行 [`pbm describe-restore`](../reference/pbm-commands.md#pbm-describe-restore) 命令跟踪恢复进度。

#### 恢复后步骤 

恢复完成后，执行以下操作：

1. 启动所有 `mongod` 节点

2. 启动所有 `pbm-agents`

3. 运行以下命令以与存储重新同步备份列表：

    ```bash
    pbm config --force-resync
    ``` 

4. 启动平衡器并启动 `mongos` 节点。

5. 创建新备份作为未来恢复的新基础。 

### 从 PBM 外部创建的备份恢复

!!! Warning

    此功能是实验性的。
    
!!! important

    对于通过 PBM 创建的外部备份，PBM 对备份和目标集群执行兼容性检查。如果您恢复没有 PBM 创建的备份，它无法确保备份是正确且一致地创建的。因此，备份兼容性是您的责任。

要恢复没有 PBM 创建的外部备份，您需要为 `pbm restore` 命令指定以下内容：

* 源集群上创建备份的 `mongod` 节点的配置文件路径。这是 PBM 在恢复期间将使用的配置文件。它应该包含每个副本集名称的[存储选项 :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/configuration-options/#storage-options )，例如：

   ```yaml
   rs1:
       storage:
           directoryPerDB: true
   rs2:
       storage:
           directoryPerDB: true
   ```

   要恢复静态加密的数据，请确保源集群和目标集群上的静态数据加密设置相同。 

* 要恢复到的时间戳

要从备份恢复，请执行以下操作：

1. 启动恢复

    ```bash
    pbm restore --external -c </path/to/mongod.conf> --ts 
    ```

    如果源集群 `mongod.conf` 的路径未定义，PBM 会尝试从目标集群的 `mongod.conf` 检索所需的配置选项。    

    如果要恢复到的时间戳未定义，PBM 会在恢复期间查看实际数据，并定义所有分片上最新的公共集群时间。PBM 将数据库恢复到此时。

2. 接下来，复制数据文件。请注意，您必须将数据复制到**集群/副本集的每个数据承载节点**。

3. 通过运行以下命令完成恢复：

    ```bash
    pbm restore-finish <restore_name> -c </path/to/pbm.conf.yaml>
    ```    

    在此阶段，Percona Backup for MongoDB 准备集群/副本集启动的数据并确保其一致性。 

    !!! note

        如果您使用文件系统作为远程备份存储，`pbm-agent` 和 `pbm` CLI 都必须对其具有相同的权限。为此，请以 `mongod` 用户身份运行 `pbm restore-finish` 命令：

        ```bash
        sudo -u mongod -s pbm restore-finish <restore_name> -c </path/to/pbm-conf.yaml> --mongodb-uri=MONGODB_URI
        ```

4. 不要忘记完成[恢复后步骤](#post-restore-steps)。

## 有用的链接 

* [查看恢复进度](../usage/restore-progress.md)





  



