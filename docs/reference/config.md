# 集群（或非分片副本集）中的 Percona Backup for MongoDB 配置

配置信息存储在 `admin.pbmConfig` 集合的单个文档中。该单个副本由集群（或非分片副本集）中的所有 `pbm-agent` 进程共享，可以使用 `pbm` CLI 工具读取或更新。

您可以通过运行以下命令查看整个配置：

```javascript
db.getSiblingDB("admin").pbmConfig.findOne()
```

但您不必使用 `mongo` shell；`pbm` CLI 有一个 "config" 子命令来读取和更新它。

Percona Backup for MongoDB 配置包含以下设置：

* [远程备份存储配置](configuration-options.md) 

* [时间点恢复配置](pitr-options.md) 

* [恢复选项](restore-options.md) 

* [日志选项](logging-options.md) 从版本 2.9.0 开始可用。


运行 [`pbm config --list`](../reference/pbm-commands.md#pbm-config) 查看整个配置。敏感字段（如密钥）将被编辑。

## 从 YAML 文件插入整个 Percona Backup for MongoDB 配置

如果您是第一次初始化集群或非分片副本集，最简单的方法是将整个配置编写为 YAML 文件，并使用
`pbm config --file` 命令在一个命令中上传所有值。

您可以使用[配置文件模板 :octicons-link-external-16:](https://github.com/percona/percona-backup-mongodb/blob/v{{release}}/packaging/conf/pbm-conf-reference.yml) 并取消注释所需字段。或者在编写自己的配置文件时将其用作示例。

配置文件必须具有远程备份存储配置。在存储部分查找每个支持的远程备份存储的配置文件示例。有关可用配置文件选项的更多信息，请参阅[配置文件选项](configuration-options.md)。

使用以下命令上传配置文件。例如，配置文件名为 `pbm_config.yaml`：

```bash
pbm config --file pbm_config.yaml
```

如果它是集群，请在连接到配置服务器副本集时执行命令。否则，正常连接到非分片副本集。（如果您还不熟悉 MongoDB 连接字符串，请参阅[MongoDB 连接字符串 - 提醒（或入门）](../details/authentication.md)。）

## 访问或更新单个配置值

您可以一次设置一个值。对于嵌套值，使用点连接的键名，如下例所示：

```sh
pbm config --set storage.s3.bucket="operator-testing"
```

您可以仅指定键名以列出单个值。如果已设置，命令返回该值。

=== "成功"

    ```sh
    pbm config storage.s3.bucket
    operator-testing
    ```

=== "无值"

    ```sh
    pbm config storage.s3.INVALID-KEY
    Error: unable to get config key: invalid config key
    ``` 

## 同步配置

当您在初始设置期间或进行更改后将配置文件上传到 PBM 时，PBM 会自动检测是否需要更新 PBM 控制集合中有关备份、恢复和时间点恢复块的本地元数据。 

例如，如果您更改存储配置，元数据也会更改，PBM 从存储导入它。但如果您仅启用/禁用了时间点恢复，元数据保持不变。

尽管 PBM 自动同步元数据，但在某些情况下您需要运行手动同步：

* 当您手动更改存储时。例如，您手动添加了备份或更改了备份路径。
* 作为物理恢复后的恢复后步骤。在将数据复制回 `mongod` 节点后，您必须手动触发从备份存储的元数据同步。  

要同步元数据，请在集群/副本集中运行以下命令：

```bash
pbm config --force-resync
```
