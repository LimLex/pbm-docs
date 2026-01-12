# 远程备份存储选项

## 通用选项

### storage.type

*类型*：字符串 <br>
*必需*：    是   

远程备份存储类型。支持的值：`s3`、`minio`、`gcs`、`filesystem`、`azure`。

## AWS S3 存储选项

```yaml
storage:
  type: s3
  s3:
    region: <string>
    bucket: <string>
    prefix: <string>
    endpointUrl: <string>
    endpointUrlMap: 
      "node01:2017": <string>
      "node02:2017": <string>
    credentials:
      access-key-id: <your-access-key-id-here>
      secret-access-key: <your-secret-key-here>
      session-token: <string>
    uploadPartSize: <int>
    maxUploadParts: <int>
    storageClass: STANDARD
    serverSideEncryption:
      sseAlgorithm: aws:kms
      kmsKeyID: <your-kms-key-here>
      sseCustomerAlgorithm: AES256
      sseCustomerKey: <your_encryption_key>
    retryer:
      numMaxRetries: 3
      minRetryDelay: 30ms
      maxRetryDelay: 5m
    maxObjSizeGB: 5018
```

### storage.s3.provider


*类型*：字符串 <br>
*必需*：否

存储提供商的名称。此字段已弃用。


### storage.s3.bucket


*类型*：字符串 <br>
*必需*：是

存储桶的名称。有关存储桶名称要求，请参阅 [AWS 存储桶命名规则](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules)。

### storage.s3.region

*类型*：字符串 <br>
*必需*：是（对于 AWS）

存储桶的位置。
使用 [AWS 区域列表](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region) 定义存储桶区域

### storage.s3.prefix

*类型*：字符串 <br>
*必需*：否

存储桶中数据目录的路径。如果未定义，备份存储在存储桶的根目录中。

### storage.s3.endpointUrl

*类型*：字符串 <br>
*必需*：是（对于 MinIO）

访问存储桶的 URL。 

### storage.s3.endpointUrlMap

*类型*：字符串数组 <br>
*必需*：否

不同服务器上的 `pbm-agents` 到同一存储的自定义路径列表。如果 `pbm-agents` 位于隐藏在不同网络配置后面的服务器上，请使用此选项。在[支持同一 S3 存储的多个端点](../details/endpoint-map.md) 部分了解更多信息。支持 Amazon S3 和 Microsoft Azure Blob 存储。从版本 2.8.0 开始可用。

### storage.s3.forcePathStyle

*类型*：boolean <br>
*必需*：否

