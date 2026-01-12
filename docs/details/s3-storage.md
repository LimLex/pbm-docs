# AWS S3 存储

Percona Backup for MongoDB (PBM) 可与 AWS S3 和其他 S3 兼容存储服务配合使用。我们使用以下服务测试 PBM：

* [Amazon Simple Storage Service :octicons-link-external-16:](https://docs.aws.amazon.com/s3/index.html)
* [MinIO :octicons-link-external-16:](https://min.io/)

本文档提供原生 AWS S3 服务的概述。要使用 MinIO 和其他 S3 兼容存储服务，请参阅 [S3 兼容存储](minio.md)。

[配置示例 :material-arrow-down:](#configuration-example){.md-button}


## 存储桶创建

要创建存储桶，请执行以下操作。

1. 安装并配置 [AWS CLI :octicons-link-external-16:](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

2. 创建 S3 存储桶

    ```bash
    aws s3api create-bucket --bucket my-s3-bucket --region us-east-1
    ```
  
3. 验证存储桶创建

    ```bash
    aws s3 ls
    ```

创建存储桶后，应用适当的[权限以便 PBM 使用存储桶](storage-configuration.md#permissions-setup)。

## 配置示例

!!! important
    
    Percona Backup for MongoDB (PBM) 需要自己的专用 S3 存储桶，专门用于备份相关文件。确保此[存储桶已创建](#storage-bucket-creation) 并仅由 PBM 管理。

这是 Percona Backup for MongoDB 中 AWS S3 存储的基本配置示例。您可以找到[配置文件模板 :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) 并取消注释所需字段。


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
    serverSideEncryption:
      sseAlgorithm: aws:kms
      kmsKeyID: <your-kms-key-here>
```

有关配置选项的说明，请参阅[配置文件选项](../reference/configuration-options.md)。

## 微调存储配置 

以下部分介绍如何微调存储配置： 

* [服务器端加密](#server-side-encryption) 
* [调试日志](#debug-logging) 
* [存储类](#storage-classes)
* [上传重试](#upload-retries) 
* [同一 S3 存储的多个端点](endpoint-map.md) 

### 服务器端加密

Percona Backup for MongoDB 支持 [S3 存储桶](../reference/glossary.md#bucket) 的[服务器端加密](../reference/glossary.md#server-side-encryption)，支持以下加密类型：

* [存储在 AWS KMS 中的客户提供的密钥 (SSE-KMS)](#using-aws-kms-keys-sse-kms)
* [存储在客户端的客户提供的密钥 (SSE-C)](#using-customer-provided-keys-sse-c)
* [Amazon S3 管理的加密密钥 (SSE-S3)](#using-amazon-s3-managed-keys-sse-s3)

####  使用 AWS KMS 密钥 (SSE-KMS)

要使用 SSE-KMS 加密，请在 Percona Backup for MongoDB 配置文件中指定以下参数： 

```yaml
serverSideEncryption:
   kmsKeyID: <kms_key_ID>
   sseAlgorithm: aws:kms
```  

#### 使用客户提供的密钥 (SSE-C)

!!! admonition "版本添加：[2.0.1](../release-notes/2.0.1.md)" 

Percona Backup for MongoDB 还支持使用存储在客户端的客户提供的密钥进行服务器端加密 (SSE-C)。Percona Backup for MongoDB 将加密密钥作为对 S3 存储的请求的一部分提供。S3 存储使用它们通过 `AES-256` 加密算法加密/解密数据。通过这种方式，您可以节省订阅 AWS KMS 服务的费用，并可以使用您选择的 S3 兼容存储进行服务器端加密。

!!! warning

    1. 仅对空存储桶启用/禁用服务器端加密。否则，Percona Backup for MongoDB 无法正确保存/检索存储中的对象。
    2. S3 存储不管理或存储加密密钥。您有责任跟踪哪个密钥用于加密存储桶中的哪个对象。如果您丢失密钥，任何没有加密密钥的对象请求都会失败，您将丢失该对象。 

要使用 SSE-C 加密，请在 Percona Backup for MongoDB 配置文件中指定以下参数：    

```yaml
serverSideEncryption:
  sseCustomerAlgorithm: AES256
  sseCustomerKey: <your_encryption_key>
``` 

#### 使用 Amazon S3 管理的密钥 (SSE-S3)

!!! admonition "版本添加：[2.6.0](../release-notes/2.6.0.md)" 

Percona Backup for MongoDB 支持使用 Amazon S3 管理的密钥进行服务器端加密 (SSE-S3)，这是 Amazon AWS S3 中的默认加密方法。Amazon S3 使用唯一密钥加密每个对象。作为额外的安全措施，它使用定期轮换的密钥加密密钥本身。Amazon S3 服务器端加密使用 256 位高级加密标准 Galois/Counter Mode (AES-GCM) 加密所有上传的对象。

要使用 SSE-S3 加密，请在 Percona Backup for MongoDB 配置文件中指定以下参数：

```yaml
serverSideEncryption:
   sseAlgorithm: AES256
```  

### 调试日志

您可以为 Percona Backup for MongoDB 中的不同类型的 S3 请求启用调试日志。Percona Backup for MongoDB 在 `pbm logs` 输出中打印 S3 日志消息，以便您可以调试和诊断 S3 请求问题或故障。

要启用 S3 调试日志，请在 Percona Backup for MongoDB 配置中设置 `storage.s3.DebugLogLevel` 选项。支持的值有：`LogDebug`、`Signing`、`HTTPBody`、`RequestRetries`、`RequestErrors`、`EventStreamBody`。

### 存储类 

Percona Backup for MongoDB 支持 [Amazon S3 存储类 :octicons-link-external-16:](https://aws.amazon.com/s3/storage-classes/)。了解您的数据访问模式后，您可以在 Percona Backup for MongoDB 配置中设置 S3 存储类。当 Percona Backup for MongoDB 将数据上传到 S3 时，数据会分发到相应的存储类。对 S3 存储桶存储类型的支持使您能够有效管理 S3 存储空间和成本。

要设置存储类，请在 Percona Backup for MongoDB 配置文件中指定 `storage.s3.storageClass` 选项：

```yaml
storage:
  type: s3
  s3:
    storageClass: INTELLIGENT_TIERING
```

当选项未定义时，使用 S3 Standard (`STANDARD`) 存储类型。

### 上传重试 

您可以设置 Percona Backup for MongoDB 上传数据到 S3 存储的尝试次数以及等待下次重试的最小和最大时间。在 Percona Backup for MongoDB 配置中设置选项 `storage.s3.retryer.numMaxRetries`、`storage.s3.retryer.minRetryDelay` 和 `storage.s3.retryer.maxRetryDelay`。

```yaml
retryer:
  numMaxRetries: 3
  minRetryDelay: 30
  maxRetryDelay: 5
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

4. 验证您的自定义证书是否被识别。检查 PBM 日志以确认成功访问 S3。 


或者，您可以在 Percona Backup for MongoDB 配置中禁用 S3 存储的 TLS 验证：

```bash
pbm config --set storage.s3.insecureSkipTLSVerify=True
```

!!! warning 

    请谨慎使用此选项，因为它可能为中间人攻击留下漏洞。

