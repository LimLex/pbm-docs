# PBM 控制集合

备份的配置和状态（当前和历史）存储在 MongoDB 集群或非分片副本集本身的集合中。这些集合放在系统 `admin` 数据库中，以将它们与用户数据库命名空间清晰分离。

在分片集群中，这是配置服务器副本集的 `admin` 数据库。在非分片副本集中，PBM 控制集合存储在副本集本身的 `admin` 数据库中。

* *admin.pbmBackups* - 每个备份的日志/状态。
* *admin.pbmAgents* - 包含有关 `pbm-agents` 状态和健康的信息。
* *admin.pbmConfig* - 包含 Percona Backup for MongoDB 的配置信息。
* *admin.pbmCmd* - 用于定义和触发操作。
* *admin.pbmLock* - **pbm-agent** 同步锁结构。
* *admin.pbmLockOp* - 用于协调不互斥的操作，例如创建备份和删除备份。
* *admin.pbmLog* - 存储来自 MongoDB 环境中所有 `pbm-agents` 的日志信息。从版本 1.4.0 开始在 Percona Backup for MongoDB 中可用。
* *admin.pbmOpLog* - 存储[操作 ID](../reference/glossary.md#opid)。
* *admin.pbmPITRChunks* - 存储[时间点恢复](../reference/glossary.md#point-in-time-recovery) oplog 切片。
* *admin.pbmPITRState* - 包含时间点恢复增量备份的当前状态。
* *admin.pbmRestores* - 包含所有副本集的恢复历史和恢复状态。
* *admin.pbmStatus* - 存储 Percona Backup for MongoDB 状态记录。

`pbm` 命令行工具根据需要创建这些集合。您不必维护这些集合，但也不应该不必要地删除它们。在备份期间删除它们将导致备份中止。

填充配置集合是使用 Percona Backup for MongoDB 执行备份或恢复的先决条件。（请参阅后面的配置页面。）
