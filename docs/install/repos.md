# 从 Percona 仓库安装

从 Percona 仓库安装软件意味着订阅它们。Percona 提供 [`percona-release` :octicons-link-external-16:](https://www.percona.com/doc/percona-repo-config/index.html) 仓库管理工具。它会自动启用所需的仓库，以便您可以顺利安装和更新 Percona Backup for MongoDB 软件包和所需的依赖项。

--8<-- "pbm-install-nodes.md"

## 开始之前

检查[系统要求](../system-requirements.md) 和[支持的 MongoDB 版本](../details/versions.md)。

## 步骤

<i warning>:material-alert: 警告：</i> 以 root 用户身份或通过 `sudo` 命令运行以下命令。
{.power-number}

### 配置 Percona 仓库

Percona 提供 [`percona-release` :material-arrow-top-right: ](https://docs.percona.com/percona-software-repositories/index.html) 配置工具，可简化仓库操作并能够顺利安装和更新 Percona Backup for MongoDB 软件包和所需的依赖项。 

1. [安装 `percona-release` :octicons-link-external-16:](https://www.percona.com/doc/percona-repo-config/installing.html)。    

2. 启用仓库    

    ```bash
    sudo percona-release enable pbm release
    ```


### 安装 Percona Backup for MongoDB 软件包

=== ":material-debian: 在 Debian 和 Ubuntu 上"    

    1. 重新加载本地软件包数据库：    

        ```bash
        sudo apt update
        ```    

    2. 安装 Percona Backup for MongoDB：    

        ```bash
        sudo apt install percona-backup-mongodb
        ```    

=== ":material-redhat: 在 RHEL 和衍生版本上" 

    运行以下命令安装软件包：

    ```bash
    sudo yum install percona-backup-mongodb
    ```

=== ":fontawesome-brands-amazon: 在 Amazon Linux 2023 上"

    运行以下命令安装软件包：

    ```bash
    sudo yum install percona-backup-mongodb
    ```

--8<-- "install-result.md"

## 下一步

[初始设置 :material-arrow-right:](initial-setup.md){.md-button}
