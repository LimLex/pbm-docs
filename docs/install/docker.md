# 在 Docker 容器中运行 Percona Backup for MongoDB

Percona Backup for MongoDB 的 Docker 镜像托管在 [Docker Hub :octicons-link-external-16:](https://hub.docker.com/r/percona/percona-backup-mongodb) 上。

有关使用 Docker 的更多信息，请参阅 [Docker 文档 :octicons-link-external-16:](https://docs.docker.com/)。

确保您使用的是最新版本的 Docker。通过 apt 和 yum 提供的版本可能已过时并导致错误。

默认情况下，如果本地不可用，Docker 将从 Docker Hub 拉取镜像。

## 先决条件

* 您需要部署 MongoDB 或 Percona Server for MongoDB。请参阅[支持哪些 MongoDB 部署](../details/deployments.md)。
* 在您的部署中[创建 pbm 用户](configure-authentication.md#create-the-pbm-user)。您将需要此用户凭据来启动 Percona Backup for MongoDB 容器。 
* 对于物理备份，请确保使用包含 `mongod` 二进制文件以及 PBM 文件的容器。以下是 Dockerfile 示例：

   ```bash
   FROM percona/percona-server-mongodb:latest AS mdb
   FROM percona/percona-backup-mongodb:latest AS pbm 

   FROM oraclelinux:8 

   RUN mkdir -p /data/db /data/configdb 

   COPY --from=mdb /usr/bin/mongod /usr/bin/
   COPY --from=pbm /usr/bin/pbm* /usr/bin/
   ```

* 如果您使用专用的 PBM 容器，它也应该对 MongoDB 数据卷具有读写访问权限，并且您应该在 MongoDB 容器使用的相同用户 ID 下运行它。 

## 启动 Percona Backup for MongoDB 

使用以下命令启动 Percona Backup for MongoDB 容器：


```bash
docker run --name <container-name> -e PBM_MONGODB_URI="mongodb://<PBM_USER>:<PBM_USER_PASSWORD>@<HOST>:<PORT>" -d percona/percona-backup-mongodb:<tag>
```

其中：

* `container-name` 是您要分配给容器的名称。
* `PBM_MONGODB_URI` 是用于连接到 MongoDB 节点的 [MongoDB 连接 URI :octicons-link-external-16:](https://docs.mongodb.com/manual/reference/connection-string/) 字符串。请参阅[初始设置](initial-setup.md)如何创建 PBM 用户。 
* `tag` 是指定您需要的版本的标签。例如，`{{release}}`。Docker 识别架构（x86_64 或 ARM64）并拉取相应的镜像。请参阅[完整标签列表](https://hub.docker.com/r/perconalab/percona-backup-mongodb/tags)。

请注意，每个 MongoDB 节点（包括副本集从节点和配置服务器副本集节点）都需要单独的 Percona Backup for MongoDB 实例。因此，典型的 3 节点 MongoDB 副本集需要三个 Percona Backup for MongoDB 实例。

## 设置 Percona Backup for MongoDB 

Percona Backup for MongoDB 需要存储数据的远程存储。使用以下命令配置它：

1. 启动 Bash 会话：
	
    ```bash
    docker exec -it <container-name> bash
    ```

2. 创建 YAML 配置文件：

	```bash
	vi /tmp/pbm_config.yaml
	```
	
3. 在配置文件中指定远程存储参数。以下示例适用于 S3 兼容的备份存储。查看[支持的其他存储](../details/storage-configuration.md)：

	```yaml
	storage:
		type: s3
		s3:
		  region: <your-region-here>
		  bucket: <your-bucket-here>
	      credentials:
	        access-key-id: <your-access-key-id-here>
		secret-access-key: <your-secret-key-here>
	```

4. 上传配置文件： 
	
	```bash
	pbm config --file /tmp/pbm_config.yaml
	```
