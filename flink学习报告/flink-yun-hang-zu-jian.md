---
description: 本节主要分析 Flink 运行组件的设计
---

# Flink 运行组件

## Dispatcher

### 功能示意图

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption><p>Dispatcher 功能示意图</p></figcaption></figure>

### 类关系

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

### Dispatcher

负责对集群中的作业进行接收和分发处理操作，客 户端可以通过与 `Dispatcher` 建立 `RPC` 连接，将作业过 `ClusterClient` 提交到集群 `Dispatcher` 服务中。`Dispatcher` 通过 `JobGraph` 对象启动 `JobManager` 服务。

### DispatcherRunner

负责启动和管理 `Dispatcher` 组件，并支持对 `Dispatcher` 组件的 `Leader` 选举。当 `Dispatcher` 集群组件出现异常并停止时，会通过 `DispatcherRunner` 重新选择和启动新的 `Dispatcher` 服务，从而保证 `Dispatcher` 组件的高可用。

### DispatcherLeaderProcess

负责管理 `Dispatcher` 生命周期，同时提 供了对 `JobGraph` 的任务恢复管理功能。如果基于 `ZooKeeper` 实现了集群高可用， `DispatcherLeaderProcess` 会将提交的 `JobGraph` 存储在 `ZooKeeper` 中，当集群停止或者出现异常时，就会通过 `DispatcherLeaderProcess` 对集群中的 `JobGraph` 进行恢复，这些 `JobGraph` 都会被存储在 `JobGraphStore` 的实现类中。

### DispatcherGatewayService

主要基于`Dispatcher` 实现的 `GatewayService`，用于获取 `DispatcherGateway`。

## ResourceManager

`ResourceManager` 作为集群资源管理组件，其内部创建 `ResourceManagerRuntimeServices`。 `ResourceManagerRuntimeServices` 中包含 `SlotManager` 和 `JobLeaderIdService` 两个主要服务。其中 `SlotManager` 服务管理整个集群的 `Slot` 计算资源，并对 `Slot` 计算资源进行统一的分配和管理； `JobLeaderIdService` 通过实现 `jobLeaderIdListeners` 实时监听 `JobManager` 的运行状态，以获取集群启动的作业对应的 `JobLeaderId` 信息，防止出现 `JobManager` 无法连接的情况。

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption><p>ResourceManager</p></figcaption></figure>
