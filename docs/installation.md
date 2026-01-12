# 快速入门指南

Percona Backup for MongoDB (PBM) 是一个开源分布式解决方案，用于一致地备份和恢复 [MongoDB 分片集群和副本集](details/deployments.md)。[了解 PBM 工作原理](intro.md)。

在 [Percona 软件和平台生命周期 :octicons-link-external-16:](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mongodb) 页面上查找 Percona Backup for MongoDB 支持的平台列表。另外，请查看[支持的 MongoDB 版本](details/versions.md)。

## 教程

您可以使用任何简单的安装指南，但**我们建议使用操作系统的包管理器**，这是首先尝试该软件的便捷快速方法。

=== ":octicons-terminal-16: 包管理器"
    
    使用操作系统的包管理器安装 Percona Backup for MongoDB：

    * `apt` - 适用于 Debian 和 Ubuntu Linux
    * `yum` - 适用于 Red Hat Enterprise Linux 和兼容的 Linux 衍生版本

    [从仓库安装 :material-arrow-right:](install/repos.md){.md-button}

=== ":simple-docker: Docker"

     获取我们的 Docker 镜像并启动 PBM 进行快速评估。 

     查看 Docker 指南以获取分步说明。

     [在 Docker 中运行 :material-arrow-right:](install/docker.md){.md-button}

=== ":simple-kubernetes: Kubernetes"

    **Percona Operator for Kubernetes** 是一个控制器，用于简化需要细致和安全数据库专业知识的复杂部署。 

    查看快速入门指南，了解如何使用 Percona Operator for MongoDB 在 Kubernetes 上部署和运行 PBM。

    [在 Kubernetes 中部署快速入门 :material-arrow-right:](https://docs.percona.com/percona-operator-for-mongodb/quickstart.html){.md-button}

=== ":octicons-file-code-16: 从源代码构建"

    通过从源代码构建 PBM 来完全控制安装。

    查看下面的指南以获取分步说明。

    [从源代码构建 :material-arrow-right:](install/source.md){.md-button}

=== ":octicons-download-16: 手动下载"

    如果您需要离线安装 PBM 或偏好特定版本，请查看下面的链接以获取分步指南并访问下载目录。

    [从压缩包安装 :material-arrow-right:](install/tarball.md){.md-button}


## 下一步

[初始设置 :material-arrow-right:](install/initial-setup.md){.md-button}

