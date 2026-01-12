# 取消备份

如果例如您想对服务器进行其他维护并且不想等待大型备份完成，您可以取消正在运行的备份。

要取消备份，请使用 [`pbm cancel-backup`](../reference/pbm-commands.md#pbm-cancel-backup) 命令。

```bash
pbm cancel-backup
Backup cancellation has started
```

命令执行后，备份在 [pbm status](../troubleshoot/status.md) 输出中被标记为已取消：

```bash
pbm status
```

**输出**：

```{.text .no-copy}
2020-04-30T18:05:26Z  Canceled at 2020-04-30T18:05:37Z
```
