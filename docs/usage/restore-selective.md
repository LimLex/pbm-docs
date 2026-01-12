# 从逻辑备份进行选择性恢复

--8<-- "restore-intro.md"

## 开始之前

您可以从完整或选择性备份恢复特定数据库或集合。请阅读有关[选择性恢复的已知限制](../features/known-limitations.md#selective-backups-and-restores) 的信息。

## 恢复数据库

1. 列出备份

    ```bash
    pbm list
    ```
    
2. 以以下格式运行 ``pbm restore`` 命令：

    ```bash
    pbm restore <backup_name> --ns <database.collection>
    ```

 您可以为 `--ns` 标志指定多个命名空间作为逗号分隔列表：`<db1.col1>,<db2.*>`。例如，`--ns=customers.payments,invoices.*`。

在恢复期间，Percona Backup for MongoDB 检索指定数据库/集合的文件并恢复它。

### 使用用户和角色恢复

要从完整备份恢复[带有用户和角色的自定义数据库](../features/selective-backup.md#restore-a-database-with-users-and-roles)，请将 `--with-users-and-roles` 标志添加到 `pbm restore` 命令：

```bash
pbm restore <backup_name> --ns <database.*> --with-users-and-roles
```

### 以不同名称恢复集合

您可以在当前集合旁边以不同名称恢复特定集合。这在您排查数据库问题并需要比较两个集合中的数据以确定问题的根源时很有用。

请注意，在版本 2.8.0 中，您可以恢复单个集合，并且此集合必须是非分片的。

要恢复集合，请为 `--ns-from` 标志传递备份中的集合名称，为 `--ns-to` 标志传递新名称：

```bash
pbm restore <backup_name> --ns-from <database.collection> --ns-to <database.collection_new>
```

新集合具有与源集合相同的数据和索引。您必须为恢复的集合提供唯一名称，否则恢复将失败。

您可以以新名称恢复到指定时间。不要使用备份名称，而是按如下方式指定时间戳、源集合名称和新名称：

```bash
pbm restore --time=<timestamp> --ns-from <database.collection> --ns-to <database.collection_new>
```

## 恢复后步骤

恢复完成后，执行以下操作：

1. 启动平衡器和所有 mongos 节点以重新加载分片元数据。
2. 我们建议创建新备份作为未来恢复的新基础。

## 下一步

[时间点恢复](../usage/pitr-selective.md){.md-button}

## 有用的链接

* [查看恢复进度](../usage/restore-progress.md)
