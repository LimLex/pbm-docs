# 查看恢复进度

!!! admonition "版本添加：[2.0.0](../release-notes/2.0.0.md)"

您可以跟踪物理和逻辑恢复的状态。这使您清楚地了解恢复进度，以便您可以相应地做出反应。 

要查看恢复状态，请运行 `pbm describe-restore` 命令并指定恢复名称。要跟踪物理恢复进度，请指定 Percona Backup for MongoDB (PBM) 配置文件的路径。由于 `mongod` 节点在物理恢复期间关闭，Percona Backup for MongoDB 使用配置文件读取存储上的恢复状态。当配置了[多个备份存储](../features/multi-storage.md) 时，PBM 仅在主存储上存储恢复元数据。 

```bash
pbm describe-restore 2022-08-15T11:14:55.683148162Z -c pbm_config.yaml
```

输出提供以下信息：

-  恢复名称
-  从中恢复数据库的备份名称
-  类型
-  状态
-  opID
-  恢复开始时间
-  恢复完成时间（对于成功的恢复）
-  最后转换时间 – 恢复过程更改其状态的时间
-  每个副本集的名称、其恢复状态和最后转换时间 

仅对于物理备份，提供以下附加信息：

- 节点名称
- 节点上的恢复状态
- 最后转换时间

请参阅 [pbm describe-restore](../reference/pbm-commands.md#output_1) 以获取字段的完整列表及其描述。
