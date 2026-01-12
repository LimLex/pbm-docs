# 在 MongoDB 中配置身份验证

Percona Backup for MongoDB 使用 MongoDB 的身份验证和授权子系统。这意味着要验证 Percona Backup for MongoDB，您需要：

* 为每个副本集在 `admin` 数据库中[创建相应的 `pbm` 用户](#create-the-pbm-user)
* [为 **pbm-agent** 设置有效的 MongoDB 连接 URI 字符串](#set-the-mongodb-connection-uri-for-pbm-agent) 
* [为 `pbm` CLI 设置有效的 MongoDB 连接 URI 字符串](#set-the-mongodb-connection-uri-for-pbm-cli)

## 创建 `pbm` 用户

!!! info

    每个 PBM 代理都连接到其本地节点。要配置身份验证，请在每个副本集的**主节点**上创建所需的用户和角色。 

    - 对于独立副本集，在其主节点上执行这些步骤。
    - 对于分片集群，在每个分片副本集的主节点以及配置服务器副本集上执行这些步骤。
    
1. 创建允许对任何资源执行任何操作的角色。

     ```javascript
     db.getSiblingDB("admin").createRole({ "role": "pbmAnyAction",
           "privileges": [
              { "resource": { "anyResource": true },
                "actions": [ "anyAction" ]
              }
           ],
           "roles": []
        });
     ```

2. 创建用户并为其分配您创建的角色。

     ```javascript
     db.getSiblingDB("admin").createUser({user: "pbmuser",
            "pwd": "secretpwd",
            "roles" : [
               { "db" : "admin", "role" : "readWrite", "collection": "" },
               { "db" : "admin", "role" : "backup" },
               { "db" : "admin", "role" : "clusterMonitor" },
               { "db" : "admin", "role" : "restore" },
               { "db" : "admin", "role" : "pbmAnyAction" }
            ]
         });
     ```

您可以根据需要更改 `username` 和 `password` 值，并为 `createUser` 命令指定其他选项。但您必须授予此用户如上所示的角色。


!!! tip

    要查看集群中所有分片副本集的主机+端口列表，请运行以下命令：

    ```javascript
    db.getSiblingDB("config").shards.find({}, {host: true, _id: false})
    ```

## 为 `pbm-agent` 设置 MongoDB 连接 URI

!!! info 
    
    在安装了 `pbm-agent` 的每个节点上执行此步骤。

`pbm-agent.service` systemd 单元文件包含环境文件的位置。您在每个 `pbm-agent` 的环境文件中为 `PBM_MONGODB_URI` 变量设置 MongoDB URI 连接字符串。 

??? tip "如何查找环境文件"

    在 Ubuntu 和 Debian 中，pbm-agent.service systemd 单元文件位于路径 `/lib/systemd/system/pbm-agent.service`。

    在 Red Hat 和 CentOS 中，此文件的路径是 `/usr/lib/systemd/system/pbm-agent.service`。

    **pbm-agent.service systemd 单元文件示例**

    ```init
    [Unit]
    Description=pbm-agent
    After=time-sync.target network.target

    [Service]
    EnvironmentFile=-/etc/default/pbm-agent
    Type=simple
    User=pbm
    Group=pbm
    PermissionsStartOnly=true
    ExecStart=/usr/bin/pbm-agent

    [Install]
    WantedBy=multi-user.target
    ```
   
=== ":material-debian: 在 Debian 和 Ubuntu 上"    

    编辑环境文件 `/etc/default/pbm-agent`。使用您之前创建的 `pbm` 用户的凭据指定 MongoDB 连接 URI 字符串。通过以下格式提供 URI 确保 pbm-agent 仅连接到其本地 mongod 节点：

    ```
    PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
    ```

    * 将 <pbmuser:secretpwd> 替换为您分配给 pbm 用户的实际凭据。
    * `localhost:27017` 确保代理仅连接到本地 `mongod` 实例。
    * `authSource=admin` 将身份验证指向创建 pbm 用户的 `admin` 数据库。

=== ":material-redhat: 在 Red Hat Enterprise Linux 和衍生版本上"

    编辑环境文件 `/etc/sysconfig/pbm-agent` 使用您之前创建的 `pbm` 用户的凭据指定 MongoDB 连接 URI 字符串。通过以下格式提供 URI 确保 pbm-agent 仅连接到其本地 mongod 节点：

    ```
    PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
    ```

    * 将 <pbmuser:secretpwd> 替换为您分配给 pbm 用户的实际凭据。
    * `localhost:27017` 确保代理仅连接到本地 `mongod` 实例。
    * `authSource=admin` 将身份验证指向创建 pbm 用户的 `admin` 数据库。

为了提高文件的安全性，您可以将其所有者更改为在 systemd 中配置的用户，并设置文件权限，以便只有所有者和 root 可以读取它。 

!!! important

    每个 **pbm-agent** 进程都需要使用独立类型的连接连接到其本地主机 `mongod` 节点。避免对 **pbm-agent** 使用副本集 URI，因为这可能导致意外行为。
    
    对于分片集群，每个分片成员上的 **pbm-agent** 还需要能够通过网络访问配置服务器副本集。代理通过查询 `db.system.version` 集合自动发现配置服务器的 URI。 

    请注意，`pbm-agent` 的 MongoDB 连接 URI 与 pbm CLI 所需的连接字符串不同。
    
### 包含特殊字符的密码

如果密码包含 `#`、`@`、`/` 等特殊字符，在将它们传递给 Percona Backup for MongoDB 时，必须使用[百分比编码机制](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1)转换这些字符。例如，密码 `secret#pwd` 应在 `PBM_MONGODB_URI` 中按如下方式传递：

```
PBM_MONGODB_URI="mongodb://pbmuser:secret%23pwd@localhost:27017/?authSource=admin"
```

## 为 `pbm CLI` 设置 MongoDB 连接 URI

!!! info 
    
    仅在您将运行 `pbm` CLI 命令进行备份或恢复的主机上执行此步骤。

在 shell 中将 MongoDB URI 连接字符串设置为 `pbm` CLI 的环境变量。这允许您运行 `pbm` 命令，而无需每次都指定 `--mongodb-uri` 标志。

=== "副本集"
   
    在非分片副本集中，将 PBM CLI 指向副本集。以下命令是如何为具有副本集成员 `mongors1:27017`、`mongors2:27017` 和 `mongors3:27017` 的副本集定义 MongoDB URI 连接字符串的示例：

    ```
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongors1:27017,mongors2:27017,mongors3:27017/?authSource=admin&replSetName=xxxx"
    ```
    
=== "分片集群"

    在分片集群中，将 PBM CLI 指向配置服务器副本集。以下命令是如何为具有配置服务器 `mongocfg1:27017`、`mongocfg2:27017` 和 `mongocfg3:27017` 的分片集群定义 MongoDB URI 连接字符串的示例：
    
    ```
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongocfg1:27017,mongocfg2:27017,mongocfg3:27017/?authSource=admin&replSetName=xxxx"
    ```

!!! important 
   
    pbm CLI 需要连接到存储 PBM 控制集合的副本集。MongoDB 连接 URI 与 `pbm-agent` 所需的连接字符串不同。
    
有关要指定的连接字符串的更多信息，请参阅 [pbm 连接字符串](../details/authentication.md#mongodb-connection-strings) 部分。

如果您在 MongoDB 中使用外部身份验证方法，请参阅 [Percona Backup for MongoDB 中的外部身份验证支持](../details/authentication.md#external-authentication-support-in-percona-backup-for-mongodb) 部分以获取配置指南。


## 下一步

[配置备份存储 :material-arrow-right:](backup-storage.md){.md-button}
