# 同一存储的多个端点

!!! admonition "版本添加：[2.8.0](../release-notes/2.8.0.md)" 

在 `pbm-agents` 运行在分布在多个数据中心的服务器上的环境中，访问相同的远程备份存储可能变得具有挑战性。这可能是由于复杂的网络配置或阻止直接连接到外部世界的严格策略。结果，`pbm-agents` 无法使用相同的 URL 到达存储，这对于 Percona Backup for MongoDB 正常工作来说是必需的。

为了解决这些挑战，您可以在 PBM 配置中为特定节点配置自定义端点 URL。这允许所有 `pbm-agents` 访问相同的存储，同时尊重其数据中心的网络设置。

支持的存储类型有： 

* AWS S3 
* MinIO 和 S3 兼容存储服务 
* Microsoft Azure Blob 存储 

以下是带有端点映射的配置文件示例：

=== ":fontawesome-brands-amazon: AWS S3"

    ```yaml
    storage:
        type: s3
        s3:
          endpointUrl: http://S3:9000
          endpointUrlMap:
            "node01:27017": "did.socf.s3.com"
            "node03:27017": "https://example.aws.s3.com"
          ...
    ```

=== ":simple-minio: MinIO 和 S3 兼容存储"

    ```yaml
    storage:
        type: minio
        minio:
          endpoint: localhost:9100
          endpointMap:
            "node01:27017": "did.socf.s3.com"
            "node03:27017": "example.min.io"
          ...
    ```

=== ":material-microsoft-azure: Microsoft Azure Blob 存储"

    ```yaml
    storage:
        type: azure
        azure:
          endpointUrl: https://myaccount.blob.core.windows.net
          endpointUrlMap:
            "node01:27017": "did.socf.blob.core.windows.net"
            "node03:27017": "example.azure.blob.core.windows.net"
          ...
    ```


您可以使用 `endpointUrlMap`（对于 AWS S3 和 Azure）或 `endpointMap`（对于 MinIO 和 S3 兼容存储）参数定义特定节点。这两个参数都是 `'host:port'` 到端点 URL 的映射，如前面的示例所示。映射中未列出的节点使用分别由 `endpointUrl` 或 `endpoint` 参数定义的端点。 

为了使解决方案工作，您还应该建立映射机制。此机制应该能够将自定义端点映射到存储的主端点 URL，将来自 `pbm-agents` 的请求无缝路由到存储并返回。

通过这种控制 `pbm-agents` 到达同一存储的端点的能力，您可以减少 PBM 配置的管理开销并确保其正常运行。 
