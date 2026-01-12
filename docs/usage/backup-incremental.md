# 创建增量备份

--8<-- "prepare-backup.md"

## 步骤

1. 要启动增量备份，首先创建完整增量备份。它将作为后续增量备份的基础：

    ```bash 
    pbm backup --type incremental --base
    ```

    `pbm-agent` 开始跟踪增量备份历史，以便能够计算和保存数据块的差异。 

2. 运行常规增量备份：

    ```bash
    pbm backup --type incremental
    ```

增量备份历史如下所示：

??? example "示例输出"

    ```{.bash .no-copy} 
    Snapshots:
        2022-11-25T14:13:43Z 139.82MB <incremental> [restore_to_time: 2022-11-25T14:13:45Z]
        2022-11-25T14:02:07Z 255.20MB <incremental> [restore_to_time: 2022-11-25T14:02:09Z]
        2022-11-25T14:00:22Z 228.30GB <incremental> [restore_to_time: 2022-11-25T14:00:24Z]
        2022-11-24T14:45:53Z 220.13GB <incremental, base> [restore_to_time: 2022-11-24T14:45:55Z]
    ```


## 下一步

[列出备份](../usage/list-backup.md){.md-button}
[进行恢复](restore-incremental.md){.md-button}

## 有用的链接

* [备份和恢复类型](../features/backup-types.md)
* [安排备份](../usage/schedule-backup.md)

