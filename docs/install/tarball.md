# 从 Percona 网站下载 Percona Backup for MongoDB

您可以从 [Percona 网站 :octicons-link-external-16:](https://www.percona.com/downloads/percona-backup-mongodb/) 下载 Percona Backup for MongoDB 并安装它：

* [从二进制压缩包安装](#install-from-binary-tarballs)。
* 手动，从安装包使用 `dpkg`（Debian 和 Ubuntu）或 `rpm`（Red Hat Enterprise Linux 和 CentOS）。但是，您必须确保满足所有依赖项。

--8<-- "pbm-install-nodes.md"

## 开始之前

检查[系统要求](../system-requirements.md) 和[支持的 MongoDB 版本](../details/versions.md)。

## 从二进制压缩包安装

在 [Percona 网站 :octicons-link-external-16:](https://www.percona.com/downloads/percona-backup-mongodb/) 上的**通用 Linux** 菜单项下找到二进制压缩包的链接。
{.power-number}

1. 获取二进制压缩包。将 URL 中的版本替换为您所需的版本。

    ```bash
    wget https://downloads.percona.com/downloads/percona-backup-mongodb/percona-backup-mongodb-{{release}}/binary/tarball/percona-backup-mongodb-{{release}}-x86_64.tar.gz
    ```

2. 解压压缩包

    ```bash
    tar -xf percona-backup-mongodb-{{release}}-x86_64.tar.gz
    ```

3. 将二进制文件的位置导出到 `PATH` 变量

    例如，如果您已将压缩包解压到 `home` 目录，命令将如下所示：

    ```bash
    export PATH=~/percona-backup-mongodb-{{release}}/:$PATH
    ```

--8<-- "install-result.md"

## 下一步

[初始设置 :material-arrow-right:](initial-setup.md){.md-button}
