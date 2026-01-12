# 查看备份的详细信息

要查看备份的详细信息，请运行以下命令：

```bash
pbm describe-backup <backup-name>
```

输出提供备份名称、类型、状态、大小以及创建它的集群拓扑信息。对于[选择性备份](../features/selective-backup.md)，它还显示已备份的命名空间。 

??? example "示例输出"

    ```{.text .no-copy}
    name: "2022-08-17T10:49:03Z"
    type: logical
    last_write_ts: 1662039300,2
    last_transition_ts: "1662039304"
    namespaces:
    - Invoices.*
    mongodb_version: 5.0.10-9
    pbm_version: 2.0.0
    status: done
    size: 10234670
    error: ""
    replsets:
    - name: rs1
      status: done
      iscs: false
      last_write_ts: 1662039300,2
      last_transition_ts: "1662039304"
      error: ""
    ```

从版本 2.10.0 开始，命令输出显示整个集群的未压缩备份大小以及每个副本集的压缩/未压缩大小。这有助于 PBM 在执行[使用回退目录的物理恢复](../features/physical.md#physical-restores-with-a-fallback-directory) 时评估所需的磁盘空间。

??? example "示例输出"

    ```{.text .no-copy}
    pbm describe-backup 2025-06-05T16:57:35Z
    name: "2025-06-05T16:57:35Z"
    opid: 6841cc7f1f576b79efb26752
    type: physical
    ...
    status: done
    size_h: 3.3 GiB
    size_uncompressed_h: 4.0 GiB
    ....
    replsets:

    - name: rs2
      status: done
      node: rs202:30202
      size_h: 3.3 GiB
      size_uncompressed_h: 3.6 GiB

!!! admonition "版本添加：[2.3.0](../release-notes/2.3.0.md)"

您可以查看*逻辑*或*选择性*备份中包含的集合列表。这简化了故障排除，因为它有助于识别频繁创建或删除数据库的环境中的备份内容。

要查看备份内容，请使用 `--with-collections` 标志：

```bash
pbm describe-backup <backup-name> --with-collections
```

??? example "示例输出"

    ```{.text .no-copy}
    name: "2023-09-14T14:44:33Z"
    opid: 65031c51e6a16fa0e3deeb5f
    type: logical
    last_write_time: "2023-09-14T14:44:39Z"
    last_transition_time: "2023-09-14T14:44:57Z"
    mongodb_version: 6.0.9-7
    fcv: "6.0"
    pbm_version: 2.2.1
    status: done
    size_h: 89.3 KiB
    replsets:
    - name: rs0
      status: done
      node: rs00:30000
      last_write_time: "2023-09-14T14:44:38Z"
      last_transition_time: "2023-09-14T14:44:56Z"
      collections:
      - admin.pbmRRoles
      - admin.pbmRUsers
      - admin.system.roles
      - admin.system.users
      - admin.system.version
      - db0.c0
      - db0.c1
      - db1.c0
    - name: rs1
      status: done
      node: rs10:30100
      last_write_time: "2023-09-14T14:44:38Z"
      last_transition_time: "2023-09-14T14:44:49Z"
      collections:
      - admin.pbmRRoles
      - admin.pbmRUsers
      - admin.system.roles
      - admin.system.users
      - admin.system.version
      - db0.c0
      - db1.c0
      - db1.c1
    - name: cfg
      status: done
      node: cfg0:27000
      last_write_time: "2023-09-14T14:44:39Z"
      last_transition_time: "2023-09-14T14:44:42Z"
      configsvr: true
      collections:
      - admin.pbmAgents
      - admin.pbmBackups
      - admin.pbmCmd
      - admin.pbmConfig
      - admin.pbmLock
      - admin.pbmLockOp
      - admin.pbmLog
      - admin.pbmOpLog
      - admin.pbmPITRChunks
      - admin.pbmRRoles
      - admin.pbmRUsers
      - admin.system.roles
      - admin.system.users
      - admin.system.version
      - config.chunks
      - config.collections
      - config.databases
      - config.settings
      - config.shards
      - config.tags
      - config.version
    ```
