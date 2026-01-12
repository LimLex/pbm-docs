## 在哪些节点上安装

### `pbm-agent`

在 MongoDB 集群（或非分片副本集）中所有具有 `mongod` 节点的服务器上安装 [`pbm-agent`](../details/pbm-agent.md)。您不需要在 `arbiter` 节点上启动它，因为它没有数据集。

### `pbm` CLI

您可以在任何或所有您希望使用它的服务器或台式计算机上安装 [`pbm` CLI](../details/cli.md)。这些计算机必须不被网络阻止访问 MongoDB 集群。
