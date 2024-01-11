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

## JobMaster

`JobMaster` 负责单个作业的管理，提供了对作业的管理行为，允许通过外部的命令干预作业的运行，如提交、取消等。同时 `JobMaster` 也维护了整个作业及其 `Task` 的状态，对外提供对作业状态的查询功能。`JobMaster` 负责接收 `JobGraph`，并将其转换为 `ExecutionGraph` ，启动调度器执行 `ExecutionGraph` 。

1. `JobMasterService` 接口定义了 `JobMaster` 启动和停止等方法。
2. `JobMasterGateway` 接口定义了 `JobMaster` 的 `RPC` 接口方法。
3. `JobMaster` 通过继承 `FencedRpcEndpoint` 抽象实现类，使得 `JobMaster` 成为 `RPC` 服务节点，这样其他组件就可以通过 `RPC` 的通信方式与 `JobMaster` 进行交互了。

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## TaskManager

作为整个运行时的工作节点，`TaskManager` 提供了作业运行过程中需要的 `Slot` 计算资源，`JobManager`中提交的 `Task` 实例都会运行在 `TaskManager` 组件上。

