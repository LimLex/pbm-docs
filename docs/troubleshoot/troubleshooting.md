# Percona Backup for MongoDB 诊断工具

Percona Backup for MongoDB 提供诊断工具来操作数据备份。

## pbm-speed-test

**pbm-speed-test** 允许现场测试逻辑备份的压缩和备份上传速度。您可以使用它：

* 在启动备份之前检查性能

* 找出什么减慢了正在运行的备份

默认情况下，**pbm-speed-test** 使用假的半随机数据文档进行操作。要
在真实集合上运行 **pbm-speed-test**，请为 `--mongodb-uri` 标志提供有效的 MongoDB 连接 URI 字符串。

运行 **pbm-speed-test** 以获取可用命令的完整集合。

### 压缩测试

```bash
pbm-speed-test compression --compression=s2 --size-gb 10
```

??? example "示例输出"

    ```{.bash .no-copy}
    Test started ....
    10.00GB sent in 8s.
    Avg upload rate = 1217.13MB/s.
    ```

**pbm-speed-test compression** 使用配置文件中的压缩库
并将假的半随机数据文档（默认 1 GB）发送到
黑洞存储。（使用 `pbm config` 命令更改压缩库）。

要在真实集合上测试压缩，传递
`--sample-collection` 标志，值为 `<my_db.my_collection>`。

运行 `pbm-speed-test compression --help` 以获取支持标志的完整集合：

```bash
pbm-speed-test compression --help
```

??? example "示例输出"

    ```{.text .no-copy}
    usage: pbm-speed-test compression

    Run compression test

    Flags:
          --help                     Show context-sensitive help (also try
                                     --help-long and --help-man).
          --mongodb-uri=MONGODB-URI  MongoDB connection string
      -c, --sample-collection=SAMPLE-COLLECTION
                                     Set collection as the data source
      -s, --size-gb=SIZE-GB          Set data size in GB. Default 1
          --compression=s2           Compression type
                                     <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>
          --compression-level=COMPRESSION-LEVEL
                                     Compression level (specific to the compression type)
                                     <none>/<gzip>/<snappy>/<lz4>/<s2>/<pgzip>/<zstd>

### 上传速度测试

```bash
pbm-speed-test storage --compression=s2
```

??? example "示例输出"

    ```
    Test started
    1.00GB sent in 1s.
    Avg upload rate = 1744.43MB/s.
    ```

`pbm-speed-test storage` 将半随机数据（默认 1 GB）发送到
配置文件中定义的远程存储。传递 `--size-gb` 标志以更改
数据大小。

要使用真实集合的数据而不是半随机数据运行测试，
传递 `--sample-collection` 标志，值为 `<my_db.my_collection>`。

运行 `pbm-speed-test storage --help` 以获取可用标志的完整集合：

```bash
pbm-speed-test storage --help
```

??? example "示例输出"

    ```{.text .no-copy}
    usage: pbm-speed-test storage

    Run storage test

    Flags:
          --help                     Show context-sensitive help (also try --help-long and --help-man).
