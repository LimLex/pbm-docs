# 为 Percona Backup for MongoDB 自动化 S3 存储桶访问

当您使用 AWS 资源（在 EC2 实例上或使用 EKS）运行 MongoDB 和 Percona Backup for MongoDB 时，可以为 Percona Backup for MongoDB 自动化 AWS S3 存储桶访问。Percona Backup for MongoDB 使用 AWS 环境变量和元数据来访问 S3 存储桶，这样您就不必在 PBM 配置文件中明确指定 S3 凭据。因此，您可以从一个地方控制对云基础设施的访问。

## IAM 实例配置文件 

IAM（身份访问管理）是 AWS 服务，允许您安全地控制对 AWS 资源的访问。

使用 IAM 实例配置文件，您可以为在 EC2 实例上运行的 Percona Backup for MongoDB 自动化 S3 存储桶访问。步骤如下：

1. 创建 [IAM 实例配置文件 :octicons-link-external-16:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) 和其中的权限策略，在其中指定授予对 S3 存储桶访问的访问级别。

2. 将 IAM 配置文件附加到 EC2 实例。

3. 配置 S3 存储存储桶并验证从 EC2 实例到它的连接。


4. 在配置文件中为 PBM 提供[远程存储信息](../install/backup-storage.md)。将 `s3.credentials` 数组留空
    
    ```yaml
    storage:
      type: s3
      s3:
       region: <your-S3-region>
       bucket: <bucket-name>
    ```

    !!! note

        如果您指定 S3 凭据，它们会覆盖 EC2 实例环境变量和元数据，并用于身份验证。


5. 启动 `pbm-agent` 进程

!!! admonition "另请参阅"

    AWS 文档：[如何授予我的 Amazon EC2 实例访问 Amazon S3 存储桶的权限？ :octicons-link-external-16:](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-instance-access-s3-bucket/)

## IAM 服务账户角色 (IRSA)

!!! admonition "版本添加：2.0.3"

[IRSA :octicons-link-external-16:](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) 是 AWS EKS（Amazon Elastic Kubernetes Service）的本机方式，允许在 EKS pod 中运行的应用程序使用在 AWS IAM 角色中配置的权限访问 AWS API。

要从在 PBM 中使用 AWS IRSA 凭据中受益，高级步骤如下：

1. [创建集群 :octicons-link-external-16:](https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up-eks-cluster.html) 使用 `eksctl` 并启用 OIDC 提供程序设置。此功能适用于 EKS 集群版本 1.13 及更高版本。
2. 创建 IAM 角色并指定定义对 S3 存储桶访问的策略。
3. 创建服务账户并使用 IAM 角色对其进行注释。
3. 使用上一步创建的服务账户配置您的 pod 并承担 IAM 角色。
4. 在配置文件中为 PBM 提供远程存储信息。将 `s3.credentials` 数组留空，因为 PBM 使用 `AWS_ROLE_ARN`/`AWS_WEB_IDENTITY_TOKEN_FILE` 环境变量，这些变量要么自动提供（即由 EKS 中的 Kubernetes 变异准入控制器注入），要么您可以手动定义（如果您不希望准入控制器修改您的 pod）


!!! note 

    如果定义了 IRSA 相关凭据，它们优先于任何 IAM 实例配置文件。但是，如果您有意在 PBM 配置文件中指定 S3 凭据，它们会覆盖任何 IRSA/IAM 实例配置文件相关凭据，并用于身份验证。

!!! admonition "另请参阅"

    AWS 文档： 

    * [为服务账户引入细粒度 IAM 角色 :octicons-link-external-16:](https://aws.amazon.com/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/)
    * [如何使用 Amazon EKS 的 IAM 服务账户角色 (IRSA) 功能来限制对 Amazon S3 存储桶的访问？ :octicons-link-external-16:](https://aws.amazon.com/premiumsupport/knowledge-center/eks-restrict-s3-bucket/)



*[EC2]: Elastic Compute Cloud
*[EKS]: Elastic Kubernetes Service
*[IAM]: Identity Access Management
*[IRSA]: IAM Roles for Service Accounts
