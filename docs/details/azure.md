# Azure Blob 存储

!!! admonition "版本添加：[1.5.0](../release-notes/1.5.0.md)"

使用基于 Microsoft 的基础设施的公司可以通过使用 [Microsoft Azure Blob Storage :octicons-link-external-16:](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) 作为远程备份存储，以更少的管理工作设置 Percona Backup for MongoDB。
 
## 创建 Blob 存储

您可以通过 Azure Portal Web 界面或使用 Azure CLI 创建 Blob 存储。

对于任一方法，您都需要一个存储账户。

=== "Azure Portal"

    1. 登录 Azure Portal
    2. 在主页上，选择**存储账户**。
    3. 从工具栏单击**创建**，使用向导创建存储账户。
    4. 创建存储账户后，从列表中选择它。
    5. 在左侧菜单中，选择**数据存储** -> **容器**。
    6. 单击 **+ 容器** 创建新容器。
    7. 输入容器名称，选择**私有**作为访问级别。
    8. 单击**创建**创建容器。

=== "Azure CLI"

    1. 安装 [Azure CLI :octicons-link-external-16:](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)。安装后，`az` 可供您使用。
    2. 登录 Azure CLI：

        ```bash
        az login
        ```

    3. 如果未为您创建资源组，请创建一个：

        ```bash
        az group create --name <your-resource-group> --location <your-location>
        ```

        要获取可用位置列表，请运行：

        ```bash
        az account list-locations
        ```

    4. 创建存储账户：

        ```bash
        az storage account create --name <storage-account-name> --resource-group <your-resource-group> --location <your-location> --sku Standard_LRS
        ```

    4. 创建 Blob 容器：

        ```bash
        az storage container create --account-name <storage-account-name> --name <your-container>  --public-access off
        ```

        ??? example "预期输出"

            ```{.json .no-copy}
            {
              "created": true
            }
            ```

创建存储桶后，应用适当的[权限以便 PBM 使用存储桶](storage-configuration.md#permissions-setup)。

## 配置示例

您可以找到[配置文件模板 :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) 并取消注释所需字段。

```yaml
storage:
  type: azure
  azure:
    account: <your-storage-account>
    container: <your-container>
    prefix: pbm
    credentials:
      key: <your-access-key>
```

有关配置选项的说明，请参阅[配置文件选项](../reference/configuration-options.md)。


## 上传重试 

您可以设置 Percona Backup for MongoDB 上传数据到 Microsoft Azure 存储的尝试次数以及等待下次重试的最小和最大时间。在 Percona Backup for MongoDB 配置中设置选项 `storage.azure.retryer.numMaxRetries`、`storage.azure.retryer.minRetryDelay` 和 `storage.azure.retryer.maxRetryDelay`。

```yaml
retryer:
  numMaxRetries: 3
  minRetryDelay: 800ms
  maxRetryDelay: 60s
```

此上传重试增加了在不稳定连接情况下完成数据上传的机会。
