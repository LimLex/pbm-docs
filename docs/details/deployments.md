# 支持的 MongoDB 部署

Percona Backup for MongoDB 适用于分片集群和副本集。它不适用于独立的 MongoDB 实例。这是因为 Percona Backup for MongoDB 需要 [oplog](../reference/glossary.md#oplog) 来保证备份一致性。Oplog 在启用复制的节点上可用。

出于测试目的，您可以在单节点副本集上部署 Percona Backup for MongoDB。要将独立服务器转换为副本集，请在配置文件中指定 `replication.replSetName` 选项，然后初始化副本集。

!!! admonition "版本添加：[2.1.0](../release-notes/2.1.0.md)"

[物理恢复](../features/physical.md) 可用于具有仲裁节点的部署。但是，由于这些节点没有[安装 `pbm-agent`](../install/repos.md#what-nodes-to-install-on)，您必须[在恢复之前手动停止它们](../usage/restore.md#before-you-start)。

!!! admonition "另请参阅"

    MongoDB 文档：[将独立服务器转换为副本集](https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/)

