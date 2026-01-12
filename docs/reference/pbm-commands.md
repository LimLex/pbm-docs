# `pbm` 命令

`pbm` CLI 是用于控制备份系统的命令行实用程序。本页介绍 Percona Backup for MongoDB 中可用的 `pbm` 命令。

有关如何开始使用 Percona Backup for MongoDB，请参阅[初始设置](../install/initial-setup.md)。

## pbm backup

创建备份快照并将其保存在远程备份存储中。

该命令具有以下语法：

```bash
pbm backup [<flags>]
```

有关使用 `pbm backup` 的更多信息，请参阅[启动备份](../usage/start-backup.md)

该命令接受以下标志：

| Flag           | Description                                           |
| -------------- | ----------------------------------------------------- |
| `-t`, `--type` | 备份类型。支持的值：physical、logical（默认）、incremental、[external](../features/snapshots.md)。未指定时，Percona Backup for MongoDB 创建逻辑备份。 |
| `--base`       | 仅用于增量备份。将备份设置为基础并开始跟踪增量备份历史，以计算并保存后续增量备份的数据块差异。 |  
| `--compression`| 使用压缩创建备份。 <br> 支持的压缩方法：`gzip`、`snappy`、`lz4`、`s2`、`pgzip`、`zstd`。默认：`s2` <br> `none` 值表示备份期间不进行压缩。 |
| `--compression-level` | 配置压缩级别从 0 到 10。默认值取决于使用的压缩方法。  |
| `--num-parallel-collections`| 设置特定逻辑备份期间并行处理的集合数。未定义时，`pbm-agent` 处理为 `backup.numParallelCollections` 配置参数定义的并行集合数。如果未定义，默认并行处理的集合数是逻辑 CPU 数的一半。从版本 2.7.0 开始可用。|
| `-o`, `--out=text`    | 显示输出格式为纯文本或 JSON 对象。支持的值：`text`、`json` |
| `--wait`       | 等待备份完成。该标志会阻塞 shell 会话。|
| `--wait-time`  | 等待 PBM 报告命令执行状态的时间。将此标志与 `--wait` 标志一起使用。您可以指定持续时间（分钟或小时）（例如 5m、1h）。 <br><br>未设置时，PBM 等待直到命令执行。 <br><br>如果执行命令所需的时间超过定义的等待时间，PBM 打印 `Operation is in progress. Check pbm status and logs` 错误消息并解除阻塞 shell 会话。`pbm-agent` 继续执行命令，使您能够通过 `pbm status` 命令跟踪其进度。从版本 2.6.0 开始可用。| 
| `-l`, `--list-files` | 仅用于外部备份。显示每个节点要复制的文件列表。|
| `--ns="database.collection"`| 创建指定命名空间的逻辑备份 - 数据库和集合。要备份数据库中的所有集合，请以 `--ns="database.*"` 格式指定值。从版本 2.8.0 开始，您可以为备份传递多个命名空间作为逗号分隔的列表。格式为 `ns=db1.*,db2.coll2,db3.coll1,db3.collX` 。|

??? "JSON 输出"

    ```json
    {
      "name": "<backup_name>",
      "storage": "<my-backup-dir>"
    }
    ```

## pbm backup-finish

关闭 `backupCursor` 并完成外部备份。必须在运行 `pbm backup -t external` 后运行。要了解更多信息，请参阅[基于快照的物理备份 API](../features/snapshots.md)。

该命令具有以下语法：

```bash
pbm backup-finish [backup-name] 
```

## pbm cancel-backup

取消正在运行的备份。备份在备份列表中标记为已取消。

该命令接受以下标志：

| Flag                | Description              | 
| ------------------- | ------------------------ |
| `-o`, `--out=text`  | 显示输出格式为纯文本或 JSON 对象。支持的值：`text`、`json`         |

??? "JSON 输出"

    ```json
    {
      "msg": "Backup cancellation has started"
    }
    ```

## pbm cleanup

删除过时的备份快照和时间点恢复 oplog 切片。

该命令具有以下语法：

```bash
pbm cleanup [<flags>]
```

该命令接受以下标志：

| Flag                     | Description               |
| ------------------------ | ------------------------- |
| `--older-than=TIMESTAMP` | 删除早于指定日期/时间的备份，格式为：<br> - `%Y-%M-%DT%H:%M:%S`（例如 2020-04-20T13:13:20），<br> - `%Y-%M-%D`（例如 2020-04-20），<br> - `XXd`（例如 30d）。仅支持天数|
| `-w`, `--wait`           | 等待清理完成。该标志会阻塞 shell 会话|
| `--wait-time`  | 等待 PBM 报告命令执行状态的时间。将此标志与 `--wait` 标志一起使用。您可以指定持续时间（分钟或小时）（例如 5m、1h）。 <br><br>未设置时，PBM 等待直到命令执行。 <br><br>如果执行命令所需的时间超过定义的等待时间，PBM 打印 `Operation is in progress. Check pbm status and logs` 错误消息并解除阻塞 shell 会话。`pbm-agent` 继续执行命令，使您能够通过 `pbm status` 命令跟踪其进度。从版本 2.6.0 开始可用。| 
| `-y`, `--yes`            | 在不询问用户确认的情况下清理数据存储|
| `--dry-run`              | 检查要删除的旧数据而不删除它。允许验证要删除的数据| 


