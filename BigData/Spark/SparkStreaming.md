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

# 2. **Dstream** **入门**

## 2.1  **WordCount** **案例实操**

需求：使用 netcat 工具向 9999 端口不断的发送数据，通过 SparkStreaming 读取端口数据并统计不同单词出现的次数

1. 添加依赖

   ```xml
   <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.12</artifactId>
    <version>3.0.0</version>
   </dependency>
   ```

2. 编写代码

   ```scala
   package net.codeshow.spark.streaming
   
   object SparkStreaming01_WordCount {
     def main(args: Array[String]): Unit = {
       import org.apache.spark.SparkConf
       import org.apache.spark.streaming.{Seconds, StreamingContext}
   
       //    TODO 创建环境对象
       val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
       //    第一个参数表示环境配置
       //    第二个参数表示批量处理的周期(采集周期)
       val ssc = new StreamingContext(sparkConf, Seconds(3))
   
       //    TODO 逻辑处理
       //    获取端口数据
       val lines = ssc.socketTextStream("localhost", 9999)
       val words = lines.flatMap(_.split(" "))
       val wordToOne = words.map((_, 1))
       val wordToCount = wordToOne.reduceByKey(_ + _)
       wordToCount.print()
   
       //由于SparkStreaming是一个长期执行的任务，所以不能关闭
       //    如果main方法执行完毕，应用程序也会自动结束，所以不能让main方法执行完毕
       //    ssc.stop()
       //    1.启动采集器
       ssc.start()
       //    2.等待采集器关闭
       ssc.awaitTermination()
     }
   }
   ```

3. 启动程序并通过 netcat 发送数据

   ```shell
   nc -lk 9999
   hello spark
   ```

## 2.2 **WordCount** **解析**

​		Discretized Stream 是 Spark Streaming 的基础抽象，代表持续性的数据流和经过各种 Spark 原语操作后的结果数据流。在内部实现上，DStream 是一系列连续的 RDD 来表示。每个 RDD 含有一段时间间隔内的数据。

![image-20210101113626755](.\images\119.png)

对数据的操作也是按照 RDD 为单位来进行的

![image-20210101113704061](.\images\120.png)

计算过程由 Spark Engine 来完成

![image-20210101113749516](.\images\121.png)



# 3.**DStream** **创建**

## 3.1 **RDD** **队列**

### 3.1.1  **用法及说明**

​		测试过程中，可以通过使用 ssc.queueStream(queueOfRDDs)来创建 DStream，每一个推送到这个队列中的 RDD，都会作为一个 DStream 处理。

### 3.1.2 **案例实操**

需求：循环创建几个 RDD，将 RDD 放入队列。通过 SparkStream 创建 Dstream，计算WordCount

1. 编写代码

   ```scala
   object RDDStream {
    def main(args: Array[String]) {
    //1.初始化 Spark 配置信息
    val conf = new SparkConf().setMaster("local[*]").setAppName("RDDStream")
    //2.初始化 SparkStreamingContext
    val ssc = new StreamingContext(conf, Seconds(4))
    //3.创建 RDD 队列
    val rddQueue = new mutable.Queue[RDD[Int]]()
    //4.创建 QueueInputDStream
    val inputStream = ssc.queueStream(rddQueue,oneAtATime = false)
    //5.处理队列中的 RDD 数据
    val mappedStream = inputStream.map((_,1))
    val reducedStream = mappedStream.reduceByKey(_ + _)
    //6.打印结果
    reducedStream.print()
    //7.启动任务
    ssc.start()
   //8.循环创建并向 RDD 队列中放入 RDD
    for (i <- 1 to 5) {
    rddQueue += ssc.sparkContext.makeRDD(1 to 300, 10)
    Thread.sleep(2000)
    }
    ssc.awaitTermination()
    } }
   ```

