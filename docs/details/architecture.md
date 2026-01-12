# 架构

Percona Backup for MongoDB 由以下组件组成：

* [`pbm-agent`](../reference/glossary.md#pbm-agent) 是在集群或副本集中**每个**非仲裁节点的 `mongod` 节点上运行的进程，用于执行备份和恢复操作。

* [`pbm` CLI](../reference/glossary.md#pbm-cli) 是指令 `pbm-agents` 执行操作的命令行实用程序。

* [PBM 控制集合](../reference/glossary.md#pbm-control-collections) 是 MongoDB 中存储配置数据和备份状态的特殊集合。`pbm` CLI 和 `pbm-agent` 都使用 PBM 控制集合来检查 MongoDB 中的备份状态并相互通信。

* 远程备份存储是 Percona Backup for MongoDB 保存备份的位置。它可以是 [S3 兼容存储](../reference/glossary.md#s3-compatible-storage) 或文件系统类型存储。

以下图表说明了 Percona Backup for MongoDB 组件如何与 MongoDB 通信。

![image](../_images/pbm-architecture.png){ align=center}