## pbm config

设置、更改或列出 Percona Backup for MongoDB 配置。

该命令具有以下语法：

```bash
pbm config [<flags>] [<key>]
```

该命令接受以下标志：

| Flag               | Description                           |
| ------------------ | ------------------------------------- | 
| `--force-resync`   | 将存储在 PBM 控制集合中的 PBM 元数据（备份、时间点恢复块、恢复）与当前存储中的数据重新同步。 <br> 从版本 2.10.0 开始，PBM 仅从存储中检索最新恢复的恢复元数据以提高重新同步性能。要检索完整的恢复历史，还要添加 `--include-restores` 标志。备份和时间点恢复块的重新同步保持不变。|            
| `--list`           | 列出当前设置                  |
| `--file=FILE`      | 从 YAML 文件上传配置信息   |
| `--set=SET`        | 设置新的配置选项值。以 `<key.name=value>` 格式指定选项。                                    |
| `-o`, `--out=text` | 显示输出格式为纯文本或 JSON 对象。支持的值：text、json                      |
| `-w`, `--wait`     | 等待备份列表与存储的重新同步完成。您只能将此标志与 `--force-resync` 标志一起使用。|
| `--wait-time`  | 等待 PBM 报告重新同步执行状态的时间。将此标志与 `--wait` 标志一起使用。您可以指定持续时间（分钟或小时）（例如 5m、1h）。 <br><br>未设置时，PBM 等待直到命令执行。 <br><br>如果执行命令所需的时间超过定义的等待时间，PBM 打印 `Operation is in progress. Check pbm status and logs` 错误消息并解除阻塞 shell 会话。`pbm-agent` 继续执行命令，使您能够通过 `pbm status` 命令跟踪其进度。从版本 2.6.0 开始可用。| 
| `--include-restores`| 从存储重新同步完整的恢复元数据历史。将此标志与 `--force-resync` 标志一起使用。请注意，检索完整的恢复历史可能会影响重新同步性能。从版本 2.10.0 开始可用。 |


??? "PBM 配置输出"

    ```json
    {
      "pitr": {
        "enabled": false,
        "oplogSpanMin": 0
      },
      "storage": {
        "type": "filesystem",
        "s3": {
          "region": "",
          "endpointUrl": "",
          "bucket": ""
        },
        "azure": {},
        "filesystem": {
          "path": "<my-backup-dir>"
        }
      },
      "restore": {
        "batchSize": 500,
        "numInsertionWorkers": 10
      },
      "backup": {}
    }
    ```

??? "设置配置值"   

    ```json
    [
      {
        "key": "pitr.enabled",
        "value": "true"
      }
    ]
    ``` 

## pbm delete-backup

删除指定的备份快照或早于指定时间的所有备份快照，可选择按特定类型过滤。该命令删除未运行的备份，无论使用何种远程备份存储。

以下是命令语法：

```bash
pbm delete-backup [<flags>] [<name>]
```

该命令接受以下标志：

| Flag                     | Description             |
| ------------------------ | ----------------------- |
| `--older-than=TIMESTAMP` | 删除早于指定日期/时间的备份，格式为：<br> - `%Y-%M-%DT%H:%M:%S`（例如 2023-04-20T13:13:20）或 <br> - `%Y-%M-%D`（例如 2023-04-20）|
| `--type=TYPE`           | 删除指定类型的备份。必须与 `-older-than` 标志一起使用。从版本 2.4.0 开始可用|
| `--force`                | 在不询问用户确认的情况下强制删除备份。已弃用。请改用 `--yes` 标志。 |
| `-y`, `--yes`            | 在不询问用户确认的情况下删除备份 |
| `--dry-run`              | 打印要删除的备份快照列表而不删除它们。您可以使用该标志检查将删除的确切内容。从版本 2.4.0 开始可用。 | 

## pbm delete-pitr

删除为时间点恢复生成的 oplog 切片。

该命令具有以下语法：

```bash
pbm delete-pitr [<flags>]
```

该命令接受以下标志：

