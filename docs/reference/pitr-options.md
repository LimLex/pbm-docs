# 时间点恢复选项

```yaml
pitr:
  enabled: <boolean>
  oplogSpanMin: <float64>
  compression: <string>
  compressionLevel: <int>
  oplogOnly: <boolean>
  priority: 
    "rs1:27017": 1
    "rs2:27018": 2
    "rs3:27019": 1 
```

### pitr.enabled

*类型*：boolean <br>
*默认*：False

启用时间点恢复

### pitr.oplogSpanMin

*类型*：float64 <br>
*默认*：10

oplog 跨度的持续时间（以分钟为单位）。如果在 `pbm-agent` 正在创建 oplog 切片时设置，切片的跨度会立即更新。

如果新持续时间小于前一个持续时间，则触发 `pbm-agent` 保存具有更新跨度的新切片。如果持续时间更大，则下一个切片在计划时间使用更新的跨度保存。

### pitr.compression

*类型*：字符串 <br>
*默认*：s2

时间点恢复 oplog 切片的压缩方法。 

支持的值：`gzip`、`snappy`、`lz4`、`s2`、`pgzip`、`zstd`。默认：`s2`。

### pitr.compressionLevel

*类型*：int

压缩级别从 `0` 到 `10`。默认值取决于使用的压缩方法。

下表显示了每种压缩方法的可用压缩级别：

| 压缩方法   | 压缩级别           | 默认
| ------------------   | ---------------------------- | ----------
| `zstd`               | 1 - 最快速度, 2 - 默认, 3 - 更好的压缩, 4 - 最佳压缩 | 2
| `snappy`             | 无级别|
| `lz4`                | 从 1（最快）到 16 | 1
| `gzip` 和 `pgzip`   | -1 - 默认压缩, 0 - 无压缩, 1 - 最快速度, 9 - 最佳压缩| -1


请注意，您指定的值越大，压缩数据所需的时间和计算资源就越多。

### pitr.oplogOnly

*类型*：boolean <br>
*默认*：False <br>
*必需*：否

控制是否需要基础备份来启动[时间点恢复 oplog 切片](../features/point-in-time-recovery.md#oplog-slicing)。设置为 true 时，Percona Backup for MongoDB 在没有基础备份快照的情况下保存 oplog 块。

要了解更多用法，请参阅[时间点恢复 oplog 重放](../usage/oplog-replay.md)。

### pitr.priority

*类型*：字符串数组

`mongod` 节点列表及其保存 oplog 切片的优先级。优先级最高的节点被选为保存 oplog 切片。如果多个节点具有相同的优先级，则随机选择其中一个。

如果未设置，副本集节点具有以下默认优先级：

* 隐藏节点 - 2.0
* 从节点 - 1.0
* 主节点 - 0.5