默认情况下，PBM 使用[路径样式 URL](https://docs.aws.amazon.com/AmazonS3/latest/userguide/VirtualHosting.html#path-style-access) 访问 S3 存储桶。将此选项设置为 `false` 指示 PBM 发送虚拟主机样式请求。

### storage.s3.credentials.access-key-id

*类型*：字符串 <br>
*必需*：是

您对存储桶的访问密钥。当您使用 EC2 实例配置文件运行 Percona Backup for MongoDB 时，可以省略此选项。要了解更多信息，请参阅[为 Percona Backup for MongoDB 自动化 S3 存储桶访问](../manage/automate-s3-access.md)

### storage.s3.credentials.secret-access-key

*类型*：字符串 <br>
*必需*：是

用于签署对存储桶的编程请求的密钥。当您使用 EC2 实例配置文件运行 Percona Backup for MongoDB 时，可以省略此选项。要了解更多信息，请参阅[为 Percona Backup for MongoDB 自动化 S3 存储桶访问](../manage/automate-s3-access.md)

### storage.s3.credentials.session-token

*类型*：字符串 <br>
*必需*：否

用于验证访问 S3 存储的[临时安全凭据](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) 的 AWS 会话令牌。 

### storage.s3.uploadPartSize

*类型*：int <br>
*必需*：否

要上传到存储桶的数据块大小（字节）。默认：10MB

如果要上传的文件大小超过最大允许文件大小，Percona Backup for MongoDB 会自动增加 `uploadPartSize` 值。（最大允许文件大小使用 `uploadPartSize` \* [`maxUploadParts`](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#pkg-constants) 的默认值计算，约为 97.6 GB）。

`uploadPartSize` 值打印在 `pbm-agent` 日志中。

通过设置此选项，如果 Percona Backup for MongoDB 由于某种原因无法执行，您可以手动调整数据块的大小。定义的 `uploadPartSize` 值会覆盖默认值，并用于计算最大允许文件大小

### storage.s3.maxUploadParts

*类型*：int <br>
*必需*：否 <br>
*默认*：10,000

要上传到存储桶的最大数据块数。默认：10,000

通过设置此选项，您可以覆盖 [AWS SDK](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#MaxUploadParts) 中定义的值。

在使用支持较少块数进行多部分上传的 S3 提供商时，这可能很有用。

`maxUploadParts` 值打印在 pbm-agent 日志中。

### storage.s3.storageClass

*类型*：字符串 <br>
*必需*：否
*示例*：STANDARD

分配给存储在 S3 存储桶中的对象的[存储类 :octicons-link-external-16:](https://aws.amazon.com/s3/storage-classes/)。如果未提供，将使用 `STANDARD` 存储类。 

### storage.s3.debugLogLevels

*类型*：字符串 <br>
*必需*：否

为不同类型的 S3 请求启用 S3 调试日志。S3 日志消息打印在 `pbm logs` 输出中。

从版本 2.10.0 开始，PBM 使用 AWS SDK v2。AWS SDK v1 值已弃用。它们保留用于向后兼容。

请查找下面的映射表：

| AWS SDK v1 值 | AWS SDK v2 值 |
|------------------|------------------|
| `LogDebug`       | `Request` <br> `Response`|
| `Signing`        | `Signing`|
| `HTTPBody`       | `RequestWithBody` <br> `ResponseWithBody`|
| `RequestRetries` | `DebugWithRequestRetries`|
| `RequestErrors`  | `DebugWithRequestErrors`|
| `EventStreamBody`| `RequestWithBody` <br> `ResponseWithBody`|


要指定多个事件类型，请用逗号分隔它们。要了解更多关于事件类型的信息，请参阅[文档 :octicons-link-external-16:](https://pkg.go.dev/github.com/aws/aws-sdk-go@v1.40.7/aws#LogLevelType)

未定义时，不执行 S3 调试日志。

### storage.s3.insecureSkipTLSVerify

*类型*：bool <br>
*必需*：否 <br>
*默认*：False

禁用 S3 存储的 TLS 验证。这允许 Percona Backup for MongoDB 将数据上传到使用自签名 TLS 证书的类似 S3 的存储。

!!! warning 
    
    请谨慎使用此选项，因为它可能为中间人攻击留下漏洞。

## 服务器端加密选项

### storage.s3.serverSideEncryption.sseAlgorithm

*类型*：字符串 <br>
*必需*：否 

用于服务器端加密的密钥管理模式，加密密钥存储在 AWS KMS 中。

支持的值：`aws:kms`

### storage.s3.serverSideEncryption.kmsKeyID

*类型*：字符串 <br>
*必需*：否

您存储在 AWS KMS 中的客户管理的密钥。

### storage.s3.serverSideEncryption.sseCustomerAlgorithm

*类型*：字符串 <br>
*必需*：否 

用于[使用客户提供的密钥进行服务器端加密 (SSE-C)](../details/s3-storage.md#server-side-encryption) 的密钥管理模式。

支持的值：`AES256`

### storage.s3.serverSideEncryption.sseCustomerKey

*类型*：字符串 <br>
*必需*：否

您的自定义加密密钥。此密钥不存储在 S3 存储端。因此，您有责任跟踪哪些数据使用哪些密钥加密以及存储密钥。 

## 上传重试选项

### storage.s3.retryer.numMaxRetries

*类型*：int <br>
*必需*：否 <br>
*默认*：3

上传数据到 S3 存储的最大重试次数。零值表示不执行重试。 

### storage.s3.retryer.minRetryDelay

*类型*：time.Duration <br>
*必需*：否 <br>
*默认*：30ms

下次重试前等待的最短时间，指定为 *time.Duration*。支持 ms、s 等单位。如果未提供单位，则默认为纳秒。 

### storage.s3.retryer.maxRetryDelay

*类型*：time.Duration <br>
*必需*：否 <br>
*默认*：5m

下次重试前等待的最长时间，指定为 *time.Duration*。支持 ms、s 等单位。如果未提供单位，则默认为纳秒。 

### storage.s3.maxObjSizeGB

*类型*：float64 <br>
*必需*：否 <br>
*默认*：5018

要存储在备份存储上的最大文件大小。如果要上传的文件超过此限制，PBM 会将其拆分为多个部分，每个部分都在限制范围内。了解更多关于[管理大型备份文件](../features/split-merge-backup.md) 的信息。

## MinIO 类型存储选项

您可以将此存储类型用于其他 S3 兼容存储服务

```yaml
storage:
  type: minio
  minio:
    region: <string>
    bucket: <string>
    prefix: <string>
    endpoint: <string>
    endpointMap: 
      "node01:2017": <string>
      "node02:2017": <string>
    secure: false
    insecureSkipTLSVerify: false
    forcePathStyle: false
    credentials:
      access-key-id: <string>
      secret-access-key: <string>
      session-token: <string>
      signature-ver: V4
    partSize: 10485760 (10 MB)
    retryer:
      numMaxRetries: 10
    maxObjSizeGB: 5018
    debugTrace: false 
```

### storage.minio.region

*类型*：字符串 <br>
*必需*：否

存储桶的位置。使用 [AWS 区域列表](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region) 定义存储桶区域。如果未指定，使用默认的 `us-east-1` 区域。

### storage.minio.bucket


*类型*：字符串 <br>
*必需*：是

存储桶的名称。有关存储桶名称要求，请参阅 [AWS 存储桶命名规则](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules)。

### storage.minio.prefix


*类型*：字符串 <br>
*必需*：否

存储桶中数据目录的路径。如果未定义，备份存储在存储桶的根目录中。

### storage.minio.endpoint

*类型*：字符串 <br>
*必需*：是

您的 MinIO 服务器可访问的网络地址（URL 或 IP:port）。

### storage.minio.endpointMap

*类型*：字符串数组 <br>
*必需*：否

不同服务器上的 `pbm-agents` 到同一 MinIO 存储的自定义端点映射。如果 `pbm-agents` 位于隐藏在不同网络配置后面的服务器上，请使用此选项。在[支持同一 S3 存储的多个端点](../details/endpoint-map.md) 部分了解更多信息。支持 Amazon S3、MinIO 和 Microsoft Azure Blob 存储。从版本 2.8.0 开始可用。

### storage.minio.secure

*类型*：boolean <br>
*必需*：否 <br>
*默认*：false

定义 PBM 和 S3 存储之间的通信是使用 HTTP 还是 HTTPS 协议。默认：`false`。

### storage.minio.insecureSkipTLSVerify

*类型*：boolean <br>
*必需*：否 <br>
*默认*：false

禁用 MinIO / S3 兼容存储的 TLS 验证。这允许 Percona Backup for MongoDB 将数据上传到使用自签名 TLS 证书的 MinIO / S3 兼容存储。请谨慎使用，因为它可能为中间人攻击留下漏洞。

### storage.minio.forcePathStyle

*类型*：boolean <br>
*必需*：否 <br>
*默认*：false

强制使用[路径样式访问](../reference/glossary.md#path-style-access-to-the-storage) 存储。默认为 `false`，这意味着 PBM 使用[虚拟主机样式](../reference/glossary.md#virtual-hosted-style-access) 访问存储

### storage.minio.credentials.access-key-id

*类型*：字符串<br>
*必需*：是

您对存储桶的访问密钥。

### storage.minio.credentials.secret-access-key

*类型*：字符串<br>
*必需*：是

用于签署对存储桶的编程请求的密钥。

### storage.minio.credentials.session-token

*类型*：字符串<br>
*必需*：否

用于验证访问存储的临时安全凭据的 MinIO 会话令牌。

### storage.minio.credentials.signature-ver

*类型*：字符串<br>
*必需*：否<br>
*默认*：V4

指定用于身份验证的 AWS Signature 版本。接受的值：`V2`、`V4`。 

允许使用已弃用的 AWS Signature 版本 2，以与不支持 Signature 版本 4 的存储向后兼容。默认：`V4`。

### storage.minio.partSize

*类型*：int<br>
*必需*：否

要上传到存储桶的数据块大小（字节）。默认：10MB。

### storage.minio.retryer.numMaxRetries

*类型*：int<br>
*必需*：否<br>
*默认*：10

上传数据到 MinIO / S3 兼容存储的最大重试次数。零值表示不执行重试。

### storage.minio.maxObjSizeGB

*类型*：float64<br>
*必需*：否<br>
*默认*：5018

要存储在备份存储上的最大文件大小。如果要上传的文件超过此限制，PBM 会将其拆分为多个部分，每个部分都在限制范围内。了解更多关于[管理大型备份文件](../features/split-merge-backup.md) 的信息。

### storage.minio.debugTrace

*类型*：boolean<br>
*必需*：否

如果设置为 `true`，在 PBM 日志中输出所有 http 通信跟踪。默认：false。

## GCS 类型存储选项

```yaml
storage:
 type: gcs
 gcs:
    bucket: pbm-testing
    chunkSize: <int>
    prefix: pbm/test
    credentials:
      clientEmail: <your-client-email-here>
      privateKey: <your-private-key-here>
      hmacAccessKey: <your-HMAC-key-here>
      hmacSecret: <your-HMAC-secret-here>
    maxObjSizeGB: 5018
```

### storage.gcs.bucket

*类型*：字符串 <br>
*必需*：是

存储桶的名称。有关存储桶名称要求，请参阅 [GCS 存储桶命名指南](https://cloud.google.com/storage/docs/naming-buckets#requirements)。

### storage.gcs.chunkSize

*类型*：字符串 <br>
*必需*：否

在单个请求中上传到存储桶的数据块大小（字节）。较大的数据块将拆分为多个请求。默认数据块大小为 10MB。

### storage.gcs.prefix

*类型*：字符串 <br>
*必需*：否

存储桶中数据目录的路径。如果未定义，备份存储在存储桶的根目录中。

### storage.gcs.credentials.clientEmail

*类型*：字符串 <br>
*必需*：是

在 GCS 中唯一标识您的服务账户的电子邮件地址。

### storage.gcs.credentials.privateKey

*类型*：字符串 <br>
*必需*：是

用于验证请求的服务账户的私钥。

### storage.gcs.credentials.hmacAccessKey

*类型*：字符串 <br>
*必需*：是

与您的服务账户关联的 HMAC 访问密钥。访问密钥用于通过 XML API 验证对 GCS 的请求。 

从版本 2.12.0 开始，HMAC 密钥的使用已弃用。请改用 `storage.gcs.credentials.clientEmail` 和 `storage.gcs.credentials.privateKey`。

### storage.gcs.credentials.hmacSecret

*类型*：字符串 <br>
*必需*：是

链接到特定 HMAC 访问 ID 的 40 字符 Base-64 编码字符串。创建 HMAC 密钥时会收到密钥。它用于在身份验证过程中创建签名。 

从版本 2.12.0 开始，HMAC 密钥的使用已弃用。请改用 `storage.gcs.credentials.clientEmail` 和 `storage.gcs.credentials.privateKey`。

### storage.gcs.retryer.backoffInitial

*类型*：time.Duration <br>
*必需*：否
*默认*：1s

进行初始重试前等待的时间，指定为 time.Duration。支持 ms、s 等单位。有效时间单位为 "ns"、"us"（或 "µs"）、"ms"、"s"、"m"、"h"。

如果未提供单位，则默认为纳秒。

### storage.gcs.retryer.backoffMax

*类型*：time.Duration <br>
*必需*：否
*默认*：30s

重试之间的最大时间（秒）。支持 ms、s 等单位。有效时间单位为 "ns"、"us"（或 "µs"）、"ms"、"s"、"m"、"h"。

如果未提供单位，则默认为纳秒。

### storage.gcs.retryer.backoffMultiplier

*类型*：int <br>
*必需*：否
*默认*：2

每次 PBM 失败并重试时，它会通过将此数字相乘来增加等待时间。默认值为 2 秒。

例如，如果第一次等待时间是 1 秒，下一次将是 2 秒，然后是 4 秒，依此类推，直到达到最大值。默认值为 2 秒。

### storage.gcs.retryer.maxAttempts

*类型*：int <br>
*必需*：否
*默认*：5

上传数据到 GCS 存储的最大重试次数。零值表示不执行重试。从版本 2.12.0 开始可用。

### storage.gcs.retryer.chunkRetryDeadline

*类型*：time.Duration <br>
*必需*：否
*默认*：32s

当您使用可恢复上传将大文件上传到 GCS 时，数据以块的形式发送。如果由于网络问题、超时或临时错误导致块上传失败，GCS 将重试发送该块。

`chunkRetryDeadline` 设置一个时间限制（秒），GCS 将在此时间内继续重试失败的块。一旦达到此截止时间，GCS 停止重试并将上传标记为失败。

支持 ms、s 等单位。有效时间单位为 "ns"、"us"（或 "µs"）、"ms"、"s"、"m"、"h"。

如果未提供单位，则默认为纳秒。

从版本 2.12.0 开始可用。

### storage.gcs.maxObjSizeGB

*类型*：float64 <br>
*必需*：否 <br>
*默认*：5018

要存储在备份存储上的最大文件大小。如果要上传的文件超过此限制，PBM 会将其拆分为多个部分，每个部分都在定义的限制范围内。了解更多关于[管理大型备份文件](../features/split-merge-backup.md) 的信息。

## 文件系统存储选项

```yaml
storage:
  type: filesystem
  filesystem:
    path: <string>
  maxObjSizeGB: 5018
```

### storage.filesystem.path

*类型*：字符串 <br>
*必需*：是

备份目录的路径

### storage.filesystem.maxObjSizeGB

*类型*：float64 <br>
*必需*：否 <br>
*默认*：5018

要存储在备份存储上的最大文件大小。如果要上传的文件超过此限制，PBM 会将其拆分为多个部分，每个部分都在定义的限制范围内。了解更多关于[管理大型备份文件](../features/split-merge-backup.md) 的信息。

## Microsoft Azure Blob 存储选项

```yaml
storage:
  type: azure
  azure:
    account: <string>
    container: <string>
    endpointUrl: <string>
    prefix: <string>
    credentials:
      key: <your-access-key>
    maxObjSizeGB: 194560
    retryer:
      numMaxRetries: 3
      minRetryDelay: 800ms
      maxRetryDelay: 60s
```

### storage.azure.account

*类型*：字符串 <br>
*必需*：是

您的存储账户的名称。

### storage.azure.container

*类型*：字符串 <br>
*必需*：是

存储容器的名称。有关命名约定，请参阅 [容器名称](https://docs.microsoft.com/en-us/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names)。

### storage.azure.endpointUrl

*类型*：字符串 <br>
*必需*：否

访问 Microsoft Azure Blob Storage 中数据的 URL。默认值为 `https://<storage-account>.blob.core.windows.net`。

### storage.azure.endpointUrlMap

*类型*：object (host:port -> endpoint URL) <br>
*必需*：否

不同服务器上的 `pbm-agents` 到同一远程存储的自定义端点 URL 映射。如果 `pbm-agents` 位于隐藏在不同网络配置后面的服务器上，请使用此选项。在[支持同一远程存储的多个端点](../details/endpoint-map.md) 部分了解更多信息。从版本 2.8.0 开始可用。


### storage.azure.prefix

*类型*：字符串 <br>
*必需*：否

存储桶中数据目录的路径。如果未定义，备份存储在存储桶的根目录中。

### storage.azure.credentials.key

*类型*：字符串 <br>
*必需*：是

您用于授权访问存储账户中数据的访问密钥。

### storage.azure.maxObjSizeGB

*类型*：float64 <br>
*必需*：否 <br>
*默认*：194560

要存储在备份存储上的最大文件大小。如果要上传的文件超过此限制，PBM 会将其拆分为多个部分，每个部分都在定义的限制范围内。了解更多关于[管理大型备份文件](../features/split-merge-backup.md) 的信息。

### storage.azure.retryer.numMaxRetries

*类型*：int <br>
*必需*：否 <br>
*默认*：3

上传数据到 Microsoft Azure 存储的最大重试次数。零值表示不执行重试。

### storage.azure.retryer.minRetryDelay

*类型*：time.Duration <br>
*必需*：否 <br>
*默认*：800ms

下次重试前等待的最短时间，指定为 `time.Duration`。支持 ms、s 等单位。如果未提供单位，则默认为纳秒。


### storage.azure.retryer.maxRetryDelay

*类型*：time.Duration <br>
*必需*：否 <br>
*默认*：60s

下次重试前等待的最长时间，指定为 `time.Duration`。支持 ms、s 等单位。如果未提供单位，则默认为纳秒。

## 阿里云 OSS 存储选项

```yaml
storage:
  type: oss
  oss:
    region: <string>
    endpointUrl: <string>
    bucket: <string>
    prefix: <string>
    credentials:
      accessKeyId: 'LTAI5t...EXAMPLE'
      accessKeySecret: 'n4VpW...EXAMPLE'
      securityToken: 'CAIS...EXAMPLE'
      roleArn: acs:ram::1234567890123456:role/db-backup-role
      sessionName: <string>
    serverSideEncryption:
      sseAlgorithm: <string>
      KMSMasterKeyID: <string>
      KMSDataEncryption: <string>
    uploadPartSize: <int>
    maxUploadParts: <int>
    connectTimeout: 5s
    maxObjSizeGB: 48700
    retryer:
      maxAttempts: 5
      maxBackoff: 300s
      baseDelay: 30ms
```

### storage.oss.region

*类型*：字符串 <br>
*必需*：是

您的 OSS 存储桶所在的区域。有关可用区域列表，请参阅 [OSS 区域和端点](https://www.alibabacloud.com/help/en/oss/user-guide/regions-and-endpoints)。

### storage.oss.endpointUrl

*类型*：字符串 <br>
*必需*：是

通过 Internet 访问 OSS 的域名。端点必须与创建存储桶时选择的区域相对应。 

### storage.oss.bucket

*类型*：字符串 <br>
*必需*：是

存储桶的名称。有关存储桶名称要求，请参阅 [OSS 存储桶命名规则](https://www.alibabacloud.com/help/en/oss/user-guide/bucket-naming-conventions)。

### storage.oss.prefix

*类型*：字符串 <br>
*必需*：否

存储桶中数据目录的路径。如果未定义，备份存储在存储桶的根目录中。

### storage.oss.credentials.accessKeyId

*类型*：字符串 <br>
*必需*：是

与用于访问阿里云 OSS 的 RAM 用户关联的 Access Key ID。

### storage.oss.credentials.accessKeySecret

*类型*：字符串 <br>
*必需*：是

与用于访问阿里云 OSS 的 RAM 用户关联的 Access Key Secret。密钥用于加密和验证签名字符串。

### storage.oss.credentials.securityToken

*类型*：字符串 <br>
*必需*：使用临时凭据时为是

与临时访问密钥 ID 和访问密钥 secret 一起使用的安全令牌。使用[安全令牌服务](https://www.alibabacloud.com/help/en/ram/product-overview/what-is-sts) 请求临时访问凭据时会收到令牌。

### storage.oss.credentials.roleArn

*类型*：字符串 <br>
*必需*：否

要承担的 RAM 角色的阿里云资源名称 (ARN)。PBM 使用此角色获取访问 OSS 资源所需的权限。

### storage.oss.credentials.sessionName

*类型*：字符串 <br>
*必需*：否

承担角色的会话标识符

### storage.oss.serverSideEncryption.sseAlgorithm

*类型*：字符串 <br>
*必需*：否 

用于在将数据存储在 OSS 中之前加密数据的加密算法。支持的值：`AES256`、`KMS`、`SM4`。默认：`AES256`

### storage.oss.serverSideEncryption.kmsMasterKeyId 

*类型*：字符串 <br>
*必需*：是（使用自定义 KMS 密钥时）

用于加密的客户主密钥的 ID。

### storage.oss.serverSideEncryption.kmsDataEncryption

*类型*：字符串 <br>
*必需*：否

使用 SSE-KMS 时用于加密数据的加密算法。仅在 `storage.oss.serverSideEncryption.sseAlgorithm` 设置为 `KMS` 时可以设置。

支持的值：`AES256`、`SM4`。默认：`AES256`

### storage.oss.uploadPartSize

*类型*：int <br>
*必需*：否

要上传到存储桶的数据块大小（字节）。默认：10MB

如果要上传的文件大小超过最大允许文件大小，Percona Backup for MongoDB 会自动增加 `uploadPartSize` 值。（最大允许文件大小使用 `uploadPartSize` \* [`maxUploadParts`](https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#pkg-constants) 的默认值计算，约为 97.6 GB）。

`uploadPartSize` 值打印在 `pbm-agent` 日志中。

通过设置此选项，如果 Percona Backup for MongoDB 由于某种原因无法执行，您可以手动调整数据块的大小。定义的 `uploadPartSize` 值会覆盖默认值，并用于计算最大允许文件大小

### storage.oss.maxUploadParts

*类型*：int <br>
*必需*：否 <br>
*默认*：10,000

要上传到存储桶的最大数据块数。默认：10,000

通过设置此选项，您可以覆盖 [Multipart upload](https://www.alibabacloud.com/help/en/oss/developer-reference/multipart-upload-3?spm=a3c0i.29367734.6737026690.4.43067d3faLVHMa) 方法中定义的值。

指定较少的块数进行多部分上传可能很有用。

`maxUploadParts` 值打印在 pbm-agent 日志中。


### storage.oss.connectTimeout

*类型*：int <br>
*必需*：否 <br>
*默认*：5s

PBM 连接到 OSS 存储时的连接超时（秒）。默认值为 5 秒。

### storage.oss.retryer.maxAttempts

*类型*：int <br>
*必需*：否 <br>
*默认*：300

对 OSS 存储的失败请求的最大重试尝试次数。默认值为 5。

### storage.oss.retryer.maxBackoff

*类型*：int <br>
*必需*：否 <br>
*默认*：300s

对 OSS 存储的失败请求的重试尝试之间的最大等待时间（秒）。默认值为 5 分钟（300 秒）。

### storage.oss.retryer.baseDelay

*类型*：int <br>
*必需*：否 <br>
*默认*：30ms

第一次重试尝试前的初始延迟。默认值为 30 毫秒。