| Flag                     | Description               |
| ------------------------ | ------------------------- |
| `-a`, `--all`            | 删除所有 oplog 切片。已弃用。请改用 `--older-than flag`  |
| `--older-than=TIMESTAMP` | 删除早于指定日期/时间的 oplog 切片，格式为： <br> - `%Y-%M-%DT%H:%M:%S`（例如 2020-04-20T13:13:20）或 <br> - `%Y-%M-%D`（例如 2020-04-20） <br><br> 当您指定时间戳时，Percona Backup for MongoDB 将其向下舍入以与最接近的备份快照的完成时间对齐，并删除早于该时间的 oplog 切片。因此，会保留额外的切片。这样做是为了确保 oplog 连续性。举例说明，PITR 时间范围是 `2021-08-11T11:16:21 - 2021-08-12T08:55:25`，备份快照是： <br><br> `2021-08-12T08:49:46Z 13.49MB [restore_to_time: 2021-08-12T08:50:06]` <br> `2021-08-11T11:36:17Z 7.37MB [restore_to_time: 2021-08-11T11:36:38]`<br> <br> 假设您指定时间戳 `2021-08-11T19:16:21`。最接近的备份是 `2021-08-11T11:36:17Z 7.37KB [restore_to_time: 2021-08-11T11:36:38]`。PBM 将时间戳向下舍入到 `2021-08-11T11:36:38` 并删除早于该时间的所有切片。结果，您的 PITR 时间范围是 `2021-08-11T11:36:38 - 2021-08-12T09:00:25`。 <br><br> **注意**：Percona Backup for MongoDB 不会删除最新备份之后的 oplog 切片。这样做是为了确保从该备份快照进行时间点恢复。例如，如果快照是 `2021-07-20T07:05:23Z [restore_to_time: 2021-07-21T07:05:44]` 并且您指定时间戳 `2021-07-20T07:05:45`，Percona Backup for MongoDB 仅删除在 `2021-07-20T07:05:23Z` 之前创建的切片。|
| `--force`                | 在不询问用户确认的情况下强制删除 oplog 切片。已弃用。请改用 `-y`/`--yes` 标志。  |
| `-o`, `--out=json`       | 显示输出为纯文本（默认）或 JSON 对象。支持的值：`text`、`json`。   |
| `--yes`                  | 在不询问用户确认的情况下删除备份 |
| `--dry-run`              | 打印要删除的 oplog 切片列表而不删除它们。您可以使用该标志检查将删除的确切内容。从版本 2.4.0 开始可用。 | 
| `-w`, `--wait`          | 等待删除操作完成。 |
| `--wait-time`  | 等待 PBM 报告命令执行状态的时间。将此标志与 `--wait` 标志一起使用。您可以指定持续时间（分钟或小时）（例如 5m、1h）。 <br><br>未设置时，PBM 等待直到命令执行。 <br><br>如果执行命令所需的时间超过定义的等待时间，PBM 打印 `Operation is in progress. Check pbm status and logs` 错误消息并解除阻塞 shell 会话。`pbm-agent` 继续执行命令，使您能够通过 `pbm status` 命令跟踪其进度。从版本 2.6.0 开始可用。| 


## pbm describe-backup

提供有关备份的详细信息：

- 备份名称
- 类型
- 状态
- 命名空间 - 选择性备份期间备份的内容
- 大小
- 失败备份的错误消息
- 最后写入时间戳 
- 最后写入时间 - 最后写入的人类可读指示 
- 最后转换时间 - 备份更改其状态的时间戳
- 集群信息：副本集名称、此副本集上的备份状态、是否用作配置服务器副本集、最后写入时间戳
- 副本集信息：名称、备份状态、最后写入时间戳和最后转换时间、`mongod` 安全选项（如果配置了加密）。
- 对于基于快照的备份，提供正在复制的文件列表
- 对于逻辑和选择性备份，提供备份中包含的集合列表。从版本 2.3.0 开始可用。

该命令具有以下语法：

```bash
pbm describe-backup [<backup-name>] [<flags>] 
```

| Flag                  | Description                           |
| --------------------- | ------------------------------------- |
| `-o`, `--out=text`    | 显示状态为纯文本或 JSON 对象。支持的值：`text`、`json`|
| `-l`, `--list-files`  | 显示基于快照的备份正在复制的文件列表 |
| `--with-collections`  | 显示备份中包含的集合。仅用于逻辑和选择性备份。从版本 2.3.0 开始可用。 |

### Output

输出文档包含以下字段： 

??? admonition "JSON 输出"

    ```json
    {
      "name": "<backup_name>",
      "opid": "<string>",
      "type": "logical",
      "last_write_ts": Timestamp,
      "last_transition_ts": Timestamp,
      "last_write_time": "2022-09-30T14:02:49Z",
      "last_transition_time": "2022-09-30T14:02:54Z",
      "namespaces": [
        "flight.booking"
      ],
      "mongodb_version": "<version>",
      "fcv": "<version>",
      "pbm_version": "<version>",
      "status": "done",
      "size": 470805945,
      "size_h": "449.0 MiB",
      "replsets": [
        {
          "name": "<name>",
          "status": "done",
          "node": "example.mongodb.com:27017",
          "last_write_ts": Timestamp,
          "last_transition_ts": Timestamp,
          "last_write_time": "2022-09-30T14:02:49Z",
          "last_transition_time": "2022-09-30T14:02:53Z",
          "configsvr": true,
          "security": {}
        },
        {...},
        {...}
      ]
    }
    ```

