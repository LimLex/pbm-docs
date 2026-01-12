# 常见问题

## PBM 和 `mongodump` 有什么区别？

`mongodump` 只是一个"逻辑"备份解决方案，而 Percona Backup for MongoDB 支持逻辑和物理备份。两种解决方案对于非分片副本集具有相同的性能。但是，与 `mongodump` 相反，Percona Backup for MongoDB 允许您实现以下目标：

* 在分片集群中进行一致的备份和恢复。
* 备份/恢复整个数据集和特定命名空间——数据库和集合。（有关更多信息，请参阅[选择性备份和恢复](../features/selective-backup.md)。）
* 将数据库恢复到特定时间点。
* 在每个副本集上并行运行备份/恢复，而 `mongodump` 在 `mongos` 节点上的一个进程中运行。

## 为什么 Percona Backup for MongoDB 使用 UTC 时区而不是服务器本地时区？

`pbm-agents` 按设计使用 UTC 时区。这样做的原因是为了避免在副本集/集群节点在地理上分布在不同时区时用户误解。

从版本 2.0.1 开始，您可以更改 ``pbm logs`` 输出的时区。

## 我可以使用 Percona Backup for MongoDB 恢复单个集合吗？

是的。从版本 2.0.0 开始，您可以使用 Percona Backup for MongoDB 恢复单个集合。此功能仅适用于逻辑备份和恢复。要了解更多信息，请参阅[选择性备份和恢复](../features/selective-backup.md)。

## 我可以备份集群中的特定分片吗？

不可以，因为这会导致备份在整个集群中具有不一致的时间戳。此类备份对于恢复无效。

Percona Backup for MongoDB 备份分片集群的整个状态，这保证了恢复期间的数据一致性。

## 我是否需要为 PITR 恢复停止平衡器？

是的。时间点恢复恢复和常规恢复的先决条件相同：


1. 在分片集群中，停止平衡器。


2. 确保在恢复期间不对数据库进行写入。这确保了数据一致性。


3. 如果启用了时间点恢复，请禁用它。这是因为 oplog 切片和恢复是互斥操作，不能一起运行。请注意，在恢复之后和下一个备份快照之前创建的 oplog 切片变得无效。创建新备份并重新启用时间点恢复。

## 我可以在 MacBook 上安装 PBM 吗？

您不能使用包管理器在 MacBook 上安装 PBM。PBM 软件包仅适用于 Linux 发行版并经过测试。但是，您可以在 MacBook 上作为 Docker 容器运行 PBM。有关指南，请参阅[在 Docker 中运行](../install/docker.md) 指南。 

## 我可以将 PBM 连接到禁用授权的 MongoDB 吗？

虽然我们**不建议**由于安全考虑而禁用 MongoDB 的授权，但在某些场景（例如测试环境）中，您可能需要将 PBM（Percona Backup Manager）连接到没有身份验证的 MongoDB 实例。按照以下步骤确保成功设置：

1. 如果您为 `mongod` 或 `mongos` 进程设置了 [`bindIP` :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-net.bindIp) 配置参数，请确保 `localhost` 值包含在允许的 IP 地址列表中。`pbm-agent` 进程使用独立类型的连接连接到其本地 MongoDB 节点。 
2. 确保 pbm-agents 的 MongoDB 连接 URI 字符串在主机信息中包含 `localhost`。 
3. 在 PBM CLI 的 MongoDB 连接 URI 字符串中，排除 `authSource` 参数，否则它会强制执行授权。
