# 卸载 Percona Backup for MongoDB

要卸载 Percona Backup for MongoDB，请执行以下步骤：


1. 在 [`pbm list`](../reference/pbm-commands.md#pbm-list) 的输出中检查当前没有正在进行的备份。

2. 在执行接下来的 2 个步骤之前，请确保您知道远程备份存储的位置，以便可以删除由 Percona Backup for MongoDB 创建的备份。如果是 S3 兼容的对象存储，您将需要使用其他工具（如 Amazon AWS 的 "aws s3"、Minio 的 `mc`、Web AWS Management Console 等）在卸载 Percona Backup for MongoDB 后执行此操作。不要忘记在删除连接凭据之前也记下它们。

3. 卸载 **pbm-agent** 和 `pbm` 可执行文件。如果您使用包管理器安装，请参阅[安装 Percona Backup for MongoDB](../installation.md) 以获取适用于您的操作系统发行版的相关包名称和命令。

4. 删除 PBM 控制集合。

5. 删除 PBM 数据库用户。如果是集群，`dropUser` 命令需要在每个分片以及配置服务器副本集上运行。

6. （可选）从远程备份存储中删除备份。
