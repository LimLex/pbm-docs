# 从备份恢复到新环境

要将备份从一个环境恢复到新环境，请确保以下事项：

1. 新环境中的 Percona Backup for MongoDB 配置必须指向为原始环境定义的相同远程存储，包括如果是对象存储的身份验证凭据。 

2. （可选）在运行时间点恢复之前，您可以运行 `pbm config --force-resync` 命令以同步原始环境和新环境之间的元数据。这使新环境了解备份存储的最新状态。

3. 当 Percona Backup for MongoDB 配置指向原始环境的远程存储位置时，不要从新环境运行 [`pbm backup`](../reference/pbm-commands.md#pbm-backup)。

4. 当 Percona Backup for MongoDB 配置指向原始环境的远程存储位置时，不要在新环境上[启用时间点恢复](point-in-time-recovery.md)。

一旦您运行 [`pbm list`](../reference/pbm-commands.md#pbm-list) 并看到从原始环境创建的备份，您就可以运行 [`pbm restore`](../reference/pbm-commands.md#pbm-restore) 命令。

恢复完成后，重新配置 PBM 以指向新环境中的远程存储，以停止原始环境产生备份数据。
