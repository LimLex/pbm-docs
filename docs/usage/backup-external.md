# 创建基于快照的备份

--8<-- "prepare-backup.md"

## 步骤

1. 要创建基于快照的备份，请使用类型 `external` 运行 [`pbm backup`](../reference/pbm-commands.md#pbm-backup) 命令：

    ```bash
    pbm backup -t external 
    ```    

    执行命令时，PBM 执行以下操作：    

    * 打开 `$backupCursor`
    * 准备数据库以进行文件复制
    * 在存储上存储备份元数据并将其添加到要复制的文件
    * 打印类似于以下的提示：    

       ```{.text .no-copy}
       Ready to copy data from:
       <node-list>
       ```    

    您还会看到备份名称。 

2. （可选）您可以使用 [`pbm describe-backup`](../reference/pbm-commands.md#pbm-describe-backup) 检查备份进度。命令输出提供备份状态以及哪些节点正在运行备份。

3. 在此阶段，您需要将 `<node-list>` 中每个节点的 `dataDir` 内容复制到存储/使用您选择的技术创建快照。 

4. 复制/快照完成后，运行以下命令以关闭 `$backupCursor` 并完成备份： 

    ```bash
    pbm backup-finish <backup_name>
    ```

## 下一步

[列出备份](../usage/list-backup.md){.md-button}
[进行恢复](restore-external.md){.md-button}

## 有用的链接

* [备份和恢复类型](../features/backup-types.md)
* [安排备份](../usage/schedule-backup.md)

