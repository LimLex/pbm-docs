# 诊断报告

!!! admonition "版本添加：[2.8.0](../release-notes/2.8.0.md)"

在排查备份和恢复问题时，查看日志和 PBM 状态有时可能不足以识别问题的根源。 

从版本 2.8.0 开始，您可以生成关于特定备份、恢复或其他命令的诊断报告。报告包括以下信息：

* 有关环境的信息：pbm-agents 状态、集群成员等。 
* 在命令执行的开始和结束时间之间收集的日志
* 如果是备份命令，备份元数据文件。
* 如果是恢复命令，恢复元数据文件和备份元数据文件。 

   <i warning>:material-alert: 警告：</i> 目前不支持物理恢复。

此数据以 JSON 格式存储在单独的文件中。

要生成报告，请运行 `pbm diagnostics` 命令：

```bash
pbm diagnostic --path=path --name=<backup-name> 
```

或者您可以使用命令的 OPID：

```bash
pbm diagnostic --path=path --opid=<OPID> 
```

其中：

* `path` 是保存报告的路径。如果目录不存在，PBM 在报告生成期间创建它。确保运行 PBM CLI 的用户对指定路径具有写入访问权限。
* `OPID` 是指定命令的唯一操作 ID。您可以从 `pbm logs`、`pbm describe-backup` / `pbm describe-restore` 输出中检索它。 
* `name` 是所需备份或恢复的名称。您可以将其用于备份和恢复，而不是 OPID。

**使用示例**

例如，您的 `pbm status` 输出具有以下备份：

```{.text .no-copy}
Backups:
========
S3 us-east-1 http://minio:9000/mybackups
  Snapshots:
    2024-11-27T13:49:31Z 95.79KB <logical> [restore_to_time: 2024-11-27T13:49:37Z]
```

要检索备份操作的 OpID，请按如下方式运行 `pbm describe-backup` 命令：

```bash
pbm describe-backup 2024-11-27T13:49:31Z | grep 'opid'
```

输出返回 OpID：

```{.text .no-copy}
opid: 6747236bfa98f6a85b9bd4e7
```

现在您可以生成诊断报告：

```bash
pbm diagnostic --path=/tmp/backup_report --opid=6747236bfa98f6a85b9bd4e7
```

检查生成的文件：

```bash
ls /tmp/backup_report
6747236bfa98f6a85b9bd4e7.backup.json  6747236bfa98f6a85b9bd4e7.log  6747236bfa98f6a85b9bd4e7.report.json
```

您可以使用 OPID 生成关于其他操作（如清理、取消等）的诊断报告。在这种情况下，报告仅包含有关您的环境和在操作执行期间收集的日志的信息。

您还可以将报告输出到归档文件，如下所示：

```bash
pbm diagnostic --path=path --opid=<OPID> --archive
``` 

诊断报告使您能够一次性收集深入分析特定操作问题所需的每个必要方面。如果您无法自己执行分析，`pbm diagnostic` 提供了一种快速便捷的方法来收集和提交所有相关信息以提交错误报告。这显著减少了您和我们的专家之间的交互时间，加速了问题解决。

Percona 客户的优势在于他们的错误报告优先级更高。如果您有兴趣享受这些好处，[立即联系我们 :octicons-link-external-16:](https://www.percona.com/about/contact)。
