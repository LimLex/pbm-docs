# 身份验证

Percona Backup for MongoDB 没有自己的身份验证和授权子系统——它使用 MongoDB 的身份验证和授权子系统。这意味着 `pbm` CLI 和 `pbm-agent` 只需要 `pbm` 用户的有效 MongoDB 连接 URI 字符串。

有关 S3 兼容远程存储身份验证配置，请参阅
[集群（或非分片副本集）中的 Percona Backup for MongoDB 配置](../reference/config.md)。

## MongoDB 连接字符串

Percona Backup for MongoDB 使用 [MongoDB 连接 URI :octicons-link-external-16:](https://docs.mongodb.com/manual/reference/connection-string/) 字符串打开
MongoDB 连接。`pbm` CLI 和 `pbm-agent` 都不接受传统风格的
命令行参数，如 `--host`、`--port`、`--user`、`--password`、
等，就像 `mongo` shell 或 `mongodump` 命令那样。


=== "`pbm-agent` 连接字符串"

     `pbm-agent` 进程应该使用独立类型的连接连接到其本地主机 `mongod`。

     ```bash
     pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
     ```

     或者：

     ```bash 
     export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin"
     pbm-agent
     ```
     
     将 `pbmuser:secretpwd` 替换为[拥有 pbm 进程的用户](../install/configure-authentication.md#create-the-pbm-user) 的凭据。

=== "`pbm` CLI 连接字符串"

     ```bash
     pbm status --mongodb-uri "mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin"
     ```

     或者：

     ```bash
     export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin"
     pbm status
     ```
     
     将 `pbmuser:secretpwd` 替换为[拥有 pbm 进程的用户](../install/configure-authentication.md#create-the-pbm-user) 的凭据

     `pbm` CLI 最终将连接到具有 PBM 控制集合的副本集。

       * 在非分片副本集中，它只是该副本集。
       * 在集群中，它是配置服务器副本集。

     您不一定必须提供该连接字符串。如果您提供到任何活动节点（分片、configsvr 或非分片副本集成员）的连接，`pbm` CLI 将自动确定正确的主机并建立到这些主机的新连接。

上面的连接 URI 是 MongoDB 驱动程序自大约 MongoDB 服务器 v3.6 发布以来普遍接受的格式。`mongo` shell
[自 v4.0 以来也接受它](https://docs.mongodb.com/v4.0/mongo/#mongodb-instance-on-a-remote-host)。使用
v4.0+ mongo shell 是从命令行调试连接 URI 有效性的推荐方法。

!!! admonition ""

    由于 Percona Backup for MongoDB 必须在 MongoDB 中进行身份验证，我们建议在连接 URI 字符串中使用 `authSource` 选项指定与 `pbm` 用户凭据关联的身份验证数据库。

    [MongoDB 连接 URI](https://docs.mongodb.com/manual/reference/connection-string/) 规范还允许通过 `defaultauthdb` 组件指定身份验证数据库。但是，在这种情况下，Percona Backup for MongoDB 仅备份此指定的数据库。

    如果 `authSource` 和 `defaultauthdb` 都未指定，身份验证数据库默认为 `admin` 数据库。

[MongoDB 连接 URI](https://docs.mongodb.com/manual/reference/connection-string/) 规范
包括您可能需要使用的几个非默认选项。例如，连接到启用网络加密的集群或非分片副本集所需的 TLS
证书/密钥是 "tls=true" 加上 "tlsCAFile" 和/或
"tlsCertificateKeyFile"（请参阅 [tls 选项](https://docs.mongodb.com/manual/reference/connection-string/#tls-options)）。

### Read Concern / Write Concern 配置

!!! admonition "版本添加：[2.5.0](../release-notes/2.5.0.md)"

默认情况下，PBM 要求副本集成员在 MongoDB 的读写操作中达到多数级别的确认。此级别由 `readConcern` 和 `writeConcern` 设置控制。 

如果您的集群失去多数或配置为在没有多数的情况下运行，您可以降低 `readConcern` 和 `writeConcern` 的级别，以便 PBM 保持运行并可以创建备份。



在 MongoDB 连接 URI 字符串中指定新值，如下所示：

=== "pbm-agent 连接字符串"    

    ```bash
    pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin&readConcernLevel=local&w=1"
    ```  

    或者：    

    ```bash
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin&readConcernLevel=local&w=1"
    pbm-agent
    ```    

=== "`pbm` CLI 连接字符串"    

    ```bash
    pbm status --mongodb-uri "mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin&readConcernLevel=local&w=1"
    ```    

    或者：    

    ```bash
    export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@mongocsvr1:27017,mongocsvr2:27017,mongocsvr3:27017/?replicaSet=configrs&authSource=admin&readConcernLevel=local&w=1"
    pbm status
    ```

支持的值有：

* 对于 `readConcern` – `local`
* 对于 `writeConcern` – 确认写入的节点数。不支持零值。

为了 PBM 的正确运行，我们建议更改两个选项的值。

要从备份恢复，首先将集群配置为具有多数。然后[进行恢复](../usage/restore.md)。  

## Percona Backup for MongoDB 中的外部身份验证支持

除了 SCRAM，Percona Backup for MongoDB 还支持您在 MongoDB 或 Percona Server for MongoDB 中使用的其他[身份验证方法 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/authentication.html)。

对于外部身份验证，您以身份验证系统使用的格式创建 `pbm` 用户，并设置 MongoDB 连接 URI 字符串以包括身份验证方法和身份验证源。

### Kerberos

对于 [Kerberos 身份验证 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/authentication.html#kerberos-authentication)，在 `$external` 数据库中创建 `pbm` 用户，格式为 `<username@KERBEROS_REALM>`（例如 [pbm@PERCONATEST.COM](mailto:pbm@PERCONATEST.COM)）。

为 MongoDB 连接 URI 指定以下字符串：

```bash
PBM_MONGODB_URI="mongodb://<username>%40<KERBEROS_REALM>@<hostname>:27018/?authMechanism=GSSAPI&authSource=%24external&replSetName=xxxx"
```

请注意，您必须首先使用 `kinit` 命令为 `pbm` 用户获取票证，然后再启动 **pbm-agent**：

```bash
sudo -u {USER} kinit pbm
```

请注意，`{USER}` 是您将运行 `pbm-agent` 进程的用户。

### LDAP 绑定

对于[通过 Native LDAP 进行身份验证和授权 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/authorization.html#authentication-and-authorization-with-direct-binding-to-ldap)，您只需在 MongoDB 中为 LDAP 组创建角色，因为用户存储在 LDAP 服务器上并由其管理。但是，您仍然将 `$external` 数据库定义为您的身份验证源：

```bash
PBM_MONGODB_URI="mongodb://<user>:<password>@<hostname>:27017/?authMechanism=PLAIN&authSource=%24external&replSetName=xxxx"
```

### AWS IAM

使用 [AWS IAM 身份验证 :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/latest/aws-iam.html) 时，在 `$external` 数据库中创建 `pbm` 用户，用户名包含 IAM 用户/角色的 ARN。


=== ":fontawesome-regular-user: 用户身份验证"

     ```
     arn:aws:iam::<ARN>:user/<user_name>
     ```

=== ":material-cloud-key-outline: 角色身份验证"

     ```
     arn:aws:iam::<ARN>:role/<role_name>
     ```

MongoDB 连接 URI 字符串如下所示：

```bash
PBM_MONGODB_URI="mongodb://<aws_access_key_id>:<aws_secret_access_key>@<hostname>:27017/?authMechanism=MONGODB-AWS&authSource=%24external&replSetName=xxxx"
```

### AWS EKS

如果 Percona Backup for MongoDB 在 Amazon Elastic Kubernetes Service (EKS) 中运行（例如，作为 Percona Operator for MongoDB），它使用存储在 EKS 中与服务账户关联并分配给部署 Percona Backup for MongoDB 的 Pod 的 IAM 角色中的凭据访问 AWS S3 存储和其他服务。  

这使您无需显式创建和传递 AWS 凭据到 Pod，从而提高了部署的整体安全性。

要了解有关管理对 EKS 的访问的更多信息，请参阅[了解 EKS Pod Identity 如何授予 Pod 访问 AWS 服务的权限 :octicons-link-external-16:](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html)。

有关如何配置 Percona Operator for MongoDB 以使用 AWS S3 存储，请参阅[配置备份存储 :octicons-link-external-16:](https://docs.percona.com/percona-operator-for-mongodb/backups-storage.html#amazon-s3-or-s3-compatible-storage) 文档。