2. 结果展示

   ```shell
   -------------------------------------------
   Time: 1539075280000 ms
   -------------------------------------------
   (4,60)
   (0,60)
   (6,60)
   (8,60)
   (2,60)
   (1,60)
   (3,60)
   (7,60)
   (9,60)
   (5,60)
   -------------------------------------------
   Time: 1539075284000 ms
   -------------------------------------------
   (4,60)
   (0,60)
   (6,60)
   (8,60)
   (2,60)
   (1,60)
   (3,60)
   (7,60)
   (9,60)
   (5,60)
   -------------------------------------------
   Time: 1539075288000 ms
   -------------------------------------------
   (4,30)
   (0,30)
   (6,30)
   (8,30)
   (2,30)
   (1,30)
   (3,30)
   (7,30)
   (9,30)
   (5,30)
   -------------------------------------------
   Time: 1539075292000 ms
   -------------------------------------------
   ```

##  3.2**自定义数据源**

### 3.2.1 **用法及说明**

   需要继承 Receiver，并实现 onStart、onStop 方法来自定义数据源采集。

### 3.2.2 **案例实操**

需求：自定义数据源，实现监控某个端口号，获取该端口号内容。

1.  自定义数据源

   ```scala
   class CustomerReceiver(host: String, port: Int) extends 
   Receiver[String](StorageLevel.MEMORY_ONLY) {
       //最初启动的时候，调用该方法，作用为：读数据并将数据发送给 Spark
    override def onStart(): Unit = {
    new Thread("Socket Receiver") {
    override def run() {
    receive()
    }
    }.start()
    }
    //读数据并将数据发送给 Spark
    def receive(): Unit = {
    //创建一个 Socket
    var socket: Socket = new Socket(host, port)
    //定义一个变量，用来接收端口传过来的数据
    var input: String = null
    //创建一个 BufferedReader 用于读取端口传来的数据
    val reader = new BufferedReader(new InputStreamReader(socket.getInputStream, 
   StandardCharsets.UTF_8))
    //读取数据
    input = reader.readLine()
    //当 receiver 没有关闭并且输入数据不为空，则循环发送数据给 Spark
    while (!isStopped() && input != null) {
    store(input)
    input = reader.readLine()
    }
    //跳出循环则关闭资源
    reader.close()
    socket.close()
    //重启任务
    restart("restart")
    }
    override def onStop(): Unit = {}
   }
   ```

2. 使用自定义的数据源采集数据

   ```scala
   object FileStream {
    def main(args: Array[String]): Unit = {
    //1.初始化 Spark 配置信息
   val sparkConf = new SparkConf().setMaster("local[*]")
   .setAppName("StreamWordCount")
    //2.初始化 SparkStreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(5))
   //3.创建自定义 receiver 的 Streaming
   val lineStream = ssc.receiverStream(new CustomerReceiver("hadoop102", 9999))
    //4.将每一行数据做切分，形成一个个单词
    val wordStream = lineStream.flatMap(_.split("\t"))
    //5.将单词映射成元组（word,1）
        val wordAndOneStream = wordStream.map((_, 1))
    //6.将相同的单词次数做统计
    val wordAndCountStream = wordAndOneStream.reduceByKey(_ + _)
    //7.打印
    wordAndCountStream.print()
    //8.启动 SparkStreamingContext
    ssc.start()
    ssc.awaitTermination()
    } }
   ```

   ## 3.3 **Kafka** 数据源（面试、开发重点）

   ### 3.3.1 **版本选型**

   ​		ReceiverAPI：需要一个专门的 Executor 去接收数据，然后发送给其他的 Executor 做计算。存在的问题，接收数据的 Executor 和计算的 Executor 速度会有所不同，特别在接收数据的 Executor速度大于计算的 Executor 速度，会导致计算数据的节点内存溢出。早期版本中提供此方式，当前版本不适用

   ​		DirectAPI：是由计算的 Executor 来主动消费 Kafka 的数据，速度由自身控制。

   ### 3.3.2 **Kafka 0-8 Receiver** 模式（当前版本不适用） 

   1. 需求：通过 SparkStreaming 从 Kafka 读取数据，并将读取过来的数据做简单计算，最终打印到控制台。

   2. 导入依赖

      ```xml
      <dependency>
       <groupId>org.apache.spark</groupId>
       <artifactId>spark-streaming-kafka-0-8_2.11</artifactId>
       <version>2.4.5</version>
      </dependency>
      ```

   3. 编写代码

      ```scala
      package com.atguigu.kafka
      import org.apache.spark.SparkConf
      import org.apache.spark.streaming.dstream.ReceiverInputDStream
      import org.apache.spark.streaming.kafka.KafkaUtils
      import org.apache.spark.streaming.{Seconds, StreamingContext}
      object ReceiverAPI {
       def main(args: Array[String]): Unit = {
       //1.创建 SparkConf
       val sparkConf: SparkConf = new 
      SparkConf().setAppName("ReceiverWordCount").setMaster("local[*]")
       //2.创建 StreamingContext
           val ssc = new StreamingContext(sparkConf, Seconds(3))
       //3.读取 Kafka 数据创建 DStream(基于 Receive 方式)
       val kafkaDStream: ReceiverInputDStream[(String, String)] = 
      KafkaUtils.createStream(ssc,
       "linux1:2181,linux2:2181,linux3:2181",
       "atguigu",
       Map[String, Int]("atguigu" -> 1))
       //4.计算 WordCount
       kafkaDStream.map { case (_, value) =>
       (value, 1)
       }.reduceByKey(_ + _)
       .print()
       //5.开启任务
       ssc.start()
       ssc.awaitTermination()
       } }
      ```

