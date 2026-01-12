# 恢复选项

```yaml
restore:
  batchSize: <int>
  numInsertionWorkers: <int>
  numParallelCollections: <int>
  numDownloadWorkers: <int>
  maxDownloadBufferMb: <int>
  downloadChunkMb: <int>
  mongodLocation: <string>
  mongodLocationMap:
     "node01:2017": <string>
     "node03:27017": <string>
```

### restore.batchSize

*类型*：int <br>
*默认*：500

要缓冲的文档数。

### restore.numInsertionWorkers

*类型*：int <br>
*默认*：10

指定每个集合并发运行的插入工作线程数。 

### restore.numParallelCollections

*类型*：int <br>
*默认*：CPU 核心数 / 2

在逻辑恢复期间并行处理的集合数。默认值是 CPU 核心数的一半。通过设置此选项的值，您可以定义新的默认值。
从版本 2.7.0 开始可用。

### restore.numDownloadWorkers

*类型*：int <br>
*默认*：CPU 核心数

在恢复期间从存储请求数据块的工作线程数。默认值等于 CPU 核心数。

### restore.maxDownloadBufferMb

*类型*：int <br>
 

用于从 S3 存储下载文件的内存缓冲区的最大大小。未指定或设置为 0 时，大小不能超过计算为 `numDownloadWorkers * downloadChunkMb * 16` MB 的值。默认情况下，CPU 核心数 * 32 * 16 MB。

### restore.downloadChunkMb

*类型*：int <br>
*默认*：32

从 S3 存储下载的数据块大小（以 MB 为单位）。

### restore.mongodLocation

*类型*：字符串

`mongod` 二进制文件的自定义路径。未定义时，Percona Backup for MongoDB 使用默认路径在物理恢复期间进行数据库重启。

### restore.mongodLocationMap

*类型*：字符串数组

每个节点上 `mongod` 二进制文件的自定义路径列表。Percona Backup for MongoDB 使用这些值在物理恢复期间进行数据库重启。 
