# 安排备份

我们建议使用 `crond` 或类似服务来安排备份快照。

!!! important

    在配置 `crond` 之前，请确保您已[安装](../installation.md) 并[配置](../install/initial-setup.md) Percona Backup for MongoDB 以在数据库中创建备份。手动启动备份以验证这一点： 

    ```bash
    pbm backup
    ```

推荐的方法是创建 `/etc/cron.d` 目录中的 `crontab` 文件并在其中指定命令。这简化了服务器管理，特别是如果多个用户可以访问它。

`pbm` CLI 需要[有效的 MongoDB URI 连接字符串](../details/authentication.md) 才能在 MongoDB 中进行身份验证。我们建议创建环境文件并在其中指定 `export PBM_MONGODB_URI=$PBM_MONGODB_URI` 语句，而不是将 MongoDB URI 连接字符串指定为命令行参数，这存在潜在的安全风险。

作为示例，让我们配置在每周日 23:30 运行备份快照。
步骤如下：


1. 创建环境文件。让我们将其命名为 `pbm-cron`。


    === ":material-debian: Debian 和 Ubuntu"

        ```bash
        vim /etc/default/pbm-cron
        ``` 

    === ":material-redhat: Red Hat Enterprise Linux 和衍生版本"

        ```bash
        vim /etc/sysconfig/pbm-cron
        ```


2. 在 `pbm-cron` 中指定环境变量：
    
     ```bash
     export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27017/?replSetName=xxxx"
     ```

3. 授予将执行 `cron` 任务的用户对 `pbm-cron` 文件的访问权限。


4. 创建 `crontab` 文件。让我们将其命名为 `pbm-backup`。

     ```bash
     touch pbm-backup
     ```

5. 在文件中指定命令：

     ```bash
     30 23 * * sun <user-to-execute-cron-task> . /etc/default/pbm-cron; /usr/bin/pbm backup
     ```

    !!! note "" 
     
        注意环境文件前的点 `.`。它为其余 shell 命令提供（包含）环境文件。
 

6. 在 `/var/log/cron` 或 `/var/log/syslog` 日志中验证备份是否正在运行：

     ```bash
     grep CRON /var/log/syslog
     ```

## 在运行时间点恢复时安排备份

如果您启用了[时间点恢复](../features/point-in-time-recovery.md)，使用 `crond` 按计划自动创建备份很方便。

您可以按任何顺序配置时间点恢复和 `crond`。但是请注意，只有在至少创建了一个完整备份后，时间点恢复才会开始运行。

 * 手动创建新备份。它将作为增量备份的起点。
 * 启用时间点恢复。
 * 配置 `crond` 按计划运行备份快照。

当需要另一个备份快照时，Percona Backup for MongoDB 会自动禁用时间点恢复，并在备份完成后重新启用它。

## 备份存储清理

以前的备份不会自动从备份存储中删除。您需要定期删除最旧的备份以限制备份存储中使用的空间量。

!!! admonition "版本添加：[2.1.0](../release-notes/2.1.0.md)"

从版本 2.1.0 开始，您可以使用 [`pbm cleanup --older-than`](../reference/pbm-commands.md#pbm-cleanup) 命令删除过时的备份快照和时间点恢复 oplog 切片。在[清理过时数据](delete-backup.md#delete-outdated-data) 部分了解有关 PBM 如何删除过时数据的更多信息。

您可以通过在 `crontab` 文件中指定以下命令来配置 `cron` 任务以自动化存储清理：

```bash
/usr/bin/pbm cleanup -y --older-than 30d --wait
``` 

此命令删除超过 30 天的备份和 oplog 切片。您可以通过为 `--older-than` 标志指定所需间隔来更改时间段。 


!!! admonition ""

    对于 PBM 版本 2.0.5 及更早版本，请使用 [`pbm delete backup --older-than <timestamp>`](../reference/pbm-commands.md#pbm-delete-backup) 命令。您可以通过在 `crontab` 文件中指定以下命令来配置 `cron` 任务以自动化备份删除：
