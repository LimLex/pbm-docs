# 分片集群中的备份

!!! important "仅适用于 PBM v1.0"

    在集群上运行 pbm backup 之前，停止平衡器。

在分片集群中，每个分片和配置服务器副本集的一个 **pbm-agent** 进程直接将备份快照写入远程备份存储。对于逻辑备份，`pbm-agents` 还写入 oplog 切片。要了解有关 oplog 切片的更多信息，请参阅时间点恢复。

`mongos` 节点不参与备份过程。

以下图表说明了备份流程。

![image](../_images/pbm-backup-shard.png)

!!! important

    如果您在 MongoDB 5.0 及更高版本中重新分片集合，或在 MongoDB 8.0 及更高版本中取消分片集合，请创建新备份以防止数据不一致和恢复失败。
