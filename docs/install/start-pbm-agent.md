# 启动 `pbm-agent` 进程

在安装了 `mongod` 节点的每个服务器上启动 `pbm-agent`。 

=== ":material-console: 使用 `systemd`"

    我们建议使用打包的服务脚本来运行 `pbm-agent`。
    
    ```bash
    sudo systemctl start pbm-agent
    sudo systemctl status pbm-agent
    ```

=== ":fontawesome-solid-user-gear: 手动"

    您可以手动启动 `pbm-agent`。您必须以 `mongod` 用户身份启动 `pbm-agent`，因为 `pbm-agent` 需要对 MongoDB 数据目录的写访问权限才能进行物理恢复。 

    以下命令启动 `pbm-agent`，将输出重定向到文件并将进程置于后台：
    
    ```bash
    su mongod nohup pbm-agent --mongodb-uri "mongodb://username:password@localhost:27018/" > /data/mdb_node_xyz/pbm-agent.$(hostname -s).27018.log 2>&1 &
    ```
    
    将 `username` 和 `password` 替换为您的 `pbm` 用户的凭据。`/data/mdb_node_xyz/` 是 `pbm-agent` 日志文件将写入的路径。确保您已创建此目录并授予 `mongod` 用户写权限。
    
    或者，如果您想从日志消息中观察和/或调试启动，可以临时在 shell 终端上运行 `pbm-agent`。

## 同一主机上的多个代理

假设您在与另一个 `mongod` 进程（监听端口 `27018`）相同的主机上运行配置服务器（监听端口 `27019`）。 

在这种情况下，应该有两个 `pbm-agent` 进程：

* 一个进程连接到 `mongod` 进程（例如 `"mongodb://username:password@localhost:27018/"`） 
* 另一个连接到 configsvr 节点（例如 `"mongodb://username:password@localhost:27019/"`）。

以下步骤说明如何实现这一点：

1. 为每个代理设置 MongoDB 连接字符串 URI：

    ```bash
    tee /etc/sysconfig/pbm-agent1 <<EOF
    PBM_MONGODB_URI="mongodb://backupUser:backupPassword@localhost:27018/?authSource=admin"
    EOF
    ```

    ```bash
    tee /etc/sysconfig/pbm-agent2 <<EOF
    PBM_MONGODB_URI="mongodb://backupUser:backupPassword@localhost:27019/?authSource=admin"
    EOF
    ```

2. 为每个代理准备服务文件：

    ```bash
    tee /usr/lib/systemd/system/pbm-agent1.service <<EOF
    [Unit]
    Description=pbm-agent for mongod1
    After=time-sync.target network.target    

    [Service]
    EnvironmentFile=-/etc/sysconfig/pbm-agent1
    Type=simple
    User=mongod
    Group=mongod
    PermissionsStartOnly=true
    ExecStart=/usr/bin/pbm-agent    

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
    
    ```bash
    tee /usr/lib/systemd/system/pbm-agent2.service <<EOF
    [Unit]
    Description=pbm-agent for mongod2
    After=time-sync.target network.target    

    [Service]
    EnvironmentFile=-/etc/sysconfig/pbm-agent2
    Type=simple
    User=mongod
    Group=mongod
    PermissionsStartOnly=true
    ExecStart=/usr/bin/pbm-agent    

    [Install]
    WantedBy=multi-user.target
    EOF
    ```

3. 重新加载系统单元并启动 `pbm-agent` 进程： 

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl start pbm-agent1
    sudo systemctl start pbm-agent2
    ```

## 如何查看 `pbm-agent` 日志

使用打包的 `systemd` 服务，输出到 `stdout` 的日志由 systemd 的默认重定向到 `systemd-journald` 捕获。您可以使用以下命令查看它。有关 `--lines`、`--follow` 等有用选项，请参阅 `man journalctl`。

```bash
journalctl -u pbm-agent.service
-- Logs begin at Tue {{year}}-01-22 09:31:34 JST. --
Jan 22 15:59:14 : Started pbm-agent.
Jan 22 15:59:14 pbm-agent[3579]: pbm agent is listening for the commands
...
...
```

如果您手动启动了 `pbm-agent`，请查看您将 `stdout` 和 `stderr` 重定向到的文件。

当消息 `pbm agent is listening for the commands` 打印到 `pbm-agent` 日志文件时，`pbm-agent` 确认它已成功连接到其 `mongod` 节点。

## 下一步 

[创建备份 :material-arrow-right:](../usage/backup-physical.md){.md-button}

