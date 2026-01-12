# 配置备份压缩

默认情况下，Percona Backup for MongoDB 在创建备份时使用 `s2` 压缩方法。

您可以通过将 `--compression` 标志传递给 **pbm backup** 命令来使用不同的压缩方法启动备份。

例如，要使用 `gzip` 压缩启动备份，请使用以下命令：

```bash
pbm backup --compression=gzip
```

支持的压缩类型有：`gzip`、`snappy`、`lz4`、`pgzip`、`zstd`。`none` 值表示备份期间不进行压缩。

## 压缩级别设置

您可以配置备份的压缩级别。为 [`--compression-level`](../reference/backup-options.md#backupcompressionlevel) 标志指定值。 

默认压缩级别因使用的压缩方法而异。 

下表显示了每种压缩方法的可用压缩级别：

| 压缩方法   | 支持的压缩级别 | 默认
| ------------------   | ---------------------------- | ----------
| `zstd`               | 1 - 最快速度, 2 - 默认, 3 - 更好的压缩, 4 - 最佳压缩 | 2
| `snappy`             | 无级别|
| `lz4`                | 从 1（最快）到 16 | 1
| `gzip` 和 `pgzip`   | -1 - 默认压缩, 0 - 无压缩, 1 - 最快速度, 9 - 最佳压缩| -1

请注意，您指定的值越高，压缩数据所需的时间和计算资源就越多。
