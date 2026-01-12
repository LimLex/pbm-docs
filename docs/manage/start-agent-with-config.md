# 使用配置文件启动 pbm-agent

!!! admonition "版本添加：[2.9.0](../release-notes/2.9.0.md)"

`pbm-agent` 需要 MongoDB 连接字符串 URI 才能启动。您可以通过环境变量或使用配置文件来定义它。后一个选项使您能够指定其他配置，如备份的并行连接数和[自定义日志配置](logpath.md)。 

本文档说明如何使用配置文件启动 `pbm-agent`。有关如何使用环境变量，请参阅[在 MongoDB 中配置身份验证](../install/configure-authentication.md#set-the-mongodb-connection-uri-for-pbm-agent)

步骤根据您的安装方法略有不同。在以下示例中，我们提供所有可用的配置选项以供参考。请注意，对于初始 `pbm-agent` 启动，`mongodb-uri` 是必需的。

## 从软件包安装 {.power-number}

1. 创建配置文件。例如，`/etc/pbm-agent.yaml`。 

	```yaml title="/etc/pbm-agent.yaml"
	mongodb-uri: mongodb://pbm:mysecret@localhost:27017/
	backup:
	  dump-parallel-collections: 10
	log:
	  path: "/var/log/pbm.json"
	  level: "I"
	  json: true
	```

2. 修改 `pbm-agent.service` systemd 单元文件。为 `ExecStart` 参数指定配置文件的路径。 

    ```init title="pbm-agent.service"
    ....

    [Service]
    ....
    ExecStart=/usr/bin/pbm-agent -f /etc/pbm-agent.yaml

    ....
    ```

3. 启动 `pbm-agent`。

    ```bash
    sudo systemctl start pbm-agent
    ```

## 从源代码和压缩包安装 {.power-number}

对于初始 `pbm-agent` 启动，您需要 `pbm` 用户的凭据。有关如何创建此用户，请参阅[创建 pbm 用户](../install/configure-authentication.md#create-the-pbm-user) 部分。 

1. 创建配置文件。在 MongoDB 连接字符串 URI 中指定 `pbm` 用户凭据。

	```yaml title="/etc/pbm-agent.yaml"
	mongodb-uri: mongodb://pbm:mysecret@localhost:27017/
	backup:
	  dump-parallel-collections: 10
	log:
	  path: "/var/log/pbm.json"
	  level: "I"
	  json: true
	```

2. 检查您是否已将 PBM 二进制文件的位置导出到 `$PATH` 变量
3. 启动 `pbm-agent`：

    ```bash
    pbm-agent -f /etc/pbm-agent.yaml
	```