### 3.3.3  Kafka 0-8 Direct 模式（当前版本不适用） 

1. 需求：通过 SparkStreaming 从 Kafka 读取数据，并将读取过来的数据做简单计算，最终打印到控制台。

2. 导入依赖

   ```xml
   <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-8_2.11</artifactId>
    <version>2.4.5</version>
   </dependency>
   ```

3. 编写代码（自动维护 offset）

   ```scala
   import kafka.serializer.StringDecoder
   import org.apache.kafka.clients.consumer.ConsumerConfig
   import org.apache.spark.SparkConf
   import org.apache.spark.streaming.dstream.InputDStream
   import org.apache.spark.streaming.kafka.KafkaUtils
   import org.apache.spark.streaming.{Seconds, StreamingContext}
   object DirectAPIAuto02 {
    val getSSC1: () => StreamingContext = () => {
    val sparkConf: SparkConf = new 
   SparkConf().setAppName("ReceiverWordCount").setMaster("local[*]")
    val ssc = new StreamingContext(sparkConf, Seconds(3))
    ssc
    }
    def getSSC: StreamingContext = {
    //1.创建 SparkConf
    val sparkConf: SparkConf = new 
   SparkConf().setAppName("ReceiverWordCount").setMaster("local[*]")
    //2.创建 StreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(3))
        //设置 CK
    ssc.checkpoint("./ck2")
    //3.定义 Kafka 参数
    val kafkaPara: Map[String, String] = Map[String, String](
    ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> 
   "linux1:9092,linux2:9092,linux3:9092",
    ConsumerConfig.GROUP_ID_CONFIG -> "atguigu"
    )
    //4.读取 Kafka 数据
    val kafkaDStream: InputDStream[(String, String)] = 
   KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc,
    kafkaPara,
    Set("atguigu"))
    //5.计算 WordCount
    kafkaDStream.map(_._2)
    .flatMap(_.split(" "))
    .map((_, 1))
    .reduceByKey(_ + _)
    .print()
    //6.返回数据
    ssc
    }
    def main(args: Array[String]): Unit = {
    //获取 SSC
    val ssc: StreamingContext = StreamingContext.getActiveOrCreate("./ck2", () => 
   getSSC)
    //开启任务
    ssc.start()
    ssc.awaitTermination()
    } }
   ```