| Field       | Description |
| ----------- | ----------- |
| `name`      | 备份名称 |
| `opid`      | 操作的唯一标识符 |
| `type`      | 备份类型。支持的值：logical、physical、incremental、external |
| `last_write_ts` | 最后写入的时间戳 |
| `last_transition_ts` | 备份更改其状态的时间戳 |
| `last_write_time` | 最后写入的人类可读指示 |
| `last_transition_time` | 备份更改其状态的时间的人类可读指示|
| `namespaces` | 备份中包含的命名空间列表。用于选择性备份 |
| `mongodb_version` | MongoDB 版本 |
| `fcv` | 功能兼容性版本 |
| `pbm_version` | Percona Backup for MongoDB 版本 |
| `status` | 备份状态。支持的值：running、dumpDone、done、copyReady、error、canceled |
| `size` | 备份大小（字节） |
| `size_h` | 人类可读格式的备份大小 |
| `error`  | 失败备份的错误消息 |
| `replsets` | 备份中包含的副本集列表。每个副本集具有以下字段： <br> - `name` - 副本集名称 <br> - `status` - 此副本集上的备份状态 <br> - `node` - 节点名称和端口 <br> - `last_write_ts` - 最后写入的时间戳 <br> - `last_transition_ts` - 备份更改其状态的时间戳 <br> - `last_write_time` - 最后写入的人类可读指示 <br> - `last_transition_time` - 备份更改其状态的时间的人类可读指示 <br> - `security` - `mongod` 进程的安全选项 <br> - `configsvr` - 指示这是配置服务器副本集 |


## pbm describe-restore

显示有关恢复的详细信息：

* 恢复名称
* opID
* 从中恢复数据库的备份名称
* 类型
* 状态
* 恢复的开始和完成时间
* 最后转换时间 – 恢复过程更改其状态的时间
* 每个副本集的名称、其恢复状态和最后转换时间

对于**仅物理备份**，提供以下附加信息：

* 节点名称
* 节点上的恢复状态
* 最后转换时间

该命令具有以下语法：

```bash
pbm describe-restore [<restore-timestamp>] [<flags>] 
```

该命令接受以下标志：

| Flag                     | Description             |
| ------------------------ | ----------------------- |
| `-c`, `--config=CONFIG`  | 仅用于**物理恢复**。将 Percona Backup for MongoDB 指向配置文件，以便它可以从远程存储读取恢复状态。例如，`pbm describe-restore -c /etc/pbm/conf.yaml <restore-name>`。|
| `-o`, `--out=TEXT`       | 显示输出为纯文本（默认）或 JSON 对象。支持的值：``text``、``json``。|

### Output

输出文档包含以下字段：

??? admonition "选择性恢复状态"

    ```json
    {
     "name": "<restore_name>",
     "opid": "string",
     "backup": "<backup_name>",
     "type": "logical",
     "status": "done",
     "start": "Time"
     "finish": "Time"
     "namespaces": [
        "<database.*>"
     "last_transition_time": "Time"
     ]
     "replsets": [
       {
         "name": "rs1",
         "status": "done",
         "last_transition_time": "Time"
       },
       {
        "name": "rs0",
        "status": "done",
        "last_transition_time": "Time"
       },
       {
         "name": "cfg",
         "status": "done",
         "last_transition_time": "Time"
       }
     ],
    }
    ```

??? admonition "物理恢复状态"

    ```json
    {
     "name": "<restore_name>",
     "opid": "string",
     "backup": "<backup_name>",
     "type": "physical",
     "status": "done",
     "start": "Time"
     "finish": "Time"
     "last_transition_time": "Time",
     "replsets": [
       {
         "name": "rs1",
         "status": "done",
         "last_transition_time": "Timestamp",
         "nodes": [
           {
             "name": "IP:port",
             "status": "done",
             "last_transition_time": "Timestamp"
           }
         ]
       }
     ],
    }
    ```

| Field       | Description |
| ----------- | ----------- |
| `name`      | 恢复名称 |
| `opid`      | 操作的唯一标识符 |
| `backup`    | 从中恢复数据库的备份名称 |
| `type`      | 恢复类型。支持的值：logical、physical|
| `status`    | 恢复状态。支持的值：running、copyReady、done、error |
| `start`     | 恢复开始的时间 |
| `finish`    | 恢复完成的时间。仅用于成功的恢复（状态为 `done`） | 
| `error`     | 失败恢复的错误消息 |
| `last_transition_time` | 恢复过程更改其状态的时间的人类可读指示 |
| `namespaces` | 恢复中包含的命名空间列表。用于选择性恢复 |
| `replsets`  | 恢复中包含的副本集列表。每个副本集具有以下字段： <br> - `name` - 副本集名称 <br> - `status` - 此副本集上的恢复状态 <br> - `error` - 失败恢复的错误消息 <br> - `last_transition_time` - 恢复过程更改其状态的时间的人类可读指示 <br> - `nodes` - 恢复中包含的节点列表。 |
| `replsets.nodes`     | 恢复中包含的节点列表。每个节点具有以下字段： <br> - `name` - 节点名称和端口 <br> - `status` - 节点上的恢复状态 <br> - `error` - 失败恢复的错误消息 <br> - `last_transition_time` - 恢复过程更改其状态的时间的人类可读指示 |

## pbm diagnostic

生成有关特定备份或恢复的详细信息的报告。您也可以将其用于其他命令。要了解更多信息，请参阅[诊断报告](../troubleshoot/pbm-report.md)。

该命令具有以下语法：

```bash
pbm diagnose --path path --name <backup-name> --opid <OPID>
```

该命令接受以下标志：

| Flag                | Description                      |
| ------------------- | -------------------------------- |
| `--path`            | 保存报告的路径。如果目录不存在，PBM 在报告生成期间创建它。确保运行 PBM CLI 的用户对指定路径具有写入访问权限 |
| `--name`            | 所需备份或恢复的名称 |
| `--opid`            | 指定命令的唯一操作 ID。您可以从 `pbm logs`、`pbm describe-backup` / `pbm describe-restore` 输出中检索它。 |
| `--archive`         | 在指定路径中创建报告的 .zip 归档文件。|


