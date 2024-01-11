# Flink 调度

## Flink 执行图

Flink 中的执行图可以分成四层：`StreamGraph` -> `JobGraph` -> `ExecutionGraph` -> 物理执行图。

### StreamGraph

根据用户通过 Stream API 编写的代码生成的最初的图。用来表示程序的拓扑结构。

### JobGraph

`StreamGraph` 经过优化后生成了 `JobGraph`，提交给 `JobManager` 的数据结构。 主要的优化为，将多个符合条件的节点 chain 在一起作为一个节点，这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗。

### ExecutionGraph

`JobManager` 根据 `JobGraph` 生成 `ExecutionGraph`。`ExecutionGraph` 是 `JobGraph` 的并行化版本，是调度层最核心的数据结构。

### 物理执行图

`JobManager` 根 据 `ExecutionGraph` 对 Job 进行调度后， 在各个 `TaskManager` 上部署 `Task` 后形成的“图”，并不是一个具体的数据结构。

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption><p>Flink 官方文档中对执行图的介绍</p></figcaption></figure>

## 调度器

调度器在 `JobMaster` 中。调度器是 Flink 作业执行的核心组件，管理作业执行的所有相关过程，包括 `JobGraph` 到 `ExecutionGraph` 的转换、作业生命周期管理（作业的发布、取消、停止）、作业的 `Task` 生命周期管理（`Task` 的发布、取消、停止) 、资源申请与释放、作业和 `Task` 的容错等。

