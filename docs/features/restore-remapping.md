# 恢复到不同名称的集群/副本集

从版本 [1.8.0](../release-notes/1.8.0.md) 开始，您可以将[逻辑备份](../features/logical.md) 恢复到具有相同或更多分片数量且这些分片具有不同副本集名称的新环境。 
从版本 [2.2.0](../release-notes/2.2.0.md) 开始，您可以恢复具有[自定义分片名称](https://www.mongodb.com/docs/manual/reference/command/addShard/#mongodb-dbcommand-dbcmd.addShard) 的环境。 

从版本 [2.2.0](../release-notes/2.2.0.md) 开始，您可以将[物理](../features/physical.md) 和[增量物理](../features/incremental-backup.md) 备份恢复到具有不同副本集名称的新环境。请注意，**分片数量必须与**创建备份的环境中的分片数量相同。

要将数据恢复到具有不同副本集名称的环境，请配置目标环境和源环境之间的名称映射。您可以为 `pbm` CLI 设置 `PBM_REPLSET_REMAPPING` 环境变量，或为 PBM 命令使用 `--replset-remapping` 标志。映射格式是 `<rsTarget>=<rsSource>`。

!!! important

    为集群中的所有分片配置副本集名称映射。否则，Percona Backup for MongoDB 会尝试将未指定的分片恢复到具有相同名称的目标分片。如果没有具有此名称的分片或它已映射到另一个源分片，则恢复失败。

配置副本集名称映射：


=== ":material-application-variable-outline: 在 shell 中为 `pbm` CLI 使用环境变量"

    ```bash
    export PBM_REPLSET_REMAPPING="rsTarget1=rsSource1,rsTarget2=rsSource2"
    ``` 

    假设您的源副本集是 `rsA` 和 `rsB`，而目标副本集是 `rsX` 和 `rsY`。那么导出环境变量的命令如下：

    ```bash
    export PBM_REPLSET_REMAPPING="rsX=rsA,rsY=rsB"
    ``` 

=== ":material-console: 使用命令行"

    ```bash
    pbm restore <timestamp> --replset-remapping="rsTarget1=rsSource1,rsTarget2=rsSource2"
    ```

    假设您的源副本集是 `rsA` 和 `rsB`，而目标副本集是 `rsX` 和 `rsY`。那么运行恢复的命令如下：

    ```bash
    pbm restore <timestamp> --replset-remapping="rsX=rsA,rsY=rsB"
    ```

    `--replset-remapping` 标志可用于以下命令：`pbm restore`、`pbm list`、`pbm status`、`pbm oplog-replay`。 

!!! note 

    恢复完成后，在新环境上执行[恢复后步骤](../usage/restore.md#post-restore-steps)。

这种将数据恢复到具有不同副本集名称和分片数量的集群的能力扩展了兼容恢复的环境集。
