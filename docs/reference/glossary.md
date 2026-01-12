# 术语表

## ACID
     
保证数据库事务可靠处理的一组属性。代表 [`原子性`](#atomicity)、[`一致性`](#consistency)、[`隔离性`](#isolation)、[`持久性`](#durability)。

## Amazon S3

Amazon S3（Simple Storage Service）是通过 Amazon Web Services 提供的 Web 服务接口提供的对象存储服务。

## 原子性

原子性意味着数据库操作遵循"全有或全无"规则。事务要么完全应用，要么根本不应用。

## Blob
    
Blob 代表二进制大对象，包括图像和多媒体文件等对象。换句话说，这些是您存储在 Microsoft 数据存储平台中的各种数据文件。Blob 组织在[容器](#container) 中，这些容器保存在您的存储账户下的 Azure Blob 存储中。

## Bucket

存储桶是 s3 远程存储上的容器，用于存储备份。

## Collection
     
集合是 MongoDB 中组织数据的方式。它类似于关系数据库中的表。

## 完成时间

从版本 2.0.0 开始，完成时间重命名为 "restore_to_time"

完成时间是分片集群/非共享副本集在恢复后将返回到的时间。它反映在 ``pbm list`` / ``pbm status`` 命令输出的 "complete" 部分中。

在 `logical` 备份中，完成时间几乎与备份完成时间一致。为了定义完成时间，Percona Backup for MongoDB 等待备份快照在所有集群节点上完成。然后它捕获从备份开始时间到该时间的 oplog。 

在 `physical` 备份中，完成时间仅在备份开始时间后几秒。通过保持 ``$backupCursor`` 打开，Percona Backup for MongoDB 保证检查点数据在备份期间不会更改。通过这种方式，Percona Backup for MongoDB 可以提前定义完成时间。

## 一致性

在备份和恢复的上下文中，一致性意味着恢复的数据将在给定时间点保持一致。原子操作的部分或不完整写入磁盘（例如，分别写入表和索引数据结构）在恢复后不会提供给客户端。这同样适用于已开始但在备份完成时未完成的多文档事务。

## Container 
   
容器就像 Azure Blob 存储中包含一组 [blob](#blob) 的目录。

## 持久性
   
一旦事务被提交，它将保持如此。

## EBS 快照

EBS（Amazon Elastic Block Storage）快照是数据的时间点副本，可用于启用灾难恢复、跨区域和账户迁移数据，以及改善备份合规性。

## Endpoint

S3 兼容存储服务（如 MinIO）可访问的网络地址（URL 或 IP）。  

## GCP
   
GCP（Google Cloud Platform）是在 Google Cloud 基础设施上运行的服务集合，包括存储服务。

## 隔离性

隔离性要求意味着没有事务可以干扰另一个事务。

## Jenkins
     
[Jenkins](http://www.jenkins-ci.org) 是一个持续集成系统，我们使用它来帮助确保我们生产的软件的持续质量。它帮助我们实现以下目标：

* 在任何平台上主干中没有失败的测试
* 帮助开发人员确保合并请求在所有平台上构建和测试，
* 没有已知的性能回归（没有很好的解释）。

## MinIO

MinIO 是与 [Amazon S3](#amazon-s3) 兼容的云存储服务器，在 Apache License v2 下发布。

## Oplog
  
Oplog（操作日志）是一个固定大小的集合，它保持修改数据库中数据的所有操作的滚动记录。 

## Oplog 切片

存储在 MongoDB 的 Oplog Store 数据库中的 [oplog](#oplog) 条目的压缩包。oplog 大小捕获大约 10 分钟的时间范围。对于快照，oplog 大小由最慢的副本集成员执行 mongodump 所需的时间定义。    

## OpID

操作（如备份、恢复、重新同步）的唯一标识符。当 pbm-agent 开始处理操作时，它获取锁和 opID。这防止处理同一操作两次（例如，如果分布式系统中有网络问题）。使用 opID 作为日志过滤器允许查看正在进行的操作的日志。

## 存储的路径样式访问

构造 S3 URL 的方法，其中存储桶名称出现在 URL 的路径部分。URL 格式为 `<https://s3.example.com/bucket-name/object-key>`。对于像 MinIO 这样的 S3 兼容存储系统，这是首选，特别是在没有通配符 DNS 或自定义 SSL 证书的环境中。

## `pbm-agent`

`pbm-agent` 是在 mongod 节点上运行的用于备份和恢复操作的 PBM 进程。每个 mongod 节点（包括副本集从成员和配置服务器副本集节点）都需要一个 pbm-agent 实例。   

## pbm CLI
     
用于控制备份系统的命令行界面。PBM CLI 可以连接到多个集群，以便用户可以管理许多集群上的备份。

## PBM 控制集合
   
PBM 控制集合是包含配置、身份验证数据和备份状态的[集合](#collection)。它们存储在集群或非分片副本集的 admin 数据库中，并作为 [`pbm-agent`](#pbm-agent) 和 [`pbm CLI`](#pbm-cli) 之间的通信通道。`pbm CLI` 为新操作创建新的 `pbmCmd` 文档。`pbm-agents` 监控它并在处理操作时更新。

## Percona Backup for MongoDB

Percona Backup for MongoDB (PBM) 是用于 MongoDB 非分片副本集和集群的低影响备份解决方案。它支持 [Percona Server for MongoDB](#percona-server-for-mongodb) 和 MongoDB Community Edition。 

## Percona Server for MongoDB 

Percona Server for MongoDB 是具有企业级功能的 MongoDB Community Edition 的即插即用替代品。

## 时间点恢复
     
时间点恢复是将数据库恢复到特定时间点。数据从备份快照恢复，然后从 oplog 重放发生在数据上的事件。 

## 副本集
   
副本集是托管相同数据集的一组 `mongod` 节点。

## S3 兼容存储 

这是基于 [S3](#amazon-s3) API 构建的存储。
 
## 服务器端加密
   
服务器端加密是远程存储服务器在接收数据时对数据进行加密。数据在写入 S3 存储桶时加密，在您访问数据时解密。 

## 技术预览功能

技术预览功能尚未准备好用于企业使用，也不包含在通过 SLA 的支持中。它们包含在此版本中，以便用户可以在未来 GA 版本中完整发布功能（或如果认为功能无用则删除功能）之前提供反馈。此功能可能会从技术预览到 GA 发生变化（API、CLI 等）。 

## 虚拟主机样式访问

构造 S3 URL 的方法，其中存储桶名称是域名的一部分。URL 格式为 `<https://bucket-name.s3.example.com/object-key>`。在新区域中由 AWS S3 要求；在大规模部署中实现更好的路由和性能。
