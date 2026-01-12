## 先决条件

运行 [`pbm status`](../reference/pbm-commands.md#pbm-status) 或 [`pbm list`](../reference/pbm-commands.md#pbm-list) 命令以检查完整备份快照是否存在以及是否有 oplog 切片。

## 开始之前

1. 停止平衡器和 `mongos` 节点。
2. 确保在恢复期间不对数据库进行写入。 
