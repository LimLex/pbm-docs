# 通过 Percona Monitoring and Management 排查备份管理问题

如果您在通过 PMM 进行物理备份和恢复时遇到问题，请按照以下故障排除步骤操作：

1. 确保运行 `pmm` 进程的用户在 PBM 内具有足够的权限。您可以在 [PMM 文档 :octicons-link-external-16:](https://docs.percona.com/percona-monitoring-and-management/3/install-pmm/install-pmm-client/connect-database/mongodb.html#create-pmm-account-and-set-permissions) 中找到详细说明。
2. 确保您在 `pbm-agent` 进程的 MongoDB 连接 URI 字符串中指定了步骤 1 中的用户。
3. 确认运行 `pbm-agent` 进程的系统用户（默认情况下为 `mongod`）对 MongoDB dbpath 和备份存储位置都具有读/写访问权限。
4. 检查步骤 4 中提到的用户的 `systemd` 重启策略。确保它未设置为 `always` 或 `on-success`。在物理恢复期间，数据库不得自动重启，因为这由 `pbm-agent` 控制。


