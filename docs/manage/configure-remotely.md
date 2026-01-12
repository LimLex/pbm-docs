# 通过管道配置 Percona Backup for MongoDB

!!! admonition "版本添加：2.0.1"

要应用或更新配置，Percona Backup for MongoDB 会读取文件系统上的配置文件。当您远程运行 PBM（在云中作为 Docker 容器或 Kubernetes 中的 pod）时，每次更新配置文件时都必须将其上传到远程主机的文件系统。  

为了简化配置管理，您可以在本地管理配置文件，并使用 UNIX 管道将文件内容传递给远程主机/在容器中运行的 Percona Backup for MongoDB。 

操作方法如下：

1. 创建/更新配置文件（例如，`/etc/pbm_config.yaml`）
2. 为配置文件路径创建环境变量

    ```bash
    export CONFIG_PATH="/etc/pbm_config.yaml"
    ```

3. 将配置文件内容传递给 Percona Backup for MongoDB。例如，如果您在 Docker 中运行 Percona Backup for MongoDB，请使用以下命令之一：
   
    * 连接到现有容器并传递配置：

        ```bash
        cat "$CONFIG_PATH" | docker compose exec -T $SERVICE_NAME pbm config --file="-"
        ```

        将 `$SERVICE_NAME` 替换为您的[服务名称](https://docs.docker.com/compose/compose-file/#services-top-level-element)。

    * 创建新容器以传递配置并退出： 

        ```bash
        cat "$CONFIG_PATH" | docker run -i --env PBM_MONGODB_URI="mongodb://<PBM_USER>:<PBM_USER_PASSWORD>@<HOST>:<PORT>" --network=$NET_ID $CONTAINER_ID pbm config --file="-"
        ```

        指定有效的 PBM_MONGODB_URI 连接字符串、容器将连接到的网络 ID 和容器 ID。

因此，您的 DBA 花费在管理 Percona Backup for MongoDB 上的时间更少，可以专注于其他活动。
