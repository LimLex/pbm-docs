# 创建选择性逻辑备份

--8<-- "prepare-backup.md"

## 步骤

!!! admonition "版本添加：[2.0.0](../release-notes/2.0.0.md)"

在开始之前，请阅读有关[选择性备份已知限制](../features/known-limitations.md#selective-backups-and-restores) 的信息。

要创建选择性备份，请运行 `pbm backup` 命令并为 `--ns` 标志提供格式为 `<database.collection>` 的值。`--ns` 标志值区分大小写。例如，要备份 "Payments" 集合，请运行以下命令：

```bash
pbm backup --ns=customers.payments
```

要备份 "Invoices" 数据库及其包含的所有集合，请按如下方式运行 ``pbm backup`` 命令：

```bash
pbm backup --ns=invoices.*
```

要备份多个命名空间，请将它们指定为 `--ns` 标志的逗号分隔列表：`<db1.col1>`、`<db2.*>`、`<db3.collX>`。要指定的命名空间数量没有限制。

在备份过程中，Percona Backup for MongoDB 以新的多文件格式存储数据，其中每个集合都有一个单独的文件。无论这是完整备份还是选择性备份，都会为所有命名空间存储 oplog。

多格式是完整和选择性备份的默认数据格式，因为它允许选择性恢复。但是请注意，您只能从使用早期版本的 Percona Backup for MongoDB 创建的备份进行完整恢复。 


## 下一步

[列出备份](../usage/list-backup.md){.md-button}
[进行恢复](restore-selective.md){.md-button}

## 有用的链接

* [备份和恢复类型](../features/backup-types.md)
* [安排备份](../usage/schedule-backup.md)

