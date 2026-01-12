# 从源代码构建

--8<-- "pbm-install-nodes.md"

## 开始之前

检查[系统要求](../system-requirements.md) 和[支持的 MongoDB 版本](../details/versions.md)。

## 先决条件 

要从源代码构建 Percona Backup for MongoDB，您需要以下内容：

* Go 1.19 或更高版本。[安装和设置 Go 工具 :octicons-link-external-16:](https://golang.org/doc/install)
* make
* git
* Red Hat Enterprise Linux / CentOS 的 `krb5-devel` 或 Debian / Ubuntu 的 `libkrb5-dev`。此包是 Percona Server for MongoDB 中 Kerberos 身份验证所必需的。

## 步骤

以下是构建 Percona Backup for MongoDB 的方法：
{.power-number}

1. 克隆仓库

    ```bash
    git clone https://github.com/percona/percona-backup-mongodb
    ```

2. 转到项目目录并构建它

    ```bash
    cd percona-backup-mongodb
    make build
    ```

**make** 完成后，您可以在 `./bin` 目录中找到 `pbm` 和 `pbm-agent` 二进制文件。 

3. 检查 Percona Backup for MongoDB 是否已正确构建并可以使用。 

    ```bash
    cd bin
    ./pbm version
    ```

    ??? example "输出"    

    ```{.text .no-copy}
    Version:   [pbm version number]
    Platform:  linux/amd64
    GitCommit: [commit hash]
    GitBranch: main
    BuildTime: [time when this version was produced in UTC format]
    GoVersion: [Go version number]
    ```

    !!! tip    

        除了指定 pbm 二进制文件的路径，您还可以将其添加到 PATH 环境变量：    

        ```bash
        export PATH=/percona-backup-mongodb/bin:$PATH
        ```

## 安装后步骤

=== ":material-debian: 在 Debian 和 Ubuntu 上"
    
    安装后，执行以下操作：
    {.power-number}

     1. 创建环境文件：

         ```bash
         touch /etc/default/pbm-agent
         ```

     2. 创建 `pbm-agent.service` systemd 单元文件。

         ```bash
         sudo vim /lib/systemd/system/pbm-agent.service
         ```

     3. 在 `pbm-agent.service` 文件中，指定以下内容：

         ```init
         [Unit]
         Description=pbm-agent
         After=time-sync.target network.target

         [Service]
         EnvironmentFile=-/etc/default/pbm-agent
         Type=simple
         User=mongod
         Group=mongod
         PermissionsStartOnly=true
         ExecStart=/usr/bin/pbm-agent

         [Install]
         WantedBy=multi-user.target
         ```
         
        !!! note

            确保 `ExecStart` 目录包含 Percona Backup for MongoDB 二进制文件。否则，请从安装路径的 `./bin` 目录复制它们。

     4. 让 `systemd` 知道新服务：

         ```bash
         sudo systemctl daemon-reload
         ```

=== ":material-redhat: 在 Red Hat Enterprise Linux 和衍生版本上"

    安装后，执行以下操作：
    {.power-number}

    1. 创建环境文件：
   
        ```bash
        touch /etc/sysconfig/pbm-agent
        ```

    2. 创建 `pbm-agent.service` systemd 单元文件。

        ```bash
        sudo vim /usr/lib/systemd/system/pbm-agent.service
        ```

    3. 在 `pbm-agent.service` 文件中，指定以下内容：

         ```init
         [Unit]
         Description=pbm-agent
         After=time-sync.target network.target

         [Service]
         EnvironmentFile=-/etc/default/pbm-agent
         Type=simple
         User=mongod
         Group=mongod
         PermissionsStartOnly=true
         ExecStart=/usr/bin/pbm-agent

         [Install]
         WantedBy=multi-user.target
         ```
         
        !!! note

            确保 `ExecStart` 目录包含 Percona Backup for MongoDB 二进制文件。否则，请从安装路径的 `./bin` 目录复制它们。

     4. 让 `systemd` 知道新服务：

         ```bash
         sudo systemctl daemon-reload
         ```

## 下一步

[初始设置 :material-arrow-right:](initial-setup.md){.md-button}
