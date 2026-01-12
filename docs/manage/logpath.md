# 将 pbm-agent 日志输出到文件

!!! admonition "版本添加：[2.8.0](../release-notes/2.8.0.md)"

默认情况下，每个 `pbm-agent` 将其日志写入以下路径：

* `admin.pbmLogs` PBM 控制集合，用作集中式日志存储
* 运行代理的每个主机上的 `STDERR` 流

您可以配置每个 `pbm-agent` 将日志信息输出到自定义路径的文件。此增强功能使您能够实现以下目标：

* **优化系统日志记录**，因为 `pbm-agent` 日志存储在单独的文件中，不会与其他系统日志混合
* **引入日志轮转策略**，以有效使用存储空间 
* **简化日志收集过程**，以便进一步分析
* **通过将日志存储在安全位置所需的时间段来满足日志记录和审计要求**

## 日志配置定义

您可以在启动 `pbm-agent` 时定义日志配置，如下所示：

* 使用命令行标志。此设置具有最高优先级，并覆盖其他设置选项。
* 使用环境变量 
* 通过配置文件。此设置从版本 2.9.0 开始可用。

## 日志配置选项

日志配置选项如下：

| 环境变量 | 命令行标志 | 配置文件选项 |  描述 | 
|----------------------|-------------------|-------------------|-------------|
| `LOG_PATH` | `--log-path` | `log.path` | 日志文件的路径。如果文件不存在，则创建该文件。默认值为 `/dev/stderr`，这意味着日志写入标准错误输出。 |
| `LOG_LEVEL` | `--log-level` | `log.level`| 日志严重性级别。支持的级别（从低到高）：D - Debug（默认）、I - Info、W - Warning、E - Error、F - Fatal。<br><br> 输出包括指定的严重性级别和所有更高级别 |
| `LOG_JSON`| `--log-json` | `log.json` | 以 JSON 格式输出日志消息。如果未定义，日志以默认文本格式写入。 |

## 示例

使用以下命令在每个节点上启动 `pbm-agent`：


=== ":material-console: 命令行"

	```bash
	pbm-agent --log-path=/var/log/pbm-agent.log --log-level=W --log-json
	```

=== ":material-variable: 环境变量"

	```bash
	export LOG_PATH=/var/log/pbm-agent.log
	export LOG_LEVEL=W
	export LOG_JSON=1
	pbm-agent
	```

=== ":octicons-file-code-24: 配置文件"

	1. 创建配置文件。例如，`/etc/pbm-agent.yaml`

    	```yaml title="/etc/pbm-agent.yaml"
        log:
           path: "/var/log/pbm.json"
           level: "I"
           json: true
    	```
    
    2. 启动 `pbm-agent`：

        ```bash
		pbm-agent -f /etc/pbm-agent.yaml
		```


如果 PBM 由于某种错误无法写入指定文件，它会回退到 `STDERR` 流。您始终可以使用 `pbm logs` 命令检索日志信息。

此将日志输出到自定义路径文件的能力增强了日志管理，使有效监控和审计 PBM 操作变得更加容易，并有助于合规性。

## 配置重新加载

您可以通过配置文件动态更改日志级别和输出格式。 

更新配置文件中的所需选项。PBM 检测更改并应用它们，从而无需重启代理。
