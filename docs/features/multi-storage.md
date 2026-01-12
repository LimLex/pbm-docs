# 多个备份存储

!!! admonition "版本添加：[2.6.0](../release-notes/2.6.0.md)"

备份策略中的一个良好做法是遵循 3-2-1 规则：在 2 个不同的存储上存储 3 份数据副本，并保留 1 份异地副本。您无需将数据传输到多个存储，而是可以配置这些存储并让 PBM 直接在不同时间表上向它们创建备份。 

例如，您可以按如下方式配置备份：

* 每周创建启用时间点恢复的完整物理备份，并将它们存储在 AWS S3 存储桶上， 
* 每天在另一个存储上创建增量备份。
* 每月创建整个数据库的 EBS 快照并将其存储在外部异地服务器上。

这种为备份定义多个存储的能力带来以下好处：

* 在云存储的情况下节省数据传输成本
* 通过您自己的与 PBM 接口的应用程序和工具或通过 Percona Everest 提高遵循组织备份策略的有效性

## 配置配置文件 

默认情况下，PBM 将备份和时间点恢复 oplog 切片存储到您在初始设置期间在配置文件中定义的远程备份存储。这是**主**备份存储。

要向额外的 — **外部**备份存储创建备份，引入了配置配置文件的概念。配置配置文件是仅存储外部备份存储配置的文件。

以下是配置配置文件的示例：

```yaml title="minio.yaml"
storage:
  type: s3
  s3:
	endpointUrl: "http://localhost:9000"
	bucket: mybucket
	region: my-region
	credentials:
		access_key_id: myaccesskey
		secret_access_key: mysecretkey
	
```

要将配置配置文件上传到 PBM，请使用 [`pbm profile add`](../reference/pbm-commands.md#pbm-profile-add) 命令并指定配置文件的路径。

```bash
pbm profile add <profile_name> /path/to/profile.yaml
```

要显示有关外部备份存储的信息，请使用 [`pbm profile show`](../reference/pbm-commands.md#pbm-profile-show) 命令：

```bash
pbm profile show <profile_name>
```

请参阅 [pbm 命令](../reference/pbm-commands.md) 参考中的配置配置文件管理命令的完整列表。

## 向外部存储创建备份

要向外部备份存储创建备份，请为 `pbm backup` 命令使用 `--profile` 标志传递配置文件名称。例如，要运行物理备份并将其存储在通过 `minio` 配置配置文件定义的 MinIO 存储中，请运行以下命令：

```bash
pbm backup -t physical --profile=minio --wait 
```

??? example "示例输出"

    ```{.text .no-copy}
    Starting backup '2024-06-25T11:25:30Z'....Backup '2024-06-25T11:25:30Z' to remote store 's3://http://minio:9000/backup' has started
	```

您可以在外部存储上创建除基于快照的备份之外的任何类型的备份。

请注意，对于向外部存储创建的备份，时间点恢复 oplog 切片不会自动停止。因此，PBM 在主存储和外部存储上保存与此类备份相关的 oplog 块。

当您创建增量备份时，请确保将整个备份链保存在同一存储上。要切换备份存储，您必须在其上创建新的基础备份以开始新的增量链。 

## 从外部存储恢复

在开始之前，请确保 `pbm-agents` 对远程存储上的备份具有读取权限。另外，[进行恢复的所有准备工作](../usage/restore.md#before-you-start)。

1. 通过运行 `pbm list` 或 `pbm status` 命令列出备份。
    
    ```bash
	pbm list
	```

	输出显示备份名称和时间戳。外部备份用星号标记：

	??? example "示例输出"

	    ```{.text .no-copy}
	    Backup snapshots:
	      2024-06-25T10:53:57Z <logical> [restore_to_time: 2024-06-25T10:54:02Z]
	      2024-06-25T10:54:55Z <logical, *> [restore_to_time: 2024-06-25T10:55:02Z]
	      2024-06-25T10:57:49Z <logical, *> [restore_to_time: 2024-06-25T10:57:56Z]

	    PITR <on>:
	      2024-06-25T10:54:03Z - 2024-06-25T10:57:51Z
	    ```

2. 要进行时间点恢复，您必须为 `pbm restore` 命令明确传递备份名称：

    ```bash
    pbm-restore --time=<timestamp> --base-snapshot <backup-name>
    ```

3. 恢复完成后，根据恢复类型执行所需的恢复后步骤。
4. 创建新备份作为未来恢复的新基础。 

## 删除备份

您只能按名称从外部存储删除备份。 

运行 `pbm delete` 命令并传递备份名称：

```bash
pbm delete-backup 2024-06-25T10:54:55Z
```

## 实现细节

1. 您可以在外部存储上创建除基于快照的备份之外的任何类型的备份。
2. 要启动时间点恢复 oplog 切片，您必须在主存储上创建备份。来自外部存储的备份不被视为 oplog 切片的有效基础备份。
3. PBM 仅在主存储上保存时间点恢复 oplog 范围。备份保存在启动备份时定义的存储上。 
4. 外部存储上的备份过程不会停止主存储上的时间点恢复 oplog 切片。因此，PBM 在主存储和外部存储上保存与此类备份相关的 oplog 块
5. 整个增量链必须存储在同一存储上。要更改增量备份的存储，您必须在新存储上使用增量基础备份启动新的备份链。
6. 要从外部存储上的备份恢复，`pbm-agents` 必须对其具有读取权限。
7. 要进行时间点恢复，您必须通过 `--base-snapshot` 标志指定备份名称。如果没有它，PBM 会在主存储上搜索基础备份。
8. 您只能使用 `pbm delete-backup <backup-name>` 命令按名称从外部存储删除备份。 
9. 您只能使用 `pbm delete-backup --older-than <time>` 或 `pbm cleanup --older-than <time>` 命令从**主**存储删除早于指定时间的备份。 