4. 编写代码（手动维护 offset）

   ```scala
   import kafka.common.TopicAndPartition
   import kafka.message.MessageAndMetadata
   import kafka.serializer.StringDecoder
   import org.apache.kafka.clients.consumer.ConsumerConfig
   import org.apache.spark.SparkConf
   import org.apache.spark.streaming.dstream.{DStream, InputDStream}
   import org.apache.spark.streaming.kafka.{HasOffsetRanges, KafkaUtils,
   OffsetRange}
   import org.apache.spark.streaming.{Seconds, StreamingContext}
   object DirectAPIHandler {
    def main(args: Array[String]): Unit = {
    //1.创建 SparkConf
    val sparkConf: SparkConf = new 
   SparkConf().setAppName("ReceiverWordCount").setMaster("local[*]")
    //2.创建 StreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(3))
        //3.Kafka 参数
    val kafkaPara: Map[String, String] = Map[String, String](
    ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> 
   "hadoop102:9092,hadoop103:9092,hadoop104:9092",
    ConsumerConfig.GROUP_ID_CONFIG -> "atguigu"
    )
    //4.获取上一次启动最后保留的 Offset=>getOffset(MySQL)
    val fromOffsets: Map[TopicAndPartition, Long] = Map[TopicAndPartition, 
   Long](TopicAndPartition("atguigu", 0) -> 20)
    //5.读取 Kafka 数据创建 DStream
    val kafkaDStream: InputDStream[String] = KafkaUtils.createDirectStream[String, 
   String, StringDecoder, StringDecoder, String](ssc,
    kafkaPara,
    fromOffsets,
    (m: MessageAndMetadata[String, String]) => m.message())
    //6.创建一个数组用于存放当前消费数据的 offset 信息
    var offsetRanges = Array.empty[OffsetRange]
    //7.获取当前消费数据的 offset 信息
    val wordToCountDStream: DStream[(String, Int)] = kafkaDStream.transform { rdd 
   =>
    offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
    rdd
    }.flatMap(_.split(" "))
    .map((_, 1))
    .reduceByKey(_ + _)
    //8.打印 Offset 信息
    wordToCountDStream.foreachRDD(rdd => {
    for (o <- offsetRanges) {
    println(s"${o.topic}:${o.partition}:${o.fromOffset}:${o.untilOffset}")
    }
    rdd.foreach(println)
    })
    //9.开启任务
    ssc.start()
    ssc.awaitTermination()
    } }
   ```

### 3.3.4  Kafka 0-10 Direct 模式

1. 需求：通过 SparkStreaming 从 Kafka 读取数据，并将读取过来的数据做简单计算，最终打印到控制台。

2. 导入依赖

   ```xml
   <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-10_2.12</artifactId>
    <version>3.0.0</version>
   </dependency>
   <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.10.1</version>
   </dependency>
   ```

3. 编写代码

   ```scala
   import org.apache.kafka.clients.consumer.{ConsumerConfig, ConsumerRecord}
   import org.apache.spark.SparkConf
   import org.apache.spark.streaming.dstream.{DStream, InputDStream}
   import org.apache.spark.streaming.kafka010.{ConsumerStrategies, KafkaUtils, 
   LocationStrategies}
   import org.apache.spark.streaming.{Seconds, StreamingContext}
   object DirectAPI {
    def main(args: Array[String]): Unit = {
    //1.创建 SparkConf
    val sparkConf: SparkConf = new 
   SparkConf().setAppName("ReceiverWordCount").setMaster("local[*]")
    //2.创建 StreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(3))
    //3.定义 Kafka 参数
    val kafkaPara: Map[String, Object] = Map[String, Object](
    ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> 
   "linux1:9092,linux2:9092,linux3:9092",
    ConsumerConfig.GROUP_ID_CONFIG -> "atguigu",
    "key.deserializer" -> 
   "org.apache.kafka.common.serialization.StringDeserializer",
    "value.deserializer" -> 
   "org.apache.kafka.common.serialization.StringDeserializer"
    )
    //4.读取 Kafka 数据创建 DStream
    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] = 
   KafkaUtils.createDirectStream[String, String](ssc,
    LocationStrategies.PreferConsistent,
    ConsumerStrategies.Subscribe[String, String](Set("atguigu"), kafkaPara))
    //5.将每条消息的 KV 取出
    val valueDStream: DStream[String] = kafkaDStream.map(record => record.value())
    //6.计算 WordCount
    valueDStream.flatMap(_.split(" "))
    .map((_, 1))
    .reduceByKey(_ + _)
    .print()
    //7.开启任务
    ssc.start()
    ssc.awaitTermination()
    } }
   ```

4. 查看 Kafka 消费进度

   ```shell
   bin/kafka-consumer-groups.sh --describe --bootstrap-server linux1:9092 --group 
   atguigu
   ```

   

