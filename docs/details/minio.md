# MinIO 和 S3 兼容存储

Percona Backup for MongoDB (PBM) 可与 AWS S3 和其他 S3 兼容存储服务配合使用。我们使用 [MinIO :octicons-link-external-16:](https://min.io/) 测试 PBM 的 S3 兼容存储服务

本文档提供 MinIO 作为最接近 S3 兼容存储的概述。要使用原生 AWS S3 服务，请参阅 [AWS S3 存储](s3-storage.md)。

[配置示例 :material-arrow-down:](#configuration-example){.md-button}

## 存储桶创建

1. 安装 [MinIO 客户端 :octicons-link-external-16:](https://min.io/docs/minio/linux/reference/minio-mc.html#install-mc)。安装后，`mc` 可供您使用。

2. 使用 MinIO 服务器配置 `mc` 命令行工具

    ```bash
    mc alias set myminio http://127.0.0.1:9000 MINIO_ACCESS_KEY MINIO_SECRET_KEY
    ```
    
3. 创建存储桶

    ```bash
    mc mb myminio/my-minio-bucket
    ```
      
4. 验证存储桶创建

   ```bash
   mc ls myminio
   ```

创建存储桶后，应用适当的[权限以便 PBM 使用存储桶](storage-configuration.md#permissions-setup)。

## 配置示例

!!! important
    
    Percona Backup for MongoDB (PBM) 需要自己的专用 S3 存储桶，专门用于备份相关文件。确保此[存储桶已创建](#bucket-creation) 并仅由 PBM 管理。

这是 Percona Backup for MongoDB 中 MinIO 和其他 S3 兼容存储服务的基本配置示例。您可以找到[配置文件模板 :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) 并取消注释所需字段。

```yaml
storage:
  type: minio
  minio:
    endpoint: localhost:9100
    bucket: pbm-example
    prefix: data/pbm/test
    credentials:
      access-key-id: <your-access-key-id-here>
      secret-access-key: <your-secret-key-here>
```

有关配置选项的说明，请参阅[配置文件选项](../reference/configuration-options.md)。

## 微调存储配置

以下部分介绍如何微调存储配置： 

* [调试日志](#debug-logging) 
* [上传重试](#upload-retries) 
* [使用自签名 TLS 证书上传数据到存储](#data-upload-to-storage-with-self-signed-tls-certificates)  
* [同一 S3 存储的多个端点](endpoint-map.md) 

### 调试日志

您可以为 Percona Backup for MongoDB 中不同类型的存储请求启用调试日志。Percona Backup for MongoDB 在 `pbm logs` 输出中打印日志消息，以便您可以调试和诊断存储请求问题或故障。

要启用调试日志，请在 Percona Backup for MongoDB 配置中设置 `storage.minio.debugTrace` 选项。这指示 PBM 还在日志中打印来自 MinIO 存储的 HTTP 跟踪。

## 上传重试 

您可以设置 Percona Backup for MongoDB 上传数据到 S3 存储的尝试次数。在 Percona Backup for MongoDB 配置中设置 `storage.minio.retryer.numMaxRetries` 选项。

```yaml
retryer:
  numMaxRetries: 3
```

此上传重试增加了在不稳定连接情况下完成数据上传的机会。

## 使用自签名 TLS 证书上传数据到存储

Percona Backup for MongoDB 支持通过 HTTPS 使用自签名或私有 CA 证书将数据上传到 S3 兼容存储服务。当您使用 MinIO、Ceph 或不使用公共证书颁发机构 (CA) 签名的证书的内部 S3 网关等服务时，此功能尤其重要。

建议提供完整的证书链以确保连接合法。`SSL_CERT_FILE` 环境变量指定 PBM 用于验证 TLS/SSL 连接的自定义证书链文件（PEM 格式）的路径。 

### 使用示例

假设您的自定义 CA 证书位于 `/etc/ssl/minio-ca.crt` 路径，您的 S3 端点是 `https://minio.internal.local:9000`。要使用自签发的 TLS 证书，请执行以下操作：

1. 确保证书文件为 PEM 格式。使用以下命令检查：

    ```bash
    cat /etc/ssl/minio-ca.crt
    ```

    ??? example "示例输出"


        ```{text .no-copy}
        -----BEGIN CERTIFICATE-----
        MIIC+TCCAeGgAwIBAgIJANH3WljB...
        -----END CERTIFICATE-----
        ```

2. 在运行 `pbm-agent` 和 PBM CLI 的每个主机上，将 `SSL_CERT_FILE` 环境变量设置为该文件的路径：

    ```bash
    export SSL_CERT_FILE=/etc/ssl/minio-ca.crt
    ```

    如果未设置此变量，PBM 使用系统根证书。

3. 重启 `pbm-agent`：

    ```bash
    sudo systemctl start pbm-agent
    ```

4. 验证您的自定义证书是否被识别。检查 PBM 日志以确认成功访问存储。 


或者，您可以在 Percona Backup for MongoDB 配置中关闭 S3 存储的 TLS 验证：

```bash
pbm config --set storage.minio.insecureSkipTLSVerify=True
```

!!! warning 

    请谨慎使用此选项，因为它可能为中间人攻击留下漏洞。

