# 从逻辑备份恢复

--8<-- "restore-intro.md"

## 注意事项

1. 在恢复运行时，防止客户端访问数据库。恢复进行时数据自然是不完整的，客户端进行的写入会导致最终恢复的数据与备份的数据不同。

2. 对于版本 2.3.1 及更早版本，在运行 `pbm restore` 之前禁用[时间点恢复](../features/point-in-time-recovery.md)。这是因为时间点恢复 oplog 切片和恢复是不兼容的操作，不能一起运行。


## 开始之前

1. 停止平衡器并禁用块自动拆分。要验证两者是否都已禁用，请运行以下命令：

    ```javascript
     sh.status() 
    ```

    您应该看到以下输出：

    ```{text .no-copy}
    autosplit:
            Currently enabled: no
      balancer:
            Currently enabled: no
            Currently running: no
    ```        

2. 关闭所有 `mongos` 节点，以在恢复进行时阻止客户端访问数据库。这确保最终恢复的数据与备份的数据不同。

3. 关闭所有 `pmm-agent` 和其他可以对数据库执行写入操作的客户端。这是确保恢复后数据一致性所必需的。

4. 对于 PBM 版本 2.3.1 及更早版本，如果启用了时间点恢复，请手动禁用它。要了解有关时间点恢复的更多信息，请参阅[时间点恢复](../features/point-in-time-recovery.md)。

## 恢复数据库

1. 列出要恢复的备份

    ```bash
    pbm list
    ```


2. 从所需的备份恢复。在以下命令中将 `<backup_name>` 替换为所需的备份：

    ```bash
    pbm restore <backup_name>
    ```

    请注意，您只能将分片备份恢复到分片环境中。它可以是您现有的集群或新集群。要了解如何将备份恢复到新环境，请参阅[将备份恢复到新环境](../features/restore-new-env.md)。

### 恢复后步骤

集群恢复完成后，执行以下操作：

1. 启动平衡器和所有 `mongos` 节点以重新加载分片元数据。 
2. 我们建议创建新备份作为未来恢复的新基础。 
3. 备份完成后，时间点恢复会自动重新启用。否则，[启用时间点恢复](../features/point-in-time-recovery.md#enable-point-in-time-recovery) 以便能够恢复到特定时间。

## 调整内存消耗

从版本 1.3.2 开始，Percona Backup for MongoDB 配置包括恢复选项，用于在内存紧张的环境中调整 `pbm-agent` 的内存消耗。这可以防止恢复操作期间出现内存不足错误。

```yaml
restore:
  batchSize: 500
  numInsertionWorkers: 10
```

默认值已调整以适应代理内存分配为 1GB 及更少的设置。


从版本 2.8.0 开始，您可以覆盖每个集合的插入工作线程数以及在逻辑恢复期间并行处理的集合数。例如：

```bash
pbm restore <backup_name>  --num-insertion-workers-per-collection 4 --num-parallel-collections 8
```

增加数量可能会提高恢复速度。但是，它也可能导致意外的磁盘和 CPU 使用率过高。在进行调整之前，请仔细考虑权衡，以确保在不使资源过载的情况下获得最佳性能。

## 从在 Percona Server for MongoDB 的先前主要版本上创建的逻辑备份恢复

在某些情况下，您可能需要从在 Percona Server for MongoDB 的先前主要版本上创建的备份恢复。要实现这一点，备份和目标环境中的[功能兼容性版本 (FCV) :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/command/setFeatureCompatibilityVersion/) 值必须匹配。 

从版本 2.1.0 开始，Percona Backup for MongoDB 在备份元数据中存储 FCV 值。如果它与目标环境上的 FCV 值不匹配，您将在 [`pbm status`](../reference/pbm-commands.md#pbm-status) 输出中看到错误。

```{.bash .no-copy}
2023-04-10T10:48:54Z 302.80KB <logical> [ERROR: backup FCV "6.0" is incompatible with the running mongo FCV "5.0"] [2023-04-10T10:49:14Z]
2023-04-10T08:40:10Z 172.25KB <logical> [ERROR: backup mongo version "6.0.5-4" is incompatible with the running mongo version "5.0.15-13"] [2023-04-10T08:40:28Z]
```

以下示例说明了从在 Percona Server for MongoDB 4.4 上创建的备份恢复到 Percona Server for MongoDB 5.0。

1. 检查备份的 FCV 值

    ```bash
    pbm status
    ```

    ??? admonition "示例输出"

        ```{.bash .no-copy}
        Snapshots:
        2023-04-10T10:51:28Z 530.73KB <logical> [ERROR: backup FCV "4.4" is incompatible with the running mongo FCV "5.0"]     [2023-04-10T10:51:44Z]
        ```

2. 将功能兼容性版本值设置为 4.4

    ```javascript
    db.adminCommand( { setFeatureCompatibilityVersion: "4.4" } )
    ```

3. 恢复数据库

    ```bash
    pbm restore 2023-04-10T10:51:28Z
    ```

4. 将功能兼容性版本值设置为 5.0

    ```javascript
    db.adminCommand( { setFeatureCompatibilityVersion: "5.0" } )
    ```

## 下一步

[时间点恢复](../usage/pitr-tutorial.md){.md-button}

## 有用的链接

* [查看恢复进度](../usage/restore-progress.md)
* [恢复到新环境](../features/restore-new-env.md)
* [恢复到不同名称的集群](../features/restore-remapping.md)
