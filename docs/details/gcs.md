# Google Cloud Storage (GCS)

您可以使用 Google Cloud Storage (GCS) 作为 Percona Backup for MongoDB 的远程备份存储。 

!!! admonition ""

    从版本 2.10.0 开始，PBM 使用 Google Cloud SDK 而不是 AWS SDK。升级后，请参阅如何[调整 PBM 配置以使用 GCS](#adjust-pbm-configuration-to-use-gcs)。


PBM 支持通过 JSON API 和 XML API 与 GCS 通信。首选方法是使用服务账户的 JSON API。HMAC 密钥主要用于与 S3 样式 API 的兼容性。

!!! warning "HMAC 密钥支持已弃用"

    从版本 2.12.0 开始，HMAC 密钥支持已弃用。我们鼓励您使用带有原生 JSON 密钥的 GCS 连接类型。

要使用 GCS，您需要以下内容：

* [创建服务账户 :octicons-link-external-16:](https://cloud.google.com/iam/docs/service-accounts-create#iam-service-accounts-create-console) 
* 为服务账户添加密钥：

    * [添加 JSON 密钥 :octicons-link-external-16:](https://cloud.google.com/iam/docs/keys-create-delete#creating) 或
    * [添加 HMAC 密钥 :octicons-link-external-16:](https://cloud.google.com/storage/docs/authentication/managing-hmackeys)。此方法已弃用，不建议使用

* [创建存储桶](#create-a-bucket)
* [将 GCS 配置添加到 PBM](#configuration-example) 

## 创建存储桶

1. 安装并配置 [gcloud CLI :octicons-link-external-16:](https://cloud.google.com/sdk/docs/install)

2. 创建存储桶

    ```bash
    gcloud storage buckets create my-gcs-bucket --location=US
    ```
      
3. 验证存储桶创建

    ```bash
    gcloud storage buckets list
    ```

创建存储桶后，应用适当的[权限以便 PBM 使用存储桶](storage-configuration.md#permissions-setup)。

## 配置示例

您可以找到[配置文件模板 :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) 并取消注释所需字段。

=== "使用 JSON 密钥"

    ```yaml
    storage:
     type: gcs
     gcs:
         bucket: pbm-testing
         prefix: pbm/test
         credentials:
           clientEmail: <your-service-account-email-here>
           privateKey: <your-private-key-here>
    ```

=== "使用 HMAC 密钥（已弃用）"

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

## 调整 PBM 配置以使用 GCS

从版本 2.10.0 开始，PBM 使用 Google Cloud SDK 而不是 AWS SDK。如果您从早期版本升级，需要按如下方式调整 PBM 配置：

1. 将 `storage.type` 从 `s3` 更改为 `gcs`。
2. 将 `storage.s3` 部分更改为 `storage.gcs` 并相应地调整参数。请参阅上面的[配置示例](#configuration-example)。根据您使用的身份验证方法选择选项。
