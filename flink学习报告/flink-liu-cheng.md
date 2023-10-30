---
description: 从一个例子开始
---

# Flink流程

笔者以单机模式在本地部署Flink，提交Flink官方给出的例子WordCount.jar

## Flink启动

运行以下命令就可以以单机模式在本地部署Flink

```bash
$ ./bin/start-cluster.sh
Starting cluster.
Starting standalonesession daemon on host LAPTOP-XIAOXIN-KONJAC.
Starting taskexecutor daemon on host LAPTOP-XIAOXIN-KONJAC.
```

此时可以通过访问`localhost:8081` 查看Flink的webUI

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption><p>flink webUI</p></figcaption></figure>

在`start-cluster.sh` 中执行了以下命令

## WorldCount

{% code lineNumbers="true" %}
```java
case class WordWithCount(word: String, count: Long)

val text = env.socketTextStream(host, port, '\n')

val windowCounts = text.flatMap { w => w.split("\\s") }
  .map { w => WordWithCount(w, 1) }
  .keyBy("word")
  .window(TumblingProcessingTimeWindow.of(Time.seconds(5)))
  .sum("count")

windowCounts.print()
```
{% endcode %}

第3行text从主机端口每次取文本取到`\n`&#x20;

第5-9行将文本分割成单词，然后设置时间窗口5秒钟，用该滚动窗口分割数据流，统计单词出现的次数。

第11行将结果输出

## 环境准备与作业提交

执行以下命令将WorldCount提交

```bash
$ ./bin/flink run examples/streaming/WordCount.jar
Job has been submitted with JobID c35dca1d97498807a1e0e17f1f1cedfa
Program execution finished
Job with JobID c35dca1d97498807a1e0e17f1f1cedfa has finished.
Job Runtime: 601 ms
```

#### 程序起点

在`./bin/flink` 中执行了以下命令

```bash
exec "${JAVA_RUN}" $JVM_ARGS $FLINK_ENV_JAVA_OPTS "${log_setting[@]}" -classpath "`manglePathList "$CC_CLASSPATH:$INTERNAL_HADOOP_CLASSPATHS"`" org.apache.flink.client.cli.CliFrontend "$@"
```

其运行了`org.apache.flink.client.cli.CliFrontend` 的main方法。

{% code lineNumbers="true" %}
```java
/** Submits the job based on the arguments. */
    public static void main(final String[] args) {
        int retCode = INITIAL_RET_CODE;
        try {
            retCode = mainInternal(args);
        } finally {
            System.exit(retCode);
        }
    }
```
{% endcode %}
