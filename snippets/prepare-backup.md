## 开始之前

1. [安装](../installation.md) 并[设置 Percona Backup for MongoDB](../install/initial-setup.md)
2. 使用 [`pbm status`](../reference/pbm-commands.md#pbm-status) 命令检查 `pbm agent` 是否正在运行
3. 检查所有 `pbm-agents` 和 PBM CLI 是否具有相同的版本。否则，我们无法保证成功的备份和其中的数据一致性。 

    要检查版本，请运行以下命令：

    *  [`pbm status`](../reference/pbm-commands.md#pbm-status) 检查 pbm-agents 的版本 
    *  [`pbm version`](../reference/pbm-commands.md#pbm-status) 检查 PBM CLI 的版本。 
