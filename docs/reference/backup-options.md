# 备份选项

```yaml
backup:
  priority:
    "localhost:28019": 2.5
    "localhost:27018": 2.5
    "localhost:27020": 2.0
    "localhost:27017": 0.1
  compression: <string>
  compressionLevel: <int>
  timeouts:
    startingStatus: 60
  oplogSpanMin: <float64>
  numParallelCollections: <int>
```

### priority

*类型*：字符串数组

`mongod` 节点列表及其创建备份的优先级。优先级最高的节点被选为创建备份。如果多个节点具有相同的优先级，则随机选择其中一个来创建备份。

如果未设置，副本集节点具有以下默认优先级：

* 隐藏节点 - 2.0
* 从节点 - 1.0
* 主节点 - 0.5

### backup.compression

*类型*：字符串 <br>
*默认*：s2

备份快照的压缩方法。 

当指定 `none` 时，备份不进行压缩。

支持的值：`gzip`、`snappy`、`lz4`、`s2`、`pgzip`、`zstd`。默认：`s2`。

<!-- backup-compression-level: -->
### backup.compressionLevel

*类型*：int

压缩级别。默认值取决于使用的压缩方法。 

下表显示了每种压缩方法的可用压缩级别：

| 压缩方法   | 支持的压缩级别 | 默认
| ------------------   | ---------------------------- | ----------
| `zstd`               | 1 - 最快速度, 2 - 默认, 3 - 更好的压缩, 4 - 最佳压缩 | 2
| `snappy`             | 无级别|
| `lz4`                | 从 1（最快）到 16 | 1
| `gzip` 和 `pgzip`   | -1 - 默认压缩, 0 - 无压缩, 1 - 最快速度, 9 - 最佳压缩| -1

请注意，您指定的值越大，压缩数据所需的时间和计算资源就越多。

### backup.timeouts.startingStatus

*类型*：unit32 <br>
*默认*：33

PBM 启动备份的等待时间（以秒为单位）。此超时控制 PBM 等待备份从初始状态转换到运行状态的时间。

  对于分片集群中的物理备份，这包括在所有分片上打开 `$backupCursor` 所需的时间。对于逻辑备份，这包括收集集合统计信息
   （collStats 操作）和创建备份元数据。

  在以下情况下增加此值很有用：
  
  - 物理备份打开 `$backupCursor` 的时间比平时长
  - 具有许多集合的大型数据库需要更多时间进行元数据收集
  - 慢速存储系统延迟备份初始化

  适用于单个副本集和分片集群上的所有备份类型（逻辑、物理、增量、外部）。

  0（零）值将超时重置为默认的 33 秒。

### backup.oplogSpanMin

*类型*：float64 <br>

与逻辑备份快照一起保存的 oplog 切片的持续时间（以分钟为单位）。默认情况下，备份 oplog 切片的持续时间等于为 [`pitr.oplogSpanMin`](pitr-options.md#pitroplogspanmin) 选项定义的值（默认 - 10 分钟）。您可以在高负载环境中减少持续时间。请注意，将持续时间设置为更短的周期可能会增加整体备份执行时间。 

### backup.numParallelCollections

*类型*：int <br>
*默认*：CPU 核心数 / 2

在逻辑备份期间并行处理的集合数。默认情况下，并行集合数是 CPU 核心数的一半。通过设置此选项的值，您可以定义新的默认值。
从版本 2.7.0 开始可用。
