# 当副本集成员数量更改时调整 PBM 配置

您可以根据工作负载添加或删除副本集成员。使用节点优先级和 `mongod` 二进制路径的默认配置，PBM 会自动扩展或缩减。 

如果您调整了这些配置中的任何一个，以下是确保 PBM 继续无错误运行所需执行的操作：

## 添加新的副本集成员 {.power-number}

1. 在新的 `mongod` 节点上[安装 Percona Backup for MongoDB](../installation.md)
2. 将新的 `mongod` 节点添加到副本集 
3. 在新节点上[设置并启动](../install/initial-setup.md) `pbm-agent`。 

    在此之前，您可能会在运行中的副本集的 `pbm status` 输出中看到以下错误：`rs202:30202 [S]: pbm-agent [v{{release}}] FAILED status: ERROR with lost agent, last heartbeat: 1748531485`。这是预期行为。

4. 调整 PBM 配置以包括新节点，用于[备份的节点优先级](../usage/backup-priority.md)、[oplog 切片](../features/point-in-time-recovery.md#adjust-node-priority-for-oplog-slices)或[定义 mongod 二进制文件的自定义路径](../usage/restore-physical.md#define-a-mongod-binary-location)。

## 删除副本集成员 {.power-number}

1. 编辑 PBM 配置以排除您计划删除的节点，用于[备份的节点优先级](../usage/backup-priority.md)、[oplog 切片](../features/point-in-time-recovery.md#adjust-node-priority-for-oplog-slices)或[定义 mongod 二进制文件的自定义路径](../usage/restore-physical.md#define-a-mongod-binary-location)。
2. 在您计划删除的节点上停止 `pbm-agent`：

    ```bash
    sudo systemctl stop pbm-agent
    ```

    您可能会在剩余节点的 `pbm status` 输出中看到如下错误：
    `- rs202:30202 [S]: pbm-agent [v{{release}}] FAILED status: > ERROR with lost agent, last heartbeat: 1748531485`。这是预期行为。

3. 删除副本集成员
4. `pbm status` 应该报告更新后的状态，没有错误。

