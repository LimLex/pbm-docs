# 远程备份存储

备份存储是任何数据库备份策略的关键组件。它作为 MongoDB 数据备份的安全、可靠位置，确保您的数据受到保护并在需要时可以恢复。备份存储的选择直接影响备份策略的可靠性、性能和成本效益。

备份存储有几个用途：

* 提供存储备份数据的安全位置
* 确保数据持久性和可用性
* 允许在不同环境之间传输备份数据

## 支持的存储类型

Percona Backup for MongoDB 支持以下存储类型：

* [Amazon S3](s3-storage.md)
* [Google Cloud Storage](gcs.md)
* [MinIO 和 S3 兼容存储](minio.md)
* [文件系统服务器存储](filesystem-storage.md)
* [Microsoft Azure Blob 存储](azure.md)
* [阿里云 OSS 存储](oss.md)

## PBM 如何在存储上组织备份

Percona Backup for MongoDB (PBM) 将备份数据保存到备份存储上的指定目录。它可以是您为存储定义的特定目录或根文件夹。 

每个备份都以 UTC 开始时间作为前缀，以便于识别，并包含：

* 包含备份信息的元数据文件
* 对于每个副本集：

  * 所有集合的压缩 mongodump 归档
  * 包含备份期间 oplog 条目的压缩 BSON 文件

Oplog 条目确保备份一致性，oplog 切片结束时间是备份快照的数据一致时间点。

使用 [`pbm list`](../reference/pbm-commands.md#pbm-list) 或 [`pbm status`](../reference/pbm-commands.md#pbm-status) 命令，您可以扫描备份目录以查找现有备份，即使您以前从未在计算机上使用过 PBM。

## 权限设置

无论您使用哪种远程备份存储，请为通过访问凭据标识的用户授予对此存储的 `List/Get/Put/Delete` 权限。

以下示例显示了 AWS S3 存储上 `pbm-testing` 存储桶的权限配置。

```json
{
    "Version": "2021-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::pbm-testing"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::pbm-testing/*"
        }
    ]
}
```

### 特定于存储的文档

请参阅您选择的存储的数据访问管理文档。

!!! admonition "另请参阅"

    * AWS 文档：[使用用户策略控制对存储桶的访问 :octicons-link-external-16:](https://docs.aws.amazon.com/AmazonS3/latest/userguide/walkthrough1.html)
    * Google Cloud Storage 文档：[访问控制概述 :octicons-link-external-16:](https://cloud.google.com/storage/docs/access-control)
    * Microsoft Azure 文档：[分配 Azure 角色以访问 blob 数据 :octicons-link-external-16:](https://docs.microsoft.com/en-us/azure/storage/blobs/assign-azure-role-data-access?tabs=portal)
    * MinIO 文档：[策略管理 :octicons-link-external-16:](https://docs.min.io/minio/baremetal/security/minio-identity-management/policy-based-access-control.html)
    * 阿里云文档：[权限和访问控制 :octicons-link-external-16:](https://www.alibabacloud.com/help/en/oss/user-guide/permissions-and-access-control)
