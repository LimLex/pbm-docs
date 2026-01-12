# pbm-agent

`pbm-agent` 是运行备份、恢复、删除和 Percona Backup for MongoDB 提供的其他操作的进程。

`pbm-agent` 实例必须为**每个**非仲裁节点的 `mongod` 实例运行。这包括当前为从节点的副本集节点和分片集群中的配置服务器副本集节点。

当 [`pbm` CLI](../reference/glossary.md#pbm-cli) 更新 [PBM 控制集合](../reference/glossary.md#pbm-control-collections) 时，会触发操作。所有 `pbm-agents` 都监控 PBM 控制集合的更改，但每个副本集中只有一个 `pbm-agent` 会被选出来执行操作。选举是通过在从节点中随机选择完成的。如果没有从节点响应，则选择主节点上的 `pbm-agent` 执行操作。

被选中的 `pbm-agent` 获取操作的锁。这防止了像备份和恢复这样的互斥操作同时执行。

??? info "PBM 内的提名和选举过程"

    如果运行在副本集主节点或配置副本集主节点上的 `pbm-agent` 失败，备份将不会启动，因为这些代理负责内部提名过程。因此，您必须确保所有 `pbm-agent` 进程都在运行。

操作完成后，`pbm-agent` 释放锁并更新 PBM 控制集合。

单个 `pbm-agent` 仅涉及一个集群（或非分片副本集）。`pbm` CLI 实用程序可以连接到其具有网络访问权限的任何集群，因此一个用户可以列出并在许多集群上启动备份或恢复。
