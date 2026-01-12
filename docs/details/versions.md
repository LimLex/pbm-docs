# 支持的 MongoDB 版本

Percona Backup for MongoDB 与以下版本兼容：

* 启用了 [MongoDB 复制 :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/) 的 MongoDB Community / Enterprise Edition。Percona Backup for MongoDB 仅支持*逻辑备份*。
* Percona Server for MongoDB 支持所有[支持的备份类型](../features/backup-types.md)。对于逻辑备份，请确保启用了 [MongoDB 复制 :octicons-link-external-16:](https://docs.mongodb.com/manual/replication/)。对于物理和增量备份，请确保将 WiredTiger 设置为存储引擎。

请参考下面的兼容性矩阵，检查您当前的 PBM 版本支持哪些 MongoDB 版本进行备份和恢复。 

!!! note ""

    已停止支持的 MongoDB 版本可能与 PBM 一起工作，但未测试兼容性。请考虑升级到较新的 MongoDB 或 Percona Server for MongoDB 版本。 

## Percona Server for MongoDB 兼容性矩阵

下表列出了每种备份类型支持的 Percona Server for MongoDB 版本。

每个条目表示在支持的 Percona Server for MongoDB 版本中引入更改的 PBM 版本。

恢复兼容性列指示您是否可以从使用先前版本的 PBM 创建的备份恢复。

| PBM 版本 | 逻辑 | 物理 |增量物理 | 恢复向后兼容性|
| ----------- |---------|----------|---------------------|---------------------|
| **2.11.0 - {{release}}** | [7.0.x], [8.0.x] | [7.0.x], [8.0.x] | [7.0.x], [8.0.x] | 是 |
| **2.8.0 - 2.10.0**  | [6.0.x], [7.0.x], [8.0.x] | [6.0.x], [7.0.x], [8.0.x] | [6.0.x], [7.0.x], [8.0.x] | 是 |
| **2.6.0 - 2.7.0** | [5.0.x], [6.0.x], [7.0.x] | [5.0.x], [6.0.x], [7.0.x] | [5.0.14-12], [6.0.3-2] 及更高版本, [7.0.x] | 是 |
| **2.2.0 - 2.5.0** | [4.4.x] 及更高版本| [4.4.6-8] 及更高版本, 5.0.x, 6.0.x| [4.4.18-18], [5.0.14-12], [6.0.3-2] 及更高版本| 是 |
| **2.1.0** | 4.4.x 及更高版本 | [4.4.6-8], [5.0.x], [6.0.x]| [4.2.24-24], [4.4.18-18], [5.0.14-12], [6.0.3-2] 及更高版本| 否。需要新备份|
| **1.7.0** | 4.2 及更高版本| 技术预览: [4.2.15-16], [4.4.6-8], 5.0 及更高版本| | 是
| **1.6.1** | 3.6 及更高版本 | 不适用 |不适用 |不适用 |是

## MongoDB Community / Enterprise Edition 兼容性矩阵

此表列出了逻辑备份支持的 MongoDB Community 和 Enterprise Edition 版本。每个条目表示在支持的 MongoDB 版本中引入更改的新 PBM 版本。 

恢复兼容性列指示您是否可以从使用先前版本的 PBM 创建的备份恢复。

| PBM 版本 | 逻辑备份 | 恢复向后兼容性|
| ----------- |-----------------| ----------------------------- |
| **2.11.0 - {{release}}** | 7.0.x 及更高版本 | 是 |
| **2.8.0 - 2.10.0**  | 6.0.x 及更高版本 | 是 |
| **2.6.0 - 2.7.0** | 5.0.x 及更高版本 | 是 |
| **2.2.0 - 2.5.0** | 4.4.x 及更高版本 | 是 |
| **2.1.0** | 4.4.x 及更高版本| 否。需要新备份|
| **1.7.0** | 4.2 及更高版本| 是
| **1.6.1** | 3.6 及更高版本|是





[8.0.x]: https://docs.percona.com/percona-server-for-mongodb/8.0/
[7.0.x]: https://docs.percona.com/percona-server-for-mongodb/7.0/
[6.0.x]: https://docs.percona.com/percona-server-for-mongodb/6.0/
[6.0.3-2]: https://docs.percona.com/percona-server-for-mongodb/6.0/release_notes/6.0.3-2.html
[5.0.x]: https://docs.percona.com/percona-server-for-mongodb/5.0/
[5.0.14-12]: https://docs.percona.com/percona-server-for-mongodb/5.0/release_notes/5.0.14-12.html
[4.4.x]: https://docs.percona.com/percona-server-for-mongodb/4.4/
[4.4.18-18]: https://docs.percona.com/percona-server-for-mongodb/4.4/release_notes/4.4.18-18.html
[4.4.6-8]: https://docs.percona.com/percona-server-for-mongodb/4.4/release_notes/4.4.6-8.html
[4.2.24-24]: https://docs.percona.com/percona-server-for-mongodb/4.2/release_notes/4.2.24-24.html
[4.2.15-16]: https://docs.percona.com/percona-server-for-mongodb/4.2/release_notes/4.2.15-16.html