## pbm help

返回有关 `pbm` 命令的帮助信息。

## pbm list

提供备份列表及其状态。备份状态如下：

* 进行中 - 备份正在运行
* 已取消 - 备份已取消
* 错误 - 备份以错误结束
* 无状态意味着备份已完成

仅列出成功完成的备份。要查看有关正在运行或失败的备份的信息，请运行 [`pbm status`](#pbm-status)。

启用时间点恢复时，`pbm list` 还提供恢复的有效时间范围和时间点恢复状态。

该命令具有以下语法：

```bash
pbm list [<flags>]
```

该命令接受以下标志：

| Flag                | Description                      |
| ------------------- | -------------------------------- |
| `--restore`         | 显示最后 N 个恢复。从版本 2.0 开始，输出显示恢复名称而不是备份名称，因为可以从单个备份进行多个恢复。           |
| `--size=0`          | 显示最后 N 个备份。它还提供恢复是否为选择性恢复的信息。         |
| `-o`, `--out=text`  | 显示输出格式为纯文本或 JSON 对象。支持的值：`text`、`json`                 |
| `--unbacked`        | 显示在没有基础备份快照的情况下保存的时间点恢复 oplog 切片。从版本 1.8.0 开始可用。|
| `--replset-remapping` | 映射数据恢复/oplog 重放的副本集名称。值格式为 `to_name_1=from_name_1,to_name_2=from_name_2`|

??? "备份列表"

    ```json
    {
      "snapshots": [
          {
            "name": "<backup-name>",
            "status": "done",
            "restoreTo": Timestamp,
            "pbmVersion": "<version>",
            "type": "logical",
            "src": ""
          },
          {
            "name": "<backup-name>",
            "status": "done",
            "restoreTo": Timestamp,
            "pbmVersion": "<version>",
            "type": "logical",
            "src": "",
            "storage": "<storage-name>"
          }
      ],
      "pitr": {
        "on": false,
        "ranges": [
          {
            "range": {
              "start": Timestamp,
              "end": Timestamp
            }
          },
          {
            "range": {
              "start": Timestamp,
              "end": Timestamp
            },
          {
            "range": {
              "start": Timestamp,
              "end": Timestamp (no base snapshot)
            }
          }
        ]
      }
    }
    ```

??? "恢复历史"
 
    完整恢复 

    ```json
     {
        "start": Timestamp,
        "status": "done",
        "type": "snapshot",
        "snapshot": "<backup_name>",
        "name": "<restore_name>"
      }
    ```

    选择性恢复

    ```json
      {
        "start": Timestamp,
        "status": "done",
        "type": "snapshot",
        "snapshot": "<backup_name>",
        "name": "<restore_name>",
        "namespaces": [
          "<database.collection>"
        ]
      }
    ```

    时间点恢复

    ```json
      {
        "start": Timestamp,
        "status": "done",
        "type": "pitr",
        "snapshot": "<backup_name>",
        "point-in-time": Timestamp,
        "name": "<restore_name>"
      }
    ```

    选择性时间点恢复

    ```json
    {
        "start": Timestamp,
        "status": "done",
        "type": "pitr",
        "snapshot": "<backup_name>",
        "point-in-time": Timestamp,
        "name": "<restore_name>",
        "namespaces": [
          "<database.collection>"
        ]
      }
    ]
    ```

## pbm logs

显示来自所有 `pbm-agent` 进程的日志信息。

该命令具有以下语法：

```bash
pbm logs [<flags>]
```

该命令接受以下标志：

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `-t`, `--tail=20`       | 显示最后 N 个条目。默认情况下，输出显示最后 20 个条目。 <br> `0` 表示显示所有日志消息。 |
| `-e`, `--event=EVENT`   | 显示按指定事件过滤的日志。支持的事件：<br> - backup<br> - restore <br> - resyncBcpList <br> - pitr <br> - pitrestore <br> - delete <br>  |
| `-o`, `--out=text`      | 显示日志信息为文本（默认）或 JSON 格式。 <br> 支持的值：`text`、`json` |
| `-n`, `--node=NODE`     | 显示指定节点或副本集的日志。<br> 以 `replset[/host:port]` 格式指定节点 |
| `-f`, `--follow`        | 跟随日志输出。允许动态查看日志 |
| `-s`, `--severity=I`    | 显示按严重性级别过滤的日志。<br> 支持的级别（从低到高）：D - Debug、I - Info（默认）、W - Warning、E - Error、F - Fatal。<br><br> 输出包括指定的严重性级别和所有更高级别 |
| `--timezone`=TIMEZONE   | 日志输出的时区。 <br>支持的值：`UTC`（默认）、`local` 或 [IANA 时区格式](https://en.wikipedia.org/wiki/Tz_database) 中的时区（例如 `America/New_York`）
| `-i`, `--opid=OPID`     | 显示正在进行的操作的日志。操作由 OpID 标识 |
| `-x`, `--extra`         | 以文本格式显示额外数据 |

在[查看备份日志](../usage/logs.md) 中查找使用示例。

??? admonition "日志输出"
  
    ```json
    [
      {
        "t": "",
        "s": 3,
        "rs": "rs0",
        "node": "example.mongodb.com:27017",
        "e": "",
        "eobj": "",
        "ep": {
          "T": 0,
          "I": 0
        },
        "msg": "listening for the commands"
      },
      ....
    ]
    ```

## pbm oplog-replay

允许在任何备份之上重放 oplog：逻辑、物理、存储级别快照（如 EBS 快照）并将其恢复到特定时间点。

要了解更多用法，请参阅时间点恢复 oplog 重放。

该命令具有以下语法：

```bash
pbm oplog-replay [<flags>]
```

该命令接受以下标志：

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `start=timestamp`       | oplog 重放的开始时间。 |
| `end=timestamp`         | oplog 重放的结束时间。   |
| `--replset-remapping`   | 映射 oplog 重放的副本集名称。值格式为 `to_name_1=from_name_1,to_name_2=from_name_2`。 |
| `-w`, `--wait`          | 等待 oplog 重放操作完成。 |
| `--wait-time`  | 等待 PBM 报告 oplog 重放执行状态的时间。将此标志与 `--wait` 标志一起使用。您可以指定持续时间（分钟或小时）（例如 5m、1h）。 <br><br>未设置时，PBM 等待直到命令执行。 <br><br>如果执行命令所需的时间超过定义的等待时间，PBM 打印 `Operation is in progress. Check pbm status and logs` 错误消息并解除阻塞 shell 会话。`pbm-agent` 继续执行命令，使您能够通过 `pbm status` 命令跟踪其进度。从版本 2.6.0 开始可用。| 


## pbm profile add

将外部存储配置保存到 PBM。此信息通过配置配置文件定义。 

要了解更多关于配置配置文件的信息，请参阅[多个存储用于备份](../features/multi-storage.md)。

该命令具有以下语法：

```bash
pbm profile add [<flags>] <profile-name> <path/to/profile.yaml>
``` 

该命令接受以下标志：

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| --sync                  | 添加定义外部存储的配置文件并从该存储同步备份列表|
|--wait                   | 等待配置文件被添加。该标志会阻塞 shell 会话。|
| `--wait-time`  | 等待 PBM 报告添加配置文件和备份同步状态的时间。将此标志与 `--wait` 标志一起使用。您可以指定持续时间（分钟或小时）（例如 5m、1h）。 <br><br>未设置时，PBM 等待直到命令执行。 <br><br>如果执行命令所需的时间超过定义的等待时间，PBM 打印 `Operation is in progress. Check pbm status and logs` 错误消息并解除阻塞 shell 会话。`pbm-agent` 继续执行命令，使您能够通过 `pbm status` 命令跟踪其进度。从版本 2.6.0 开始可用。|
|-o, --out=text           | 显示输出格式为纯文本或 JSON 对象。支持的值：`text`、`json`|

??? admonition "添加配置文件"

    ```json
    {
      "msg": "OK"
    }
    ```

## pbm profile list

提供有关添加到 PBM 的[配置配置文件](../features/multi-storage.md#configuration-profiles) 的信息。 

该命令具有以下语法：

```bash
pbm profile list [<flags>]
```

该命令接受以下标志：

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `-o`, `--out=text`      | 显示输出格式为纯文本或 JSON 对象。支持的值：`text`、`json` |

??? admonition "列出配置文件"

    ```json
    {
      "profiles": [
        {
          "name": "test1",
          "profile": true,
          "storage": {
            "type": "filesystem",
            "filesystem": {
              "path": "/tmp/local_backups"
            }
          }
        }
      ]
    }
    ```

## pbm profile remove

从 PBM 中删除指定的配置配置文件。

该命令具有以下语法：

```bash
pbm profile remove <profile-name> [<flags>]
```

该命令接受以下标志：

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `--wait`                | 等待配置文件被删除。该标志会阻塞 shell 会话。|
| `--wait-time`  | 等待 PBM 报告配置文件删除状态的时间。将此标志与 `--wait` 标志一起使用。您可以指定持续时间（分钟或小时）（例如 5m、1h）。 <br><br>未设置时，PBM 等待直到命令执行。 <br><br>如果执行命令所需的时间超过定义的等待时间，PBM 打印 `Operation is in progress. Check pbm status and logs` 错误消息并解除阻塞 shell 会话。`pbm-agent` 继续执行命令，使您能够通过 `pbm status` 命令跟踪其进度。从版本 2.6.0 开始可用。|
| `-o`, `--out=text`      | 显示输出格式为纯文本或 JSON 对象。支持的值：`text`、`json` |

??? admonition "删除配置文件"

    ```json
    {
      "msg": "OK"
    }
    ```

## pbm profile show

根据指定的配置配置文件显示外部存储配置。

该命令具有以下语法：

```bash
pbm profile show <profile-name> [<flags>]
```

该命令接受以下标志：

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `-o`, `--out=text`      | 显示输出格式为纯文本或 JSON 对象。支持的值：`text`、`json` |

??? admonition "显示配置文件"

    ```json
    {
      "name": "test1",
      "profile": true,
      "storage": {
        "type": "filesystem",
        "filesystem": {
          "path": "/tmp/local_backups"
        }
      }
    }
    ```

## pbm profile sync

根据指定的配置配置文件从外部存储同步备份列表。

该命令具有以下语法：

```bash
pbm profile sync <profile-name> [<flags>]
```

该命令接受以下标志：

| Flag                    | Description                          |
| ----------------------- | ------------------------------------ |
| `--all`                 | 从所有存储同步备份列表。|
| `--clear`               | 清除存储中的备份列表。要清除特定存储中的备份列表，请传递配置文件名称。与 `--all` 一起使用时，清除所有存储中的备份列表。 |
| `--wait`                | 等待配置文件被同步。该标志会阻塞 shell 会话。|
| `--wait-time`  | 等待 PBM 报告配置文件同步状态的时间。将此标志与 `--wait` 标志一起使用。您可以指定持续时间（分钟或小时）（例如 5m、1h）。 <br><br>未设置时，PBM 等待直到命令执行。 <br><br>如果执行命令所需的时间超过定义的等待时间，PBM 打印 `Operation is in progress. Check pbm status and logs` 错误消息并解除阻塞 shell 会话。`pbm-agent` 继续执行命令，使您能够通过 `pbm status` 命令跟踪其进度。从版本 2.6.0 开始可用。|
| `-o`, `--out=text`      | 显示输出格式为纯文本或 JSON 对象。支持的值：`text`、`json` |


## pbm restore

从指定的备份/恢复到指定的时间点恢复数据库。根据备份类型，进行逻辑、物理或基于快照的恢复。

该命令具有以下语法：

```bash
pbm restore [<flags>] [<backup_name>]
```

有关使用 `pbm restore` 的更多信息，请参阅[恢复备份](../usage/restore.md)。

该命令接受以下标志：

| Flag                | Description                           |
| ------------------- | ------------------------------------- |
| `--external`        | 指示备份是在 PBM 外部创建的（例如，基于快照）       |
| `--time=TIME`       | 将数据库恢复到指定的时间点。用于逻辑恢复，如果启用了[时间点恢复](../features/point-in-time-recovery.md)。 |
| `-w`                | 等待恢复完成。该标志会阻塞 shell 会话。 |
| `--wait-time`  | 等待 PBM 报告恢复执行状态的时间。将此标志与 `--wait` 标志一起使用。您可以指定持续时间（分钟或小时）（例如 5m、1h）。 <br><br>未设置时，PBM 等待直到命令执行。 <br><br>如果执行命令所需的时间超过定义的等待时间，PBM 打印 `Operation is in progress. Check pbm status and logs` 错误消息并解除阻塞 shell 会话。`pbm-agent` 继续执行命令，使您能够通过 `pbm status` 命令跟踪其进度。从版本 2.6.0 开始可用。|
| `-o`, `--out=text`  | 显示输出格式为纯文本或 JSON 对象。支持的值：`text`、`json` |
| `--base-snapshot`   | 从指定的备份恢复到指定的时间点。没有此标志时，使用时间戳之前最近的备份进行时间点恢复。 <br><br> 在版本 2.3.0 中，此标志对于[从物理备份进行时间点恢复](../usage/pitr-physical.md) 是可选的。 <br><br> 在版本 2.2.0 中，此标志对于进行[从物理备份进行时间点恢复](../usage/pitr-physical.md) 是必需的。没有它，PBM 查找逻辑备份以从中恢复。|
| `--replset-remapping`| 映射数据恢复/oplog 重放的副本集名称。值格式为 `to_name_1=from_name_1,to_name_2=from_name_2`|
| `--ns=<database.collection>`| 恢复指定的命名空间 - 数据库和集合。要恢复数据库中的所有集合，请将值指定为 `--ns=<database.*>`。`--ns` 标志接受多个命名空间作为逗号分隔的列表。例如，`--ns=db1.*,db2.coll2,db3.coll1,db3.collX`|
| `--with-users-and-roles` | 在选择性恢复期间恢复在自定义数据库中创建的用户和角色。将此标志与 `--ns` 标志一起使用。从版本 2.5.0 开始可用。| 
| `-c`, `--config`     | `mongod.conf` 文件的路径 |
| `--num-parallel-collections`| 设置特定逻辑恢复期间并行处理的集合数。未定义时，`pbm-agent` 处理为 `restore.numParallelCollections` 配置参数定义的并行集合数。如果未定义，默认集合数是逻辑 CPU 数的一半。从版本 2.7.0 开始可用。|
| `--num-insertion-workers-per-collection`| 指定每个集合并发运行的插入工作线程数。对于大型导入，增加数量可能会提高导入速度。从版本 2.8.0 开始可用。|
| `--ns-from`="database.collection" |指定要在新名称下恢复的集合的名称。它指示备份中存在的您要恢复的集合。从版本 2.8.0 开始可用。|
| `--ns-to`="database.collection" | 指定您从原始集合恢复的集合的新名称。从版本 2.8.0 开始可用。|
| `--fallback-enabled` | 为物理恢复启用回退目录的使用。在恢复开始时，PBM 将 `dbPath` 的内容复制到特殊的 `.fallback` 目录。如果恢复成功，PBM 删除 `.fallback` 目录。如果恢复以错误结束，PBM 触发回退过程，从 `dbPath` 删除下载的备份文件并将文件从 `.fallback` 目录复制回来，从而将集群恢复到恢复前的初始状态。默认禁用。从版本 2.10.0 开始可用|
| `--allow-partly-done` | 指示 PBM 如何处理状态为 "partlyDone" 的恢复。启用时，PBM 认为恢复成功，失败的节点通过初始同步接收数据。每个分片上至少有一个节点必须成功恢复，以作为剩余节点的数据源。当副本集中的所有节点都恢复失败时，恢复被视为失败。 <br>禁用时，如果您将 `--allow-partly-done` 与 `--fallback-enabled` 一起使用，PBM 启动回退操作。否则，恢复不会启动。 <br><br>默认启用。从版本 2.10.0 开始可用。 <br><br> 请注意，当您从使用 PBM 2.10.0 之前版本创建的备份恢复时，不得将 `--allow-partly-done` 标志设置为 `false`：PBM 自动禁用 `--fallback-enabled` 设置（如果已启用），并且根本无法启动恢复，因为这两个标志不能同时禁用 |


??? "恢复输出"

    ```json
    {
       "name": "<restore_name>"
       "snapshot": "<backup_name>"
    }
    ```

??? "时间点恢复"

    ```json
    {
      "name":"<restore_name>",
      "point-in-time":"<backup_name>"
    }
    ```
## pbm restore-finish

指示 PBM 完成基于快照的物理恢复。必须在运行 `pbm restore --external` 后运行。要了解更多信息，请参阅[基于快照的物理备份 API](../features/snapshots.md)。

该命令具有以下语法：

```bash
pbm restore-finish <restore_name> [flags]
```

该命令接受以下标志：

| Flag                | Description                           |
| ------------------- | ------------------------------------- |
| `-c`                | PBM 配置文件的路径。完成恢复所必需。|


## pbm status

显示 Percona Backup for MongoDB 的状态。输出提供以下信息：

* `pbm-agent` 进程版本、状态和运行节点类型（主节点或从节点）
* 当前正在运行的备份或恢复
* 存储在远程存储中的备份及其状态
* 时间点恢复状态
* 时间点恢复的有效时间范围和数据大小

该命令接受以下标志：

| Flag                   | Description                             |
| ---------------------- | --------------------------------------- |
| `-o`, `--out=text`     | 显示状态为纯文本或 JSON 对象。支持的值：`text`、`json` |
| `-p`, `--priority`     | 显示备份和时间点恢复 oplog 切片的节点优先级。从版本 2.6.0 开始可用。 |
| `--replset-remapping`  | 映射数据恢复/oplog 重放的副本集名称。值格式为 `to_name_1=from_name_1,to_name_2=from_name_2`|
| `-s`, `--sections=SECTIONS` | 显示指定部分的状态。您可以传递多个标志以查看多个部分的状态。支持的值：cluster、pitr、running、backups。 |

??? admonition "状态信息"

    ```json
    {
      "backups": {
        "type": "FS",
        "path": "<my-backup-dir>",
        "snapshot": [
           ...
          {
            "name": "<backup_name>",
            "size": 3143396168,
            "status": "done",
            "printStatus": "success",
            "restoreTo": Timestamp,
            "pbmVersion": "2.10.0",
            "type": "logical",
            "src": ""
      },
          },
        ],
        "pitrChunks": {
          "pitrChunks": [
             ...
            {
              "range": {
                "start": Timestamp,
                "end": Timestamp
              }
            },
            {
              "range": {
                "start": Timestamp,
                "end": Timestamp (no base snapshot) !!! no backup found
              }
            },
          ],
          "size": 677901884
        }
      },
      "cluster": [
        {
          "rs": "<replSet_name>",
          "nodes": [
            {
              "host": "<replSet_name>/example.mongodb:27017",
              "agent": "<version>",
              "role": "",
              "prio_pitr": "1.0",
              "prio_backup": "1.0",
              "ok": true
            }
          ]
        }
      ],
      "pitr": {
        "conf": true,
        "run": false,
        "error": "Timestamp.000+0000 E [<replSet_name>/example.mongodb:27017] [pitr] <error_message>"
      },
      "running": {
          "type": "backup",
          "name": "<backup_name>",
          "startTS": Timestamp,
          "status": "oplog backup",
          "opID": "6113b631ea9ba5b815fee7c6"
        }
    }
    ```



## pbm version

显示 Percona Backup for MongoDB 的版本。

该命令接受以下标志：

| Flag                   | Description                    |
| ---------------------- | ------------------------------ |
| `--short`              | 仅显示版本信息        |
| `--commit`             | 仅显示 git 提交信息     |
| `-o`, `--out=text`     | 显示输出为纯文本或 JSON 对象。支持的值：`text`、`json`|

??? "版本信息"

    ```json
    {
      "Version": "2.6.0",
      "Platform": "linux/amd64",
      "GitCommit": "f9b9948bb8201ba1a6400f6558496934a0685efd",
      "GitBranch": "main",
      "BuildTime": "{{year}}-07-28_15:24_UTC",
      "GoVersion": "go1.16.6"
    }
    ```
