# 与 MongoDB 的比较

Percona Backup for MongoDB 是一个完全支持的社区备份解决方案，可以在 MongoDB 中执行集群范围的一致性备份。下表将 Percona Backup for MongoDB 与 MongoDB 备份解决方案进行了比较：

| 功能名称 | Percona Backup for MongoDB | MongoDB Community `mongodump` | MongoDB Enterprise | MongoDB Atlas |
| -------------| -------------------------- | ----------- | ----------------- | --------------- 
| 开源软件 | 是 | 否	| 否 | 否 
| 支持分片集群备份和恢复 | 是 | 否 | 是 |	是
| 二进制数据库导出（逻辑备份） | 是 | 是 | 是 | 是 
| 选择性备份（逻辑） | 是 | 是 | 是 | 是
| 内置时间点恢复支持 |	是 | 否 | 是 | 是
| 物理备份 |	是	| 否 | 是 | 是
| 增量备份（物理） | 是 |	否 |是 | 是
| 备份管理界面	| Percona Backup for MongoDB (CLI) <br> PMM <br> mongodump / mongorestore (CLI) | - <br> - <br> mongodump / mongorestore (CLI) | Ops Manager <br> Cloud Manager <br> mongodump / mongorestore (CLI) | Atlas backups <br> mongodump / mongorestore (CLI)
| 多个备份存储| 是 | 否 | 否 | 否

## 使用 Percona Backup for MongoDB 您将获得什么

* [无需额外成本的企业功能](comparison.md) 
* [适用于分片集群和非分片副本集](../details/deployments.md)
* [简单的命令行管理实用程序](../reference/pbm-commands.md)。对于通过用户界面进行备份管理，请考虑[通过 Percona Monitoring and Management 使用 PBM :octicons-link-external-16:](https://docs.percona.com/percona-monitoring-and-management/get-started/backup/index.html)
* 简单的[与 MongoDB 集成的身份验证](../details/authentication.md#external-authentication-support-in-percona-backup-for-mongodb)
* 与 MongoDB 4.2+ 的分布式事务一致性
* 与不同存储类型的兼容性：[S3 兼容存储](../details/s3-storage.md)、[Microsoft Azure Blob 存储](../details/azure.md)、用于[本地挂载的远程文件系统备份服务器](../details/filesystem-storage.md#remote-filesystem-server-storage) 的 `filesystem` 存储类型
