# Percona Backup for MongoDB 工作原理

即使在高度可用的架构中（例如使用 MongoDB 复制），备份仍然是必需的，尽管丢失一台服务器不会致命。无论是完全还是部分数据灾难，您都可以使用 PBM（Percona Backup for MongoDB）回到过去的最佳可用备份快照。

Percona Backup for MongoDB 是一个命令行界面工具。它提供[一组命令](reference/pbm-commands.md)来管理数据库中的备份和恢复操作。

[选择哪种备份？](features/backup-types.md){.md-button}

## 使用示例

让我们看看 Percona Backup for MongoDB 是如何工作的。

在您的环境中[启动并运行 Percona Backup for MongoDB](installation.md) 后，创建备份：

```bash
pbm backup --type=logical
```

要保存备份之间发生的所有数据事件，请启用保存 oplog 切片的时间点恢复例程：

```bash
pbm config --set pitr.enabled=true
```

现在，假设您的 Web 应用程序更新于 2 月 7 日 03:00 UTC 发布。到 15:23 UTC，有人意识到此更新有一个错误，正在清除任何登录用户的历史数据。为了补救对数据的负面影响，是时候回滚到应用程序更新的时间 - 即 2 月 7 日 03:00 UTC。

```bash
pbm list
```

??? admonition "示例输出"
    
    ```{.text .no-copy}
    Backup snapshots:
        2024-02-05T13:55:55Z [complete: 2024-02-05T13:56:15]
        2024-02-07T13:57:58Z [complete: 2024-02-07T13:58:17]
        2024-02-03T08:08:15Z [complete: 2024-02-03T08:08:35]
        2024-02-09T14:06:06Z [complete: 2024-02-09T14:06:26]
        2024-02-11T14:22:41Z [complete: 2024-02-11T14:23:01]
    ```

输出列出了恢复的有效时间范围。所需时间（2 月 7 日 03:00 UTC）落在 `2024-02-03T08:08:36Z-2024-02-09T12:20:23Z` 范围内，因此让我们将数据库恢复到该时间。

```bash
pbm restore --time 2024-02-07T02:59:59
```

为了安全起见，在恢复完成后创建新备份是一个好习惯。

```bash
pbm backup
```

此备份刷新时间线，并作为保存 oplog 切片的基础。时间点恢复例程会自动重新启用。它复制备份期间拍摄的切片，并从最新时间戳继续 oplog 切片，以确保 oplog 连续性。

## 下一步

准备试试吗？ 

[快速入门](installation.md){.md-button}

## 有用的链接

* [PBM 架构](details/architecture.md)
* [备份类型](features/backup-types.md)
