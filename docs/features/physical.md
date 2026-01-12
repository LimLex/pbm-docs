# 物理备份和恢复

!!! admonition "版本添加：[1.7.0](../release-notes/1.7.0.md)" 

??? admonition "实现历史"

    下表列出了物理备份实现中的更改以及引入这些更改的版本：

    |版本 | 描述 |
    |--------|-------------|
    | [2.0.0](../release-notes/2.0.0.md)   | 物理备份和恢复，使用静态数据加密的物理恢复 |
    | [2.3.0](../release-notes/2.3.0.md)   | 混合部署中的物理备份 |
    | [2.10.0](../release-notes/2.10.0.md) | 使用回退目录的物理恢复 | 

## 可用性和系统要求

*  Percona Server for MongoDB 从版本 4.2.15-16、4.4.6-8、5.0 及更高版本开始。 
* WiredTiger 用作 Percona Server for MongoDB 中的存储引擎，因为物理备份严重依赖 WiredTiger [`$backupCursor` :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/6.0/backup-cursor.html) 功能。

!!! warning 

    在备份游标打开期间，可以创建数据库检查点，但不能删除检查点。这可能导致文件显著增长。
    
!!! admonition "另请参阅"

    Percona 博客

    * [Percona Backup for MongoDB 中的物理备份支持 :octicons-link-external-16:](https://www.percona.com/blog/physical-backup-support-in-percona-backup-for-mongodb/)
    * [Percona Server for MongoDB 中的 $backupCursorExtend :octicons-link-external-16:](https://www.percona.com/blog/2021/06/07/experimental-feature-backupcursorextend-in-percona-server-for-mongodb/)

物理备份是将物理文件从 Percona Server for MongoDB `dbPath` 数据目录复制到远程备份存储。这些文件包括数据文件、日志、索引文件等。从版本 2.0.0 开始，Percona Backup for MongoDB 还将 WiredTiger 存储选项复制到备份的元数据中。 

物理恢复是相反的过程：`pbm-agents` 关闭 `mongod` 节点，清理 `dbPath` 数据目录，并将物理文件从存储复制到其中。 

以下图表显示了物理恢复流程：

![image](../_images/pbm-phys-restore-shard.png)

在恢复期间，``pbm-agents`` 使用从备份元数据中检索的 WiredTiger 存储选项临时启动 ``mongod`` 节点。这些启动的日志保存到 ``dbPath`` 内的 ``pbm.restore.log`` 文件中。恢复成功后，此文件将被删除。但是，如果恢复失败，它仍保留用于调试。 

在物理备份和恢复期间，``pbm-agents`` 不会从数据库导出/导入数据。这显著减少了与逻辑备份相比的备份/恢复时间，是大型（多 TB）数据库推荐的备份方法。

| 优点                     | 缺点                   |
| ------------------------------ | ------------------------------- |
|- 更快的备份和恢复速度 <br> - 推荐用于大型、多 TB 数据集 <br> - 无数据库开销 | - 由于数据碎片以及在适当数据结构中保持数据和索引的额外成本，备份大小大于逻辑备份 <br> - 恢复后需要额外的手动操作 <br> - 时间点恢复需要手动操作 | 分片集群和非分片副本集 |

[创建备份](../usage/backup-physical.md){ .md-button }
[恢复备份](../usage/restore-physical.md){ .md-button }

## 混合部署中的物理备份

!!! admonition "版本添加：[2.3.0](../release-notes/2.3.0.md)"

您可以在环境中运行 MongoDB Community / Enterprise Edition 节点和 Percona Server for MongoDB (PSMDB) 节点，例如，在迁移到或评估 PSMDB 时。 

您可以使用 PBM 在此类混合部署中进行物理、增量或基于快照的备份。这使您无需为备份重新配置部署，并保持迁移和备份策略完整。

物理、增量和基于快照的备份只能从 PSMDB 节点进行，因为它们的实现基于 [`$backupCursorExtend` :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/backup-cursor.html) 功能。当需要创建备份时，PBM 搜索 PSMDB 节点并从中创建备份。PSMDB 节点不能是仲裁节点或延迟节点。 

如果有超过 2 个节点适合备份，PBM 选择具有更高[优先级](../usage/backup-priority.md)的节点。请注意，如果您至少为一个节点覆盖优先级，PBM 会为剩余节点分配优先级 `1.0` 并使用新的优先级列表。 

考虑以下[增量备份](incremental-backup.md)流程：
默认情况下，PBM 在创建后续备份时选择创建增量基础备份的节点。PBM 为此节点分配优先级 `3.0`，确保它在列表中排在第一位。如果您更改节点优先级，请创建新的增量基础备份以确保数据连续性。

混合部署中的物理恢复没有限制，除了备份和源集群中的版本必须匹配。

## 使用静态数据加密的物理恢复

!!! admonition "版本添加：[2.0.0](../release-notes/2.0.0.md)"

您可以备份和恢复静态加密的数据。因此，您可以确保数据安全，并可以遵守 GDPR、HIPAA、PCI DSS 或 PHI 等安全要求。

在备份期间，Percona Backup for MongoDB 将加密设置存储在备份元数据中。您可以使用 [`pbm describe-backup`](../reference/pbm-commands.md#pbm-describe-backup) 命令验证它们。请注意，加密密钥不会存储也不会显示为备份的一部分。

!!! important

    确保您知道使用了哪个主加密密钥并妥善保管，因为恢复需要此密钥。

!!! note

    从 [Percona Server for MongoDB 版本 4.4.19-19 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/4.4/release_notes/4.4.19-19.html)、[5.0.15-13 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/5.0/release_notes/5.0.15-13.html)、[6.0.5-4 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/6.0/release_notes/6.0.5-4.html) 及更高版本开始，使用 HashiCorp Vault 加密的静态数据的主密钥轮换已改进为在整个部署中的每个服务器上使用相同的密钥路径。对于使用 HashiCorpVault 的 Percona Server for MongoDB **之前**版本 4.4.19-19、5.0.15-13、6.0.5-4 和 PBM 2.0.5 及更早版本的恢复，请参阅[使用 HashiCorpVault 恢复 Percona Server for MongoDB **之前**版本 4.4.19-19、5.0.15-13、6.0.5-4](#restore-for-percona-server-for-mongodb-before-4419-19-5015-13-605-4-using-hashicorpvault) 部分。

要从备份恢复加密数据，请在目标集群或副本集的所有节点上配置相同的静态数据加密设置，以匹配创建备份的原始集群的设置

在恢复期间，Percona Backup for MongoDB 使用相同的主密钥将所有节点上的数据恢复。我们建议之后轮换主加密密钥以增强安全性。 

要了解有关主密钥轮换的更多信息，请参阅以下文档：

* [HashiCorp Vault 服务器中的主密钥轮换 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/6.0/vault.html#key-rotation)
* [KMIP 主密钥轮换 :octicons-link-external-16:](https://www.mongodb.com/docs/manual/tutorial/rotate-encryption-key/#kmip-master-key-rotation)

### 使用 HashiCorpVault 恢复 Percona Server for MongoDB **之前**版本 4.4.19-19、5.0.15-13、6.0.5-4

在使用 Vault 服务器进行静态数据加密的 Percona Server for MongoDB **之前**版本 4.4.19-19、5.0.15-13、6.0.5-4 中，不支持使用相同密钥对 2+ 节点进行主密钥轮换。如果您运行这些版本的 Percona Server for MongoDB 和 PBM 2.1.0 之前的版本，请考虑使用 PBM 在每个副本集的一个节点上恢复数据的场景。剩余节点在初始同步期间接收数据。 

在目标集群或副本集的每个分片的一个节点上配置静态数据加密。

在恢复期间，Percona Backup for MongoDB 在加密密钥与备份数据加密时使用的密钥匹配的节点上恢复数据。其他节点不会恢复，因此恢复状态为 "partlyDone"。您可以启动此节点并初始化副本集。剩余节点从恢复节点通过初始同步接收数据。  

## 使用回退目录的物理恢复

!!! admonition "版本添加：[2.10.0](../release-notes/2.10.0.md)"

在物理恢复阶段可能会发生意外错误，例如损坏的备份数据文件、访问备份存储时的网络问题或意外的 PBM 故障。发生这种情况时，`dbPath` 中的文件可能处于不一致状态，受影响的 `mongod` 实例无法重启。结果，集群中的副本集或分片变得无法运行。PBM 也变得无法运行，因为它依赖 MongoDB 作为通信通道和元数据存储。 

为了防止这种糟糕的情况，您可以配置 PBM 使用回退目录，并在物理恢复期间发生错误时将集群恢复到原始状态。PBM 在恢复开始时将 `dbPath` 内容复制到回退目录。然后恢复按常规流程进行。

如果恢复成功，PBM 会删除回退目录及其内容。 

如果 PBM 检测到集群处于错误状态，它会自动触发回退过程。PBM 清理从 `dbPath` 上传的备份文件，并将文件从回退目录移动到那里。这样，集群返回到恢复前的状态并可以运行。然后您可以重试恢复、尝试另一个备份或以其他方式维护它。

!!! warning 

    请注意，此功能需要权衡：您必须在每个 `mongod` 实例上有足够的磁盘空间来将 `dbPath` 的内容复制到回退目录。因此，默认情况下禁用回退目录使用。有关磁盘空间要求的更多信息，请参阅[磁盘空间评估](#disk-space-evaluation) 部分。

### 磁盘空间评估

每个 `mongod` 节点必须有足够的可用空间，以便 PBM 将 `dbPath` 的内容复制到回退目录。此外，文件复制后，至少 15% 的总磁盘容量必须保持空闲以确保操作稳定性。

在启动恢复之前，PBM 对每个 `mongod` 实例执行全面的磁盘空间评估。这包括：

* 总磁盘大小
* 已用和可用磁盘空间
* PBM 操作可用的估计大小。计算为 `总大小的 85% - 已用空间`
* 备份大小。备份大小必须小于 PBM 操作可用的估计大小。PBM 使用未压缩的备份大小进行评估。此信息存储在备份元数据中，可在 `pbm describe-backup` 命令输出中获得。

请注意，时间点恢复 oplog 块不会被评估。剩余的可用空间被认为足以让 PBM 在恢复期间成功重放它们。

为了说明此评估，请考虑以下示例：

* 磁盘总计：10 GB
* 已用：6 GB
* 可用空间：4 GB
* PBM 使用可用：10GB * 85% - 6GB = 2.5 GB

备份大小必须小于 2.5 GB 才能使用回退目录继续恢复。

PBM 详细记录此评估。您可以使用 `pbm logs` 命令查看它。

如果根据此计算，集群中即使一个节点缺乏足够的磁盘空间，PBM 也会中止恢复过程。  

### 配置

要配置使用回退目录的物理恢复，请使用 PBM 配置文件或命令行：

=== ":octicons-file-code-24: 配置文件"

    在 PBM 配置文件中指定以下选项：

    ```yaml
    restore:
      fallbackEnabled: true
    ```
    
=== ":material-console: 命令行"

     您可以直接使用 `--fallback-enabled` 标志启动带回退目录的恢复：

     ```bash
     pbm restore --time <time> --fallback-enabled=true
     ```


恢复可能在大多数节点上成功，但可能在少数节点上失败，导致 "partlyDone" 状态。您可以配置 PBM 如何处理此类部分恢复：

=== ":octicons-file-code-24: 配置文件"

    ```yaml
    restore:
      allowPartlyDone: true
    ``` 

=== ":material-console: 命令行"

    ```bash
    pbm restore --time <time> --fallback-enabled=true --allow-partly-done=true
    ```
       
如果您允许部分恢复（默认值），PBM 会完成恢复。一旦集群启动并运行，失败的节点会通过初始同步从其他成员接收必要的数据。 

如果您拒绝部分恢复，PBM 会将集群视为不健康并回退到原始状态。在这种情况下，您必须将 `restore.fallbackEnabled` 选项设置为 `true` 或使用 `--fallback-enabled` 标志运行 `pbm restore` 命令。否则，恢复不会启动。

### 实现细节

1. 回退目录的使用支持副本集和分片集群部署。 
2. 回退目录的使用支持完整物理和物理增量备份
3. 您必须在每个 `mongod` 节点上有足够的可用空间，以便 PBM 将 `dbPath` 内容复制到回退目录。文件复制后，至少 15% 的总磁盘大小必须保持空闲以确保操作稳定性。此可用空间也被认为足以在时间点恢复期间重放 oplog。
4. 对于增量备份，所有增量都包含在备份大小计算中。
5. 您只能使用回退目录恢复使用 PBM 版本 2.10.0 创建的备份。对于使用早期 PBM 版本创建的备份，PBM 没有未压缩的备份大小，无法评估回退目录的磁盘空间。因此，PBM 自动禁用 `fallbackEnabled` 设置并记录此操作。 
