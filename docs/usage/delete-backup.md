# 删除备份

使用 [`pbm delete-backup`](../reference/pbm-commands.md#pbm-delete-backup) 删除备份快照，使用 [`pbm delete-pitr`](../reference/pbm-commands.md#pbm-delete-pitr) 删除时间点恢复 oplog 切片。使用 `pbm cleanup --older-than` 命令[自动化备份存储清理](schedule-backup.md#backup-storage-cleanup)。

## 删除过时数据

!!! admonition "版本添加：[2.1.0](../release-notes/2.1.0.md)"

您可以使用 `pbm cleanup --older-than` 命令删除过时的备份快照和时间点恢复 oplog 切片。这简化了[备份轮换的自动化](schedule-backup.md#backup-storage-cleanup)。

您为 `--older-than` 标志指定的时间戳必须采用以下格式：

* `%Y-%M-%DT%H:%M:%S`（例如，2023-04-20T13:13:20）或
* `%Y-%M-%D`（2023-04-20）
* `XXd`（1d 或 30d）。仅支持天数。

在清理期间，您会看到要删除的备份和 oplog 切片，并被要求确认操作。要绕过它，请添加 `--yes` 标志：

```bash
pbm cleanup --older-than=`%Y-%M-%D --yes
```

### 行为

您指定的时间戳被认为是您希望恢复到的时间。因此，PBM 不会删除可用于恢复到此时的所有备份快照和 oplog 切片。

清理工作原理如下：

* **物理和选择性备份** 删除到指定时间。
* **增量物理备份** 如果时间戳不在备份链内，则删除到指定时间。如果时间戳在备份链内，PBM 会检查与指定时间戳相关的最新增量基础备份。PBM 保留此备份和从中派生的整个链以确保潜在的恢复。

    例如，您有以下备份列表：
    
    ```{.bash .no-copy}
    Snapshots:
        2023-04-14T19:34:52Z 520.86MB <incremental> [restore_to_time: 2023-04-14T19:34:54Z]
        2023-04-14T08:12:50Z 576.63MB <incremental, base> [restore_to_time: 2023-04-14T08:12:52Z]
        2023-04-12T03:02:08Z 498.50MB <incremental> [restore_to_time: 2023-04-12T03:02:10Z]
        2023-04-11T19:30:14Z 552.77MB <incremental, base> [restore_to_time: 2023-04-11T19:30:16Z]
        2023-04-11T14:25:51Z 572.41MB <physical> [restore_to_time: 2023-04-11T14:25:54Z]
    ```

    您希望删除所有早于 2023-04-14T15:00:00 的备份

    ```bash
    pbm cleanup --older-than="2023-04-14T15:00:00"
    ```

    此时间戳落在以 `2023-04-14T08:12:50Z` 备份开始的备份链内。这就是为什么 PBM 保留此备份和从中派生的增量备份链，并删除早于此备份的所有数据。

    ??? example "示例输出"    

        ```{.text .no-copy} 
        S3 us-east-1 s3://http://192.168.56.1:9000/bcp/pbme2etest
          Snapshots:
            2023-04-14T19:34:52Z 520.86MB <incremental> [restore_to_time: 2023-04-14T19:34:54Z]
            2023-04-14T08:12:50Z 576.63MB <incremental, base> [restore_to_time: 2023-04-14T08:12:52Z]
        ```
   
* **逻辑备份** 清理还取决于时间点恢复设置。 

    * 默认情况下，PBM 查找与指定时间戳相关的最新备份，并删除所有逻辑备份和 oplog 切片直到备份的 `restore_to_time` 值。  

       为了说明，假设您有以下备份列表：

       ```{.text .no-copy}
       Snapshots:
           2023-04-13T13:26:58Z 147.29MB <logical> [restore_to_time: 2023-04-13T13:27:15Z]
           2023-04-13T10:12:08Z 147.29MB <logical> [restore_to_time: 2023-04-13T10:12:27Z]
           2023-04-13T08:48:32Z 147.28MB <logical> [restore_to_time: 2023-04-13T08:48:51Z]
         PITR chunks [2.11MB]:
           2023-04-13T08:48:52Z - 2023-04-13T13:27:15Z
       ```

       您希望删除所有直到 2023-04-13T12:00:00 的数据。

       与此时间戳相关的最新备份是 `2023-04-13T10:12:08Z 147.29MB`。因此 PBM 删除所有早于此备份的备份。它还删除所有直到备份的 `restore_to_time: 2023-04-13T10:12:27Z` 的 oplog 切片。清理后的输出如下所示：

       示例输出：

       ```{.text .no-copy}
       Snapshots:
           2023-04-13T13:26:58Z 147.29MB <logical> [restore_to_time: 2023-04-13T13:27:15Z]
           2023-04-13T10:12:08Z 147.29MB <logical> [restore_to_time: 2023-04-13T10:12:27Z]
         PITR chunks [157.94KB]:
           2023-04-13T10:12:28Z - 2023-04-13T13:27:46Z
       ```

    * 当启用时间点恢复并且您指定的时间戳大于最新逻辑备份的 `restore_to_time` 时，PBM 保留此备份和从中派生的所有 oplog 切片以确保时间点恢复。 
    * 当指定时间戳等于任何完整逻辑、物理和基础增量备份的 `restore_to_time` 值时，PBM 删除所有逻辑备份快照和 oplog 切片直到此备份的 `restore_to_time`。

## 删除备份快照

### 注意事项

1. 您只能删除未运行的备份（具有 "done" 或 "error" 状态）。要检查备份状态，请运行 [`pbm status`](../reference/pbm-commands.md#pbm-status) 命令。

2. 您只能删除整个增量备份链，不能删除单个增量。当您为单个增量指定时间戳的增量名称时，备份删除将失败。 

3. 为了确保[时间点恢复](pitr-tutorial.md) 的 oplog 连续性，`pbm delete-backup` 命令删除除以下内容之外的任何备份：

    === "版本 2.4.0 及更高版本"

        如果启用了时间点恢复，可以作为时间点恢复基础的备份快照（逻辑、物理、基础增量）。如果有从它派生到 `now` 时间戳的连续 oplog 切片，则此类备份是有效基础。


    === "版本 2.3.1 及更早版本"

        * 可以作为任何时间点恢复基础的备份快照，并且有从它派生的时间点恢复时间范围。要删除此类备份，请首先[删除 oplog 切片](#delete-oplog-slices)，这些切片是在此备份的 `restore-to time` 值之后创建的。    

        * 如果启用了时间点恢复并且此备份之后还没有 oplog 切片，则是最新的备份。    

        为了说明这一点，让我们使用以下 `pbm list` 输出：    

        ```{.bash .no-copy}
        Backup snapshots:
          2022-10-05T14:13:50Z <logical> [restore_to_time: 2022-10-05T14:13:55Z]
          2022-10-06T14:52:42Z <logical> [restore_to_time: 2022-10-06T14:52:47Z]
          2022-10-07T14:57:17Z <logical> [restore_to_time: 2022-10-07T14:57:22Z]    

        PITR <on>:
          2022-10-05T14:13:56Z - 2022-10-05T18:52:21Z
        ```    

        您可以删除备份 `2022-10-06T14:52:42Z`，因为它没有时间点 oplog 切片。您不能删除以下备份：    

        - `2022-10-05T14:13:50Z` 因为它是从 PITR 时间范围 `2022-10-05T14:13:56Z - 2022-10-05T18:52:21Z` 恢复到任何时间点的基础
        - `2022-10-07T14:57:17Z` 因为启用了 PITR 并且之后还没有 oplog 切片。

4. 从版本 [2.4.0](../release-notes/2.4.0.md) 开始，您可以删除任何备份快照，无论从它派生的时间点恢复切片如何。此类切片在 `pbm status` 输出中被标记为 "no base backup"。但是，必须至少保留一个有效的基础备份以确保时间点恢复。 

   如果满足以下条件，此类备份是时间点恢复的有效基础：

   * 备份是以下类型之一：逻辑、物理、基础增量
   * 有从此备份派生的连续 oplog 切片，用于恢复到特定时间戳。 


### 行为

您可以删除指定的备份快照或早于指定时间的所有备份快照。从版本 2.0.0 开始，您还可以删除[选择性备份](../features/selective-backup.md)。 

=== "指定备份"

     要删除备份，请将 `<backup_name>` 指定为参数。

     ```bash
     pbm delete-backup <backup_name>
     ```

=== "早于指定时间的备份"
    
    要删除在指定时间之前创建的备份，请将 `--older-than` 标志传递给 `pbm delete-backup` 命令。为 `pbm delete-backup` 指定时间戳作为参数，格式如下：

    * `%Y-%M-%DT%H:%M:%S`（例如，2021-04-20T13:13:20Z）或
    * `%Y-%M-%D`（2021-04-20）。

    #### 示例

    查看备份：

    ```bash
    pbm list
    ```

    ??? example "示例输出"

        ```{.text .no-copy}
        Backup snapshots:
          2021-04-20T20:55:42Z
          2021-04-20T23:47:34Z
          2021-04-20T23:53:20Z
          2021-04-21T02:16:33Z
        ```

    删除在指定时间戳之前创建的备份

    ```bash
    pbm delete-backup -f --older-than 2021-04-21
    ```

    ??? example "示例输出"

        ```{.text .no-copy}
        Backup snapshots:
          2021-04-21T02:16:33Z
        ```


=== "特定类型的备份"

    要删除在指定时间之前创建的特定类型的备份，请使用 `--type` 和 `--older-than` 标志运行 `pbm delete backup`。PBM 删除所有不作为恢复到指定时间戳基础的备份。

    请注意，您必须指定两个标志才能删除所需类型的备份。
    

    #### 示例

    您有以下备份列表：

    ```{.text .no-copy}
    Backups:
      Snapshots:
        2024-02-26T10:11:05Z 905.92MB <physical> [restore_to_time: 2024-02-26T10:11:07Z]
        2024-02-26T10:06:57Z 86.99MB <logical> [restore_to_time: 2024-02-26T10:07:00Z]
        2024-02-26T10:03:24Z 234.12MB <incremental> [restore_to_time: 2024-02-26T10:03:26Z]
        2024-02-26T10:00:16Z 910.27MB <incremental, base> [restore_to_time: 2024-02-26T10:00:18Z]
        2024-02-26T09:56:18Z 961.68MB <physical> [restore_to_time: 2024-02-26T09:56:20Z]
        2024-02-26T08:43:44Z 86.83MB <logical> [restore_to_time: 2024-02-26T08:43:47Z]
      PITR chunks [8.25MB]:
        2024-02-26T08:43:48Z - 2024-02-26T10:17:21Z
    ```

    您希望删除所有早于上午 10:00 的物理备份。

    ```
    $ pbm delete-backup --older-than="2024-02-26T10:00:00" -t physical -y
    ```

    有两个物理备份快照，但只有 `2024-02-26T09:56:18Z 961.68MB <physical> [restore_to_time: 2024-02-26T09:56:20Z]` 快照通过指定时间戳。因此，PBM 仅删除这一个：

    ```{.text .no-copy}
    Snapshots:
     - "2024-02-26T09:56:18Z" [size: 961.68MB type: <physical>, restore time: 2024-02-26T09:56:20Z]
    Waiting for delete to be done .[done]
    ```

    生成的备份列表如下所示：

    ??? example "示例输出"
        
        ```{.text .no-copy}
        Backups:
          Snapshots:
            2024-02-26T10:11:05Z 905.92MB <physical> [restore_to_time: 2024-02-26T10:11:07Z]
            2024-02-26T10:06:57Z 86.99MB <logical> [restore_to_time: 2024-02-26T10:07:00Z]
            2024-02-26T10:03:24Z 234.12MB <incremental> [restore_to_time: 2024-02-26T10:03:26Z]
            2024-02-26T10:00:16Z 910.27MB <incremental, base> [restore_to_time: 2024-02-26T10:00:18Z]
            2024-02-26T08:43:44Z 86.83MB <logical> [restore_to_time: 2024-02-26T08:43:47Z]
          PITR chunks [8.73MB]:
            2024-02-26T08:43:48Z - 2024-02-26T10:17:21Z
        ```

默认情况下，``pbm delete-backup`` 命令要求您确认是否继续删除。要绕过它，请添加 `-y` 或 `--yes` 标志。

 ```bash
 pbm delete-backup --yes 2023-04-20T13:45:59Z
 ```

!!! admonition ""

    对于 Percona Backup for MongoDB 1.5.0 及更早版本，当您删除备份时，与此备份相关的所有 oplog 切片也会被删除。例如，您删除备份快照 `2020-07-24T18:13:09`，而在此之后创建了另一个快照 `2020-08-05T04:27:55`。**pbm-agent** 仅删除与 `2020-07-24T18:13:09` 相关的 oplog 切片。

    如果您删除早于指定时间的备份，同样适用。

    请注意，当启用时间点恢复时，最新的备份快照和与其相关的 oplog 切片不会被删除。

## 删除 oplog 切片

!!! admonition "版本添加：[1.6.0](../release-notes/1.6.0.md)"

您可以删除在指定时间之前保存的 oplog 切片或一次性删除所有切片。通过删除旧的和/或不必要的切片，您可以节省存储空间。 

### 行为

要查看 oplog 切片，请运行 [`pbm list`](../reference/pbm-commands.md#pbm-list) 命令。如果您已[删除快照](#delete-backup-snapshots) 并希望删除相应的 oplog 切片，请运行 `pbm list --unbacked` 命令查看它们。

=== "删除所有 oplog 切片"

    运行 `pbm delete-pitr` 并传递 `--all` 标志：

    ```bash
    pbm delete-pitr --all
    ```

=== "早于指定时间戳" 
    
    要删除在指定时间之前创建的切片，请使用 `--older-than` 标志运行 `pbm delete-pitr` 命令并为其传递时间戳。时间戳必须采用以下格式：

    * `%Y-%M-%DT%H:%M:%S`（例如，2021-07-20T10:01:18）或
    * `%Y-%M-%D`（2021-07-20）。

    ```bash
    pbm delete-pitr --older-than 2021-07-20T10:01:18
    ```

为了从最新的备份快照启用[时间点恢复](pitr-tutorial.md)，Percona Backup for MongoDB 不会删除在该快照之后创建的切片。例如，如果最新快照是 `2021-07-20T07:05:23Z [restore_to_time: 2021-07-21T07:05:44]` 并且您指定时间戳 `2021-07-20T07:05:44`，Percona Backup for MongoDB 仅删除在 `2021-07-20T07:05:23Z` 之前创建的切片。
