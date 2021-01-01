[TOC]

# 0. 名词解释

数据处理的方式:

+ 流式(Streaming)数据处理

+ 批量(batch)数据处理

数据处理延迟的长短:

+ 实时数据处理:毫秒级别

+ 离线数据处理:小时 or 天级别

SparkStreaming 是一个准实时(秒、分钟为单位)、微批次的数据处理框架



# 1. **SparkStreaming** **概述**

## 1.1 **Spark Streaming** **是什么**

![image-20210101103145449](.\images\112.png)

​		Spark 流使得构建可扩展的容错流应用程序变得更加容易。

​		Spark Streaming 用于流式数据的处理。Spark Streaming 支持的数据输入源很多，例如：Kafka、Flume、Twitter、ZeroMQ 和简单的 TCP 套接字等等。数据输入后可以用 Spark 的高度抽象原语如：map、reduce、join、window 等进行运算。而结果也能保存在很多地方，如 HDFS，数据库等。

![image-20210101103304551](.\images\113.png)

​		和 Spark 基于 RDD 的概念很相似，Spark Streaming 使用离散化流(discretized stream)作为抽象表示，叫作 DStream。DStream 是随时间推移而收到的数据的序列。在内部，每个时间区间收到的数据都作为 RDD 存在，而 DStream 是由这些 RDD 所组成的序列(因此得名“离散化”)。所以简单来将，DStream 就是对 RDD 在实时数据处理场景的一种封装。

## 1.2 **Spark Streaming** **的特点**

➢ **易用**

![image-20210101103431377](.\images\114.png)

➢ **容错**

![image-20210101103512616](.\images\115.png)

➢ **易整合到** **Spark** **体系**

![image-20210101103609219](.\images\116.png)

## 1.3**Spark Streaming** **架构**

### 1.3.1 **架构图**

➢ **整体架构图**

![image-20210101105734583](.\images\117.png)

➢ **SparkStreaming** **架构图**

![image-20210101105839087](.\images\118.png)

### 1.3.2 背压机制

​		Spark 1.5 以前版本，用户如果要限制 Receiver 的数据接收速率，可以通过设置静态配制参数 “spark.streaming.receiver.maxRate”的值来实现，此举虽然可以通过限制接收速率，来适配当前的处理能力，防止内存溢出，但也会引入其它问题。比如：producer 数据生产高于 maxRate，当前集群处理能力也高于 maxRate，这就会造成资源利用率下降等问题。

​		为了更好的协调数据接收速率与资源处理能力，1.5 版本开始 Spark Streaming 可以动态控制数据接收速率来适配集群数据处理能力。背压机制（即 Spark Streaming Backpressure）: 根据JobScheduler 反馈作业的执行信息来动态调整Receiver 数据接收率。

​		通过属性“spark.streaming.backpressure.enabled”来控制是否启用 backpressure 机制，默认值false，即不启用。

