# 阿里云对象存储服务 (OSS)

如果您在亚太地区或中国运营和/或使用阿里云基础设施，您可以使用阿里云对象存储服务 (OSS) 作为 Percona Backup for MongoDB (PBM) 的远程备份存储。这样可以确保对备份的低延迟访问并优化成本。

要使用阿里云 OSS，您需要：

* 一个启用了对象存储服务的活跃阿里云账户。在[官方文档 :octicons-link-external-16:](https://www.alibabacloud.com/help/en/account/step-1-register-an-alibaba-cloud-account?spm=a2c63.l28256.0.i0) 中了解更多关于设置阿里云账户的信息

* 访问资源访问管理 (RAM) 控制台并具有创建和管理访问策略和用户的足够权限。在[官方文档 :octicons-link-external-16:](https://www.alibabacloud.com/help/en/oss/user-guide/how-oss-works-with-ram) 中了解更多关于将 RAM 与阿里云 OSS 一起使用的信息

## 创建存储桶

您可以通过 [阿里云管理控制台 :octicons-link-external-16:](https://home.console.aliyun.com/) 或通过命令行创建存储桶。 

=== ":simple-alibabacloud: 通过阿里云管理控制台"

    1. 登录阿里云管理控制台。
    2. 导航到对象存储服务 (OSS) 部分。
    3. 导航到存储桶并单击创建新存储桶。
    4. 根据需要指定存储桶名称、区域和其他设置。参考存储桶命名约定
    5. 单击**创建**，验证存储桶信息并单击**确认**。

=== ":material-console: 通过命令行"

    1. [安装](https://www.alibabacloud.com/help/en/oss/developer-reference/install-ossutil2#DAS) 并配置阿里云 OSS 客户端。安装后，`ossutil` 命令行工具可供您使用。
    2. 指定区域：

        ```bash
        ossutil config
        ```

        按 Enter 直到看到提示 `Please enter Region [cn-hangzhou]:` 并指定所需区域。

    3. 创建存储桶：

		```bash
		ossutil mb oss://your-bucket-name
		```

		将 `your-bucket-name` 替换为您存储桶的所需名称。

	4. 验证存储桶是否已创建：

	 	```bash
		ossutil ls
		```

创建存储桶后，为您计划与 PBM 一起使用的访问凭据标识的用户应用[必要的权限](storage-configuration.md#permissions-setup)。

## 为 PBM 配置对阿里云 OSS 的访问

为了使 PBM 成功访问和操作阿里云 OSS，它需要具有对指定 OSS 存储桶读写数据的必要权限的访问凭据。

阿里云 OSS 支持以下访问模式：

* 使用与 RAM 用户关联的 Access Key ID 和 Access Key secret。这些是专为编程访问设计的永久凭据。请注意，RAM 用户必须具有访问分配给他们的 OSS 资源的所有必要权限。 

   请参阅[使用 RAM 用户的 AccessKey 对访问 OSS 资源 :octicons-link-external-16:](https://www.alibabacloud.com/help/en/oss/developer-reference/use-the-accesskey-pair-of-a-ram-user-to-initiate-a-request) 章节以获取详细说明。

* 您可以通过 RAM 角色分配权限，而不是直接授予 RAM 用户。RAM 角色是一个虚拟身份，可以附加一个或多个访问策略，定义必要的权限。RAM 用户通过承担角色来获得这些权限。

  授权的 RAM 用户可以使用 AccessKey 对调用 [AssumeRole :octicons-link-external-16:](https://www.alibabacloud.com/help/en/ram/developer-reference/api-sts-2015-04-01-assumerole#main-107864) 操作。然后用户收到 STS 令牌以及临时凭据以访问 OSS 资源。

  请参阅 [STS 临时访问授权 :octicons-link-external-16:](https://www.alibabacloud.com/help/en/oss/sts-temporary-access-authorization#section-csx-hvf-vdb) 章节以获取配置指南。


## 配置示例

以下是 Percona Backup for MongoDB 中阿里云 OSS 配置的示例：

=== "使用 AccessKey 对"

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

=== "使用 RAM 角色"

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
         roleArn: acs:ram::1234567890123456:role/db-backup-role  
         sessionName: pbm-backup-session
    ```

有关配置选项的说明，请参阅[配置文件选项](../reference/configuration-options.md)。

## 微调存储配置

以下部分介绍如何微调存储配置：

* [服务器端加密](#server-side-encryption)
* [上传重试](#upload-retries)
* [同一 S3 存储的多个端点](endpoint-map.md) 

### 服务器端加密

阿里云 OSS 提供服务器端加密 (SSE) 功能以保护您的静态数据。启用 SSE 后，您的数据在存储前会自动加密，在访问时会自动解密。

Percona Backup for MongoDB 支持 OSS 存储桶的服务器端加密，支持以下加密类型：

* [阿里云 OSS 管理的加密密钥 (SSE-OSS)](#using-oss-managed-encryption-keys-sse-oss)。此类型提供基本加密功能。 
* [由阿里云密钥管理服务管理的客户主密钥 (SSE-KMS)](#using-customer-master-keys-managed-by-key-management-service-sse-kms)。此选项提供对密钥管理和安全的更多控制，适合需要使用自管理或用户指定的密钥以满足安全和合规要求的情况。

在[服务器端加密 :octicons-link-external-16:](https://www.alibabacloud.com/help/en/oss/user-guide/server-side-encryption-8) 文档中了解更多关于服务器端加密及其使用时的计费选项。

#### 先决条件

用于 PBM 访问阿里云 OSS 的 RAM 用户必须具有在存储桶上使用服务器端加密的必要权限。确保此用户的 RAM 策略包括以下操作：

1. 管理目标存储桶的权限。

2. `PutBucketEncryption` 和 `GetBucketEncryption` 权限。

3. 对于 SSE-KMS 加密类型，RAM 用户还必须具有以下权限：

   * `kms:Encrypt`
   * `kms:Decrypt`
   * `kms:GenerateDataKey`
   * `kms:DescribeKey`

在以下阿里云 OSS 文档中了解更多关于管理 RAM 策略的信息：

* [创建自定义 RAM 策略](https://www.alibabacloud.com/help/en/ram/user-guide/create-a-custom-policy#task-glf-vwf-xdb)
* [RAM 策略的常见示例](https://www.alibabacloud.com/help/en/oss/user-guide/common-examples-of-ram-policies) 
* [服务器端加密的权限](https://www.alibabacloud.com/help/en/oss/user-guide/server-side-encryption-8#section-oe2-ypt-1fi)

#### 使用 OSS 管理的加密密钥 (SSE-OSS)

使用 OSS 管理的密钥进行服务器端加密 (SSE-OSS) 是阿里云 OSS 的默认加密方法。阿里云 OSS 自动为每个对象生成加密密钥。它还创建主密钥以加密加密密钥。 

要配置 PBM 使用 SSE-OSS，请将以下选项添加到 `oss` 配置块：

```yaml
serverSideEncryption:
   sseAlgorithm: AES256
```

#### 使用密钥管理服务管理的客户主密钥 (SSE-KMS)

使用密钥管理服务管理的客户主密钥 (CMK) 进行服务器端加密 (SSE-KMS) 为您提供对密钥管理和安全的更多灵活性。 

您有以下选项：

* 使用 KMS 提供的默认客户主密钥。OSS 在 KMS 平台中创建此密钥并使用它加密数据 
* 使用 KMS 控制台生成您自己的客户主密钥。OSS 使用此指定密钥加密数据。

要配置 PBM 使用 SSE-KMS，请将以下选项添加到 `oss` 配置块：

```yaml
serverSideEncryption:
   sseAlgorithm: KMS
   kmsMasterKeyID: your-kms-key-id # when using a custom KMS key
   kmsDataEncryption: AES256
```

### 上传重试 

您可以设置 Percona Backup for MongoDB 上传数据到阿里云 OSS 的尝试次数以及等待下次重试的最小和最大时间。 

在 Percona Backup for MongoDB 配置中设置以下选项。

```yaml
retryer:
  maxAttempts: 5
  maxBackoff: 30
  baseDelay: 30
```

此上传重试增加了在不稳定连接情况下完成数据上传的机会。
