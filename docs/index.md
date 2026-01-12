# Percona Backup for MongoDB 文档

!!! note ""

    这是最新版本 **PBM {{release}}** 的文档（[发布说明](release-notes/{{release}}.md)）。


Percona Backup for MongoDB (PBM) 是一个开源分布式解决方案，用于将 [MongoDB 分片集群和副本集](details/deployments.md) 一致地备份和恢复到特定时间点。查看[支持的 MongoDB 版本](details/versions.md)。


您可以通过以下方式在运行中的服务器上创建备份和/或将数据库恢复到特定时间点：

* 使用 [PBM 命令行界面](reference/pbm-commands.md)。 
* 通过 Web 界面 [使用 PBM 和 Percona Monitoring and Management](pmm.md)。 

阅读 [PBM 工作原理](intro.md)。查看[支持的 MongoDB 版本](details/versions.md)。


## 为什么选择 PBM？

* 通过[所有支持的备份类型](features/backup-types.md)实现跨集群和副本集的数据一致性
* 与 PBM 相关的性能或操作降级不明显
* 多种[支持的存储类型](details/storage-configuration.md)意味着没有供应商锁定
* 具有[企业级功能](features/comparison.md)的开源解决方案 

## 本文档包含

<div data-grid markdown><div data-banner markdown>

### :material-progress-download: 安装指南 { .title }

准备试用 PBM？通过分步安装说明快速开始。

[快速入门指南 :material-arrow-right:](installation.md){ .md-button }

</div><div data-banner markdown>

### :material-backup-restore: 备份管理 { .title }

了解如何维护您的备份策略。

[备份管理 :material-arrow-right:](usage/backup-physical.md){ .md-button }

</div><div data-banner markdown>

### :fontawesome-solid-gears: 管理 { .title }

调整 PBM 以有效执行日常操作。

[管理 :material-arrow-right:](manage/overview.md){.md-button}
</div><div data-banner markdown>

### :material-frequently-asked-questions: 诊断和常见问题 { .title }

我们的综合资源将帮助您克服挑战，从日常问题到具体疑问。

[运行诊断 :material-arrow-right:](troubleshoot/index.md){.md-button}

</div>
</div>



