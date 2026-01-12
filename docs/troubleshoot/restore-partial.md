---
glightbox-manual: true
---

# 部分完成的物理恢复

当您恢复物理备份时，操作可能导致以下状态：

- **Done**：恢复在所有节点上成功完成。
- **Error**：恢复失败且无法完成。
- **Partly-Done**：恢复在每个副本集的至少一个节点上成功，但在某些节点上失败。 

## 恢复状态决策流程

![image](../_images/restore-status.png)

决策流程说明如下：

1. 使用 `pbm describe-restore <restore_name>` 或 `pbm status` **检查恢复状态**。
2. **恢复是否在所有节点上成功？**
    - **是**：状态为 **Done**。继续执行[恢复后步骤](../usage/restore-physical.md#post-restore-steps)。
    - **否**：转到下一步。
3. **每个副本集中是否至少有一个节点恢复成功？**
    - **是**：`fallbackEnabled` 是否设置为 `true`？
        - **是**：是否启用了 `allowPartlyDone`？
            - **是**：状态为 **PartlyDone**。请参阅[`partlyDone` 恢复的恢复后步骤](#post-restore-steps-for-partlydone-restores)。
            - **否**：PBM 触发回退过程并将集群恢复到恢复前状态。如果 `fallbackEnabled` 被禁用，恢复状态为 **Error**。
    - **否**：状态为 **Error**。您需要手动干预。

## `partlyDone` 恢复的恢复后步骤

如果您的恢复以 **partlyDone** 状态完成，您仍然可以启动集群并等待失败的节点通过初始同步接收数据。以下是您需要做的：

1. 使用 `pbm status` 或 `pbm describe-restore <restore_name>` 检查恢复状态。
2. 启动所有 `mongod` 节点。失败的节点将从健康节点执行初始同步。
3. 在每个节点上启动 `pbm-agents`。
4. 启动平衡器和所有 `mongos` 节点。
5. 监控节点完成初始同步并报告 `ready` 状态。
6. 创建新备份作为未来恢复的新基础。
7. 如果需要，[启用时间点恢复](../features/point-in-time-recovery.md/#enable-point-in-time-recovery)。
