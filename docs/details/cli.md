# PBM 命令行实用程序 (`pbm`)

`pbm` CLI 是您用来操作 Percona Backup for MongoDB 的命令行工具。`pbm` 提供您将在 shell 中手动使用的 **pbm** 命令。它也可以作为可以在脚本中执行的命令（例如，由 `crond` 执行）。

[pbm 子命令](../reference/pbm-commands.md) 集使您能够管理 MongoDB 环境中的备份。

`pbm` 使用 [PBM 控制集合](control-collections.md) 与 `pbm-agent` 进程通信。它通过更新和读取操作、日志等相应的 PBM 控制集合来启动和监控备份或恢复操作。同样，它通过将配置保存在用于配置值的 PBM 控制集合中来修改 PBM 配置。

`pbm` CLI 没有自己的配置和/或缓存文件。在 shell 中设置 `PBM_MONGODB_URI` 环境变量是一个类似配置的步骤，应该为了实际方便而完成。（如果没有 `PBM_MONGODB_URI`，每次都需要指定 `--mongodb-uri` 命令行参数。）

要了解如何设置 `PBM_MONGODB_URI` 环境变量，请参阅[为 `pbm` CLI 设置 MongoDB 连接 URI](../install/configure-authentication.md#set-the-mongodb-connection-uri-for-pbm-cli)。有关 MongoDB URI 连接字符串的更多信息，请参阅[身份验证](authentication.md)。
