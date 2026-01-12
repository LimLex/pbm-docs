# 升级 Percona Backup for MongoDB

推荐且最方便的升级 PBM 的方法是从 Percona 仓库升级。

## 重要说明

1. 仅在主要版本内升级时支持数据备份和恢复的向后兼容性（例如，从 2.1.x 升级到 2.2.y）。当您跨主要版本升级 Percona Backup for MongoDB（例如，从 2.0.x 升级到 2.2.y）时，我们建议在升级后立即创建备份。

2. 在所有安装了 Percona Backup for MongoDB 的节点上升级它。

## 先决条件 

1. [安装 `percona-release` 工具 :octicons-link-external-16:](https://www.percona.com/doc/percona-repo-config/installing.html) 或[将其更新 :octicons-link-external-16:](https://www.percona.com/doc/percona-repo-config/updating.html) 到最新版本。

2. 启用仓库

    ```bash
    sudo percona-release enable pbm release
    ```

<i info>:material-information: 注意：</i> 对于基于 `apt` 的系统，运行 `sudo apt update` 以更新本地缓存。

## 升级到最新版本

=== ":material-debian: 在 Debian 和 Ubuntu Linux 上"

    以 root 用户身份或通过 `sudo` 运行所有命令。
    {.power-number}

    1. 停止 `pbm-agent`

        ```bash
        sudo systemctl stop pbm-agent
        ```

    2. 安装新软件包

        ```bash
        sudo apt install percona-backup-mongodb
        ```  

    3. 重新加载 `systemd` 进程

        ```bash
        sudo systemctl daemon-reload
        ```

    4. 更新权限

        对于*基于文件系统的备份存储*，授予 `mongod` 用户对备份目录的读/写权限。


    5. 启动 `pbm-agent`

        ```bash
        sudo systemctl start pbm-agent
        ```

=== ":material-redhat: 在 Red Hat Enterprise Linux 和衍生版本上"

    以 root 用户身份或通过 `sudo` 运行所有命令。
    {.power-number}

    1. 停止 `pbm-agent`

        ```bash
        sudo systemctl stop pbm-agent
        ```

    2. 安装新软件包

        ```bash
        sudo yum install percona-backup-mongodb
        ```

    3. 重新加载 `systemd` 进程

       从 v1.7.0 开始，使用以下命令重新加载 `systemd` 进程以更新单元文件：

       ```bash
       sudo systemctl daemon-reload
       ```

    4. 更新权限

        对于*基于文件系统的备份存储*，授予 `mongod` 用户对备份目录的读/写权限。

    5. 启动 `pbm-agent`

        ```bash
        sudo systemctl start pbm-agent
        ``` 

## 升级到特定版本

=== ":material-debian: 在 Debian 和 Ubuntu Linux 上"

    以 root 用户身份或通过 `sudo` 运行所有命令。
    {.power-number}

    1. 列出可用版本
 
        ```bash
        sudo apt-cache madison percona-backup-mongodb
        ```

        ??? example "示例输出"

            ```{.text .no-copy}
            percona-backup-mongodb | 2.8.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
            percona-backup-mongodb | 2.7.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
            percona-backup-mongodb | 2.6.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
            percona-backup-mongodb | 2.5.0-1.stretch | http://repo.percona.com/tools/apt stretch/main amd64 Packages
            ```

    2. 停止 `pbm-agent`

        ```bash
        sudo systemctl stop pbm-agent
        ```

    3. 安装软件包

        安装特定版本的软件包。例如，要升级到 Percona Backup for MongoDB 1.7.0，请运行以下命令：

        ```bash
        sudo apt install percona-backup-mongodb=1.7.0-1.stretch
        ```
 
    4. 更新权限

       对于*基于文件系统的备份存储*，授予 `mongod` 用户对备份目录的读/写权限。


    5. 启动 `pbm-agent`
 
        ```bash
        sudo systemctl start pbm-agent
        ``` 

=== ":material-redhat: 在 Red Hat Enterprise Linux 和衍生版本上"
  
    以 root 用户身份或通过 `sudo` 运行所有命令。
    {.power-number}

    1. 列出可用版本

        ```bash
        sudo yum list percona-backup-mongodb --showduplicates
        ```

        ??? example "示例输出"

            ```{.text .no-copy}
            Available Packages
            percona-backup-mongodb.x86_64    1.8-1.el7            pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.8.0-1.el7          pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.7.0-1.el7          pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.6.1-1.el7          pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.6.0-1.el7          pbm-release-x86_64
            percona-backup-mongodb.x86_64    1.5.0-1.el7          pbm-release-x86_64
            ```

    2. 停止 `pbm-agent`

        ```bash
        sudo systemctl stop pbm-agent
        ```

    3. 安装软件包

        安装特定版本的软件包。例如，要升级到 Percona Backup for MongoDB 1.7.1，请运行以下命令：

        ```bash
        sudo yum install percona-backup-mongodb-1.7.1-1.el7
        ```
    
    4. 更新权限

        对于*基于文件系统的备份存储*，授予 `mongod` 用户对备份目录的读/写权限。

    5. 启动 `pbm-agent`

        ```bash
        sudo systemctl start pbm-agent
        ``` 

<i info>:material-information: 注意：</i> 如果 MongoDB 在*不同于 `mongod` 的用户*下运行（Percona Server for MongoDB 的默认配置），请使用相同的用户运行 `pbm-agent`。对于基于文件系统的存储，请授予此用户对备份目录的读/写权限。
