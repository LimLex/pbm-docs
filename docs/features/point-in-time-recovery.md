# 时间点恢复

!!! admonition "版本添加：[1.3.0](../release-notes/1.3.0.md)"

??? admonition "实现历史"

    下表列出了时间点恢复实现中的更改以及引入这些更改的版本：

    |版本 | 描述 |
    |--------|-------------|
    | [1.3.0](../release-notes/1.3.0.md)   | 时间点恢复的初始实现|
    | [1.6.0](../release-notes/1.6.0.md)   | 能够更改 oplog 跨度持续时间|
    | [1.7.0](../release-notes/1.7.0.md)   | 添加了对 oplog 切片的压缩|
    | [2.3.0](../release-notes/2.3.0.md)   | 支持任何类型的基础备份|
    | [2.4.0](../release-notes/2.4.0.md)   | 与备份并行进行 oplog 切片|
    |[2.6.0](../release-notes/2.6.0.md)    | 调整 oplog 切片的节点优先级|

时间点恢复是将数据库恢复到特定时间戳。这包括从备份快照恢复数据，并从 [oplog 切片](#oplog-slicing) 重放直到指定时间发生的所有事件。 

| 优点                     | 缺点                   |
| ------------------------------ | ------------------------------- |
| 帮助您防止灾难期间的数据丢失，例如数据库崩溃、意外数据删除或集合删除，以及意外更新多个字段而不是单个字段。 | 恢复需要更长时间，因为它需要您恢复备份，然后在其上重放 oplog 事件。|

## 启用时间点恢复

将 `pitr.enabled` 配置选项设置为 `true`。

=== ":octicons-file-code-24: 命令行"

     ```bash
     pbm config --set pitr.enabled=true
     ```

=== ":material-console: 配置文件"

     ```yaml
     pitr:
       enabled: true
     ```

`pbm-agent` 开始定期[保存连续的 oplog 切片](#oplog-slicing)。保存 oplog 切片的 `pbm-agent` 根据节点的优先级在节点中随机选择，无论是默认优先级还是[用户定义的优先级](#adjust-node-priority-for-oplog-slices)。 

您可以通过为 `mongod` 节点分配优先级值来管理 `pbm-agent` 选举。有关详细信息，请参阅[调整 oplog 切片的节点优先级](#adjust-node-priority-for-oplog-slices)。


[恢复到时间点](../usage/pitr-tutorial.md){ .md-button .md-button }

## Oplog 切片

要开始保存 [oplog 切片](../reference/glossary.md#oplog)，必须满足以下先决条件：

=== ":material-data-matrix: 逻辑备份"

    * 需要完整备份快照。从版本 [2.3.0](../release-notes/2.3.0.md) 开始，它可以是逻辑、物理或增量备份。确保[存在备份](../usage/list-backup.md)。请参阅[创建备份](../usage/start-backup.md) 指南以创建备份快照。
    * 时间点恢复例程已[启用](#enable-point-in-time-recovery)。 

=== ":material-database-refresh-outline: 物理备份"

    时间点恢复例程已[启用](#enable-point-in-time-recovery)。 
    

如果您刚刚启用时间点恢复，第一个切片需要 10 分钟才会出现在 [`pbm list`](../reference/pbm-commands.md#pbm-list) 输出中。

从版本 [2.4.0](../release-notes/2.4.0.md) 开始，oplog 切片运行如下：

* **逻辑备份** 

    在备份开始之前，时间点恢复例程会自动禁用。备份例程在备份创建期间创建 oplog 切片。备份完成后，时间点恢复例程会自动重新启用。它复制备份期间拍摄的切片，并从最新时间戳继续 oplog 切片。 

* **物理备份** 

    在物理备份期间，时间点恢复例程不会禁用，并继续与备份快照操作并行保存 oplog 切片。 


因此，如果备份快照很大并且需要数小时才能完成，所有 oplog 事件都会为其保存，确保恢复到任何时间戳。

[已知限制](known-limitations.md#oplog-slicing-for-point-in-time-recovery){.md-button}


### Oplog 持续时间

!!! admonition "版本添加：[1.6.0](../release-notes/1.6.0.md)"

默认情况下，切片涵盖 10 分钟的 oplog 事件跨度。如果时间点恢复被禁用或被备份快照操作的开始中断，它可能会更短。

您可以通过配置文件更改 oplog 跨度的持续时间。为 `pitr.oplogSpanMin` 选项指定新值（以分钟为单位）。

=== ":octicons-file-code-24: 命令行"

     ```bash
     pbm config --set pitr.oplogSpanMin=5
     ```

=== ":material-console: 配置文件"

     ```yaml
     pitr:
       oplogSpanMin: 5
     ```

如果您在 `pbm-agent` 正在创建 oplog 切片时设置新持续时间，切片的跨度会立即更新。

如果新持续时间更短，这会触发 `pbm-agent` 立即使用更新的跨度创建新切片。如果新持续时间更长，`pbm-agent` 会在其计划时间使用更新的跨度创建下一个切片。

### 调整 oplog 切片的节点优先级

!!! admonition "版本添加：[2.6.0](../release-notes/2.6.0.md)"

在版本 2.6.0 之前，保存 oplog 切片的 `pbm-agent` 在副本集成员中随机选择。 

从版本 2.6.0 开始，您可以通过配置文件为所需节点分配优先级来控制从哪个节点保存 oplog 切片。例如，您可以确保备份和 oplog 切片都从组织法规中定义的特定数据中心中的节点获取。或者，您可以通过从地理位置最近的位置的节点进行备份和/或 oplog 切片来减少网络延迟。  

oplog 切片的节点优先级处理方式与[创建备份的节点优先级](../usage/backup-priority.md) 类似，但它独立于它。因此，您可以为同一节点的备份和 oplog 切片分配不同的优先级。或者，仅调整 oplog 切片的优先级，保留备份的默认优先级。 

然后 PBM 根据它们的优先级处理这两个过程。

oplog 切片的默认节点优先级与创建备份相同：

* 隐藏节点 - 优先级 2
* 从节点 - 优先级 1
* 主节点 - 优先级 0.5

要重新定义它，请在配置文件中为 [`pitr.priority`](../reference/pitr-options.md#pitrpriority) 选项指定新优先级：

```yaml
pitr:
  enabled: true
  priority:
    "rs1:27017": 1
    "rs2:27018": 2
    "rs3:27019": 1
```

使用配置文件是定义 oplog 切片优先级的唯一方法。 


优先级数组的格式是 `<hostname:port>:<priority>`。

!!! important

    一旦您在配置文件中调整节点优先级，就假定您手动控制它们。优先选择从节点而不是主节点的默认规则停止工作。

要在分片集群中定义优先级，您可以列出所有节点，或者为每个分片和配置服务器副本集中的一个节点指定优先级。`hostname` 和 `port` 唯一标识一个节点，以便 Percona Backup for MongoDB 识别它属于哪里并相应地授予优先级。

请注意，如果您仅列出特定节点，剩余节点将自动分配优先级 `1.0`。例如，您仅为分片集群的每个分片和配置服务器副本集中的一个从节点分配优先级 `2.5`。

```yaml
pitr:
  enabled: true
  priority:
    "localhost:27027": 2.5  # config server replica set
    "localhost:27018": 2.5  # shard 1
    "localhost:28018": 2.5  # shard 2
```

集群中剩余的从节点和主节点接收优先级 `1.0`。

要检查优先级，请使用 `--priority` 标志运行 `pbm status` 命令。

```bash
pbm status --priority
```

??? example "示例输出"

    ```{.text .no-copy}
    Cluster:
    ========
    rs1:
      - rs1/rs101:27017 [S], Bkp Prio: [1.0], PITR Prio: [2.5]: pbm-agent [v{{release}}] OK
      - rs1/rs102:27017 [P], Bkp Prio: [0.5], PITR Prio: [2.0]: pbm-agent [v{{release}}] OK
      - rs1/rs103:27017 [S], Bkp Prio: [1.0], PITR Prio: [1.0]: pbm-agent [v{{release}}] OK
    ```

PBM 从优先级最高的节点保存 oplog 切片。如果此节点不响应，它会选择下一个优先级节点。如果有多个具有相同优先级的节点，则随机选择其中一个来保存 oplog 切片。


### 压缩的 oplog 切片 

!!! admonition "版本添加：[1.7.0](../release-notes/1.7.0.md)"

oplog 切片默认使用 `s2` 压缩方法保存。您可以通过配置文件指定不同的压缩方法。为 [`pitr.compression`](../reference/pitr-options.md#pitrcompression) 选项指定新值。

=== ":octicons-file-code-24: 命令行"

     ```bash
     pbm config --set pitr.compression=gzip
     ```

=== ":material-console: 配置文件"

     ```yaml
     pitr:
       compression: gzip
     ```

支持的压缩方法有：`gzip`、`snappy`、`lz4`、`s2`、`pgzip`、`zstd`。

此外，您可以通过设置 [`pitr.compressionLevel`](../reference/pitr-options.md#pitrcompressionlevel) 选项来覆盖压缩方法使用的压缩级别。每个压缩级别的默认值不同。 

请注意，您指定的值越高，压缩数据所需的时间和计算资源就越多。

!!! note 

    您可以为备份快照和时间点恢复切片使用不同的压缩方法。但是，与备份快照相关的 oplog 使用与备份本身相同的压缩方法进行压缩。

### 查看 oplog 切片

oplog 切片存储在配置中定义的[远程存储](../details/storage-configuration.md#supported-storage-types) 的 `pbmPitr` 子目录中。切片名称反映此切片涵盖的开始和结束时间。

[`pbm list`](../reference/pbm-commands.md#pbm-list) 输出包括以下信息：

* 备份快照。从版本 1.4.0 开始，它还显示完成时间（在版本 2.0.0 中重命名为 `restore_to_time`）
* 恢复的有效时间范围
* 时间点恢复状态

   ```bash
   pbm list

     2021-08-04T13:00:58Z [restore_to_time: 2021-08-04T13:01:23Z]
     2021-08-04T13:00:47Z [restore_to_time: 2021-08-05T13:01:11Z]
     2021-08-06T08:02:44Z [restore_to_time: 2021-08-06T08:03:09Z]
     2021-08-06T08:03:43Z [restore_to_time: 2021-08-06T08:04:08Z]
     2021-08-06T08:18:17Z [restore_to_time: 2021-08-06T08:18:41Z]

   PITR <off>:
     2021-08-04T13:01:24 - 2021-08-05T13:00:11
     2021-08-06T08:03:10 - 2021-08-06T08:18:29
     2021-08-06T08:18:42 - 2021-08-06T08:33:09
   ```

