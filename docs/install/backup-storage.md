# 配置远程备份存储

提供远程备份存储配置的最简单方法是在 YAML 配置文件中指定它，然后使用 `pbm` CLI 将此文件上传到 Percona Backup for MongoDB。

存储配置本身不在本文档的范围内。我们假设您已经配置了支持的远程备份存储之一，并为 PBM 提供了具有适当权限的访问密钥。有关更多详细信息，请参阅[远程备份存储](../details/storage-configuration.md)。

## 注意事项

Percona Backup for MongoDB 需要自己专用的 S3 存储桶，专门用于备份相关文件。确保此存储桶仅由 PBM 创建和管理。

## 步骤 {.power-number}

1. 创建配置文件（例如 `pbm_config.yaml`）。您可以使用[模板配置文件 :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) 并根据需要进行修改。

    === ":material-aws: Amazon AWS"    

        ```yaml
        storage:
          type: s3
          s3:
            region: us-west-2
            bucket: pbm-test-bucket
            prefix: data/pbm/backup
            credentials:
              access-key-id: <your-access-key-id-here>
              secret-access-key: <your-secret-key-here>
        ```    

    === ":simple-minio: S3-compatible (MinIO)"    

        ```yaml
        storage:
          type: minio
          minio:
            endpoint: minio.example.com:9000
            bucket: pbm-test-bucket
            prefix: data/pbm/backup
            credentials:
              access-key-id: <your-access-key-id-here>
              secret-access-key: <your-secret-key-here>
        ```    

    === ":material-google-cloud: GCS (SA)"    

        ```yaml
        storage:
          type: gcs
          gcs:
             bucket: pbm-testing
             chunkSize: 16777216
             prefix: pbm/test
             credentials:
               clientEmail: <your-service-account-email>
               privateKey: <your-service-account-private-key-here>
        ```

    === ":material-google-cloud: GCS (HMAC) (deprecated)"    
   
        ```yaml
        storage:
          type: gcs
          gcs:
             bucket: pbm-testing
             prefix: pbm/test
             credentials:
               hmacAccessKey: <your-access-key-id-here>
               hmacSecret: <your-secret-key-here>
        ```

    === ":material-microsoft-azure: Microsoft Azure Blob Storage"    

        ```yaml
        storage:
          type: azure
          azure:
            account: <your-account>
            container: <your-container>
            prefix: pbm
            credentials:
              key: <your-access-key>
        ```    

    === "Alibaba Cloud Storage"

        ```yaml
        storage:
        type: oss
        oss:
          region: eu-central-1
          bucket: your-bucket-name
          endpointUrl: https://oss-eu-central-1.aliyuncs.com
          credentials:
            accessKeyID: "STS.****************"
            accessKeySecret:  "3dZn*******************************************"
        ```

    === ":material-file-tree: Shared local filesystem"    

        ```yaml
        storage:
          type: filesystem
          filesystem:
            path: /data/local_backups
        ```    

    导航到每个存储页面以获取详细的示例配置文件。

3. 将配置文件应用到 PBM

    ```bash
    pbm config --file pbm_config.yaml
    ```

要了解有关 Percona Backup for MongoDB 配置的更多信息，请参阅[集群（或非分片副本集）中的 Percona Backup for MongoDB 配置](../reference/config.md)。

## 下一步

[启动 `pbm-agent` :material-arrow-right:](start-pbm-agent.md){.md-button}
