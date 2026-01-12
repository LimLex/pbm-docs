# 从增量备份恢复

--8<-- "restore-intro.md"

## 注意事项

1. 备份和恢复数据的 Percona Server for MongoDB 版本必须在同一主要版本内。
2. 使用 PBM 2.1.0 之前的 PBM 创建的增量备份与 PBM 2.1.0 及更高版本的恢复不兼容。
3. 不支持具有仲裁节点的部署的物理恢复。
   
## 开始之前

1. 关闭所有 `mongos` 节点、`pmm-agent` 进程和可以对数据库执行写入的客户端，因为恢复进行时数据库将不可用。
2. 检查 `pbm-agent.service` 的 `systemctl` 重启策略未设置为 `always` 或 `on-success`：

    ```bash
    sudo systemctl show pbm-agent.service | grep Restart
    ```

    ??? example "示例输出"

        ```{.text .no-copy}
        Restart=no
        RestartUSec=100ms
        ```
    
    在物理恢复期间，数据库不得自动重启，因为这由 `pbm-agent` 控制。
   
## 恢复数据库

从增量备份恢复的流程与从完整物理备份恢复相同：为 `pbm restore` 命令指定备份名称：

```bash
pbm restore 2022-11-25T14:13:43Z
```

Percona Backup for MongoDB 识别备份类型，找到增量基础备份，从中恢复数据，然后从适用的增量备份恢复修改的数据。

### 恢复后步骤

恢复完成后，执行以下操作：

1. 重启所有 `mongod` 节点和 `pbm-agents`。 

    !!! note

        集群重启后，您可能会在 `mongod` 日志中看到以下消息：

        ```{.text .no-copy}
        "s":"I",  "c":"CONTROL",  "id":20712,   "ctx":"LogicalSessionCacheReap","msg":"Sessions collection is not set up; waiting until next sessions reap interval","attr":{"error":"NamespaceNotFound: config.system.sessions does not exist"}}}}
        ```

        这是数据库启动时定期检查的预期行为。在恢复期间，`config.system.sessions` 集合被删除，但 Percona Server for MongoDB 最终会重新创建它。这是正常过程。您无需执行任何操作。
    
2. 从存储重新同步备份列表。 
3. 启动平衡器和 `mongos` 节点。
4. 作为一般建议，创建新的基础备份以更新后续增量备份的起点。


## 有用的链接 

* [查看恢复进度](../usage/restore-progress.md)





  



