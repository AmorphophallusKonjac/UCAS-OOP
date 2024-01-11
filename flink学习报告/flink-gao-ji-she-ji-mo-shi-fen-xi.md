# Flink 高级设计模式分析

## 高级设计模式

### 工厂模式

Flink 中工厂模式很常见，很多组件的创建都使用了工厂模式。

下图是 `JobMaster` 的创建过程中使用的工厂模式。

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

工厂模式的使用提升了 `JobMaster` 的可维护性和可扩展性，符合开闭原则。

### 代理模式

Flink 通过 Akka 进行的分布式通信的实现，其基于 Akka 实现了 RPC 框架。其设计模式就是代理模式。

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption><p>Flink 分布式通信框架结构图。（该图来源于尚硅谷课程）</p></figcaption></figure>

### 策略模式

针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以相互替换。策略模式使得算法可以在不影响到客户端的情况下发生变化。

Flink 的调度策略使用了策略模式。

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
