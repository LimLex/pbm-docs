# 文件系统存储

## 远程文件系统服务器存储

远程文件系统服务器存储是挂载到每个 MongoDB 节点上的本地目录的文件服务器。服务器管理员必须确保集群或副本集中的所有节点都将相同的远程目录挂载到完全相同的本地路径。

!!! warning

    Percona Backup for MongoDB 将目录视为普通本地目录，不验证它是否实际从远程服务器挂载。

    如果您意外使用本地目录而不是远程挂载，其他节点将无法访问备份文件。这很可能在恢复操作期间导致错误，因为其他副本集成员上的 **pbm-agent** 进程无法访问存储在仅本地目录中的备份归档。

### 配置示例

您可以找到[配置文件模板 :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) 并取消注释所需字段。


```yaml
storage:
  type: filesystem
  filesystem:
    path: /data/local_backups
```

有关配置选项的说明，请参阅[配置文件选项](../reference/configuration-options.md)。


## 本地文件系统存储

本地文件系统存储仅支持单节点副本集。不建议在多节点设置中使用它，因为其他节点将无法访问备份文件。

对于测试，您可以使用您熟悉的任何对象存储。如果您没有对象存储，我们推荐 MinIO，因为它设置简单。如果您计划使用远程文件系统类型的备份服务器，请参阅上面的[远程文件系统服务器存储](#remote-filesystem-server-storage) 部分。
