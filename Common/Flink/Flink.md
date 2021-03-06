[TOC]

# 1.流处理简介

## 1.1 Flink 是什么

![image-20210119111834937](./images/1.png)

### 1.1.1 Flink 的全球热度

![image-20210119111952581](./images/2.png)

### 1.1.2 Flink 目前在国内企业的应用

![image-20210119112032333](./images/3.png)

## 1.2 为什么要用 Flink

1. 流数据更真实地反映了我们的生活方式
2. 传统的数据架构是基于有限数据集的
3. 我们的目标
   + 低延迟
   + 高吞吐
   + 结果的准确性和良好的容错性

### 1.2.1 哪些行业需要处理流数据

1. 电商和市场营销

   数据报表、广告投放、业务流程需要

2. 物联网（IOT） 

   传感器实时数据采集和显示、实时报警，交通运输业

3. 电信业

   基站流量调配

4. 银行和金融业

   实时结算和通知推送，实时检测异常行

## 1.3 流处理的发展和演变

### 1.3.1 传统数据处理架构

1. 事务处理

   ![image-20210119112310261](./images/4.png)

2. 分析处理

   将数据从业务数据库复制到数仓，再进行分析和查询

   ![image-20210119112349770](./images/5.png)

### 1.3.2 有状态的流式处理

![image-20210119112421422](./images/6.png)



### 1.3.3 流处理的演变

1. lambda 架构

   用两套系统，同时保证低延迟和结果准确

   ![image-20210119112521080](./images/7.png)

   ![image-20210119112545558](./images/8.png)=

## 1.4 Flink 的主要特点

### 1.4.1 事件驱动（Event-driven）

![image-20210119112637628](./images/9.png)

### 1.4.2 基于流的世界观

在 Flink 的世界观中，一切都是由流组成的，离线数据是有界的流；实时数据是一个没有界限的流：这就是所谓的有界流和无界流

![image-20210119112730063](./images/10.png)

### 1.4.3 分层API

+  越顶层越抽象，表达含义越简明，使用越方便

+ 越底层越具体，表达能力越丰富，使用越灵活

  ![image-20210119112808165](./images/11.png)

### 1.4.4 其它特点

+ 支持事件时间（event-time）和处理时间（processing-time）语义
+ 精确一次（exactly-once）的状态一致性保证
+ 低延迟，每秒处理数百万个事件，毫秒级延迟
+ 与众多常用存储系统的连接
+ 高可用，动态扩展，实现7*24小时全天候运行

## 1.5 Flink vs Spark Streaming

### 1.5.1 流（stream）和微批（micro-batching）

![image-20210119112944536](./images/12.png)

### 1.5.2 数据模型

+ spark 采用 RDD 模型，spark streaming 的 DStream 实际上也就是一组 组小批数据 RDD 的集合
+ flink 基本数据模型是数据流，以及事件（Event）序列

### 1.5.3 运行时架构

+ spark 是批计算，将 DAG 划分为不同的 stage，一个完成后才可以计算下一个
+ flink 是标准的流执行模式，一个事件在一个节点处理完后可以直接发往下一个节点进行处理



# 2. 快速上手

## 2.1 搭建 maven 工程

pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>FlinkStudy</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>1.10.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>
    </dependencies>
</project>
```

## 2.2 批处理 wordcount

```java
package net.codeshow.wc;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.AggregateOperator;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

/**
 * @Description
 * @Author eric
 * @Version V1.0.0
 * @Date 2021/1/19
 */

// 批处理 word count
public class WordCount {
    public static void main(String[] args) throws Exception {
//        创建执行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
//        从文件中读取数据
        String inputPath = "/Users/cuiguangsong/my_files/workspace/java/FlinkStudy/src/main/resources/hello.txt";
        DataSet<String> inputDataSet = env.readTextFile(inputPath);
//        对数据集进行处理，按空格分词展开，转换成(word,1)二元组进行统计
        DataSet<Tuple2<String, Integer>> resultSet = inputDataSet.flatMap(new MyFlatMapper())
//                按照第一个位置的word分组
                .groupBy(0)
//                将第二个位置上的数据求和
                .sum(1);
        resultSet.print();


    }

    //    自定义类，实现FlatMapFunction接口
    public static class MyFlatMapper implements FlatMapFunction<String, Tuple2<String, Integer>> {

        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
//        按照空格分词
            String[] words = value.split(" ");
//            遍历所有word，包装成二元组输出
            for (String word : words) {
                out.collect(new Tuple2<>(word, 1));

            }

        }
    }
}

```

## 2.3 流处理 wordcount

```java
package net.codeshow.wc;

import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.utils.ParameterTool;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

/**
 * @Description
 * @Author eric
 * @Version V1.0.0
 * @Date 2021/1/19
 */
public class StreamWordCount {
    public static void main(String[] args) throws Exception {
//        创建流处理执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
//        String inputPath = "/Users/cuiguangsong/my_files/workspace/java/FlinkStudy/src/main/resources/hello.txt";
//        DataStream<String> inputDataStream = env.readTextFile(inputPath);
//        用parameterTool工具从程序启动参数中提取配置项
        ParameterTool parameterTool = ParameterTool.fromArgs(args);
        String host = parameterTool.get("host");
        int port = parameterTool.getInt("port");

//        从socket文本流读取数据
        DataStream<String> inputDataStream = env.socketTextStream(host, port);
//        基于数据流进行转换计算
        DataStream<Tuple2<String, Integer>> resultStream = inputDataStream.flatMap(new WordCount.MyFlatMapper())
                .keyBy(0)
                .sum(1);
        resultStream.print();
//        执行任务
        env.execute();

    }
}

```

测试——在 linux 系统中用 netcat 命令进行发送测试。

```bash
nc -lk 7777
```

# 3. Flink 部署

## 3.1 Standalone 模式

### 3.1.1 **安装**

解压缩 flink-1.10.1-bin-scala_2.12.tgz，进入 conf 目录中

1. *修改* *flink/conf/flink-conf.yaml* *文件*

   ![image-20210119163357963](./images/13.png)

2. *修改* */conf/slaves* *文件*

   ![image-20210119163420142](./images/14.png)

3. 分发给另外两台机器

   ![image-20210119163438028](./images/15.png)

4. *启动*

   ![image-20210119163501810](./images/16.png)

   访问 http://localhost:8081 可以对 flink 集群和任务进行监控管理。

   ![image-20210119163548369](./images/17.png)



### 3.1.2 **提交任务**

1. *准备数据文件（如果需要）*

   ![image-20210119163726083](./images/18.png)

2. *把含数据文件的文件夹，分发到* *taskmanage* *机器中*

   ![image-20210119163746586](./images/19.png)

   如 果 从 文 件 中 读 取 数 据 ， 由 于 是 从 本 地 磁 盘 读 取 ， 实 际 任 务 会 被 分 发 到taskmanage 的机器中，所以要把目标文件分发

3. *执行程序*

   ```bash
   ./flink run -c com.atguigu.wc.StreamWordCount –p 2 FlinkTutorial-1.0-SNAPSHOT-jar-with-dependencies.jar --host lcoalhost –port 7777
   ```

   ![image-20210119163843491](./images/20.png)

4. *查看计算结果*

   注意：如果输出到控制台，应该在 taskmanager 下查看；如果计算结果输出到文件，同样会保存到 taskmanage 的机器下，不会在 jobmanage 下

   ![image-20210119163927842](./images/21.png)

5. *在* *webui* *控制台查看计算过程*

   ![image-20210119163957214](./images/22.png)

## 3.2 Yarn 模式

以 Yarn 模式部署 Flink 任务时，要求 Flink 是有 Hadoop 支持的版本，Hadoop环境需要保证版本在 2.2 以上，并且集群中安装有 HDFS 服务

### 3.2.1  **Flink on Yarn**

Flink 提供了两种在 yarn 上运行的模式，分别为 Session-Cluster 和 Per-Job-Cluster模式。

1. **Session-cluster** **模式**

   ![image-20210119164145653](./images/23.png)

   Session-Cluster 模式需要先启动集群，然后再提交作业，接着会向 yarn 申请一块空间后，资源永远保持不变。如果资源满了，下一个作业就无法提交，只能等到yarn 中的其中一个作业执行完成后，释放了资源，下个作业才会正常提交。所有作业共享 Dispatcher 和 ResourceManager；共享资源；适合规模小执行时间短的作业

   在 yarn 中初始化一个 flink 集群，开辟指定的资源，以后提交任务都向这里提交。这个 flink 集群会常驻在 yarn 集群中，除非手工停止

2. **Per-Job-Cluster** **模式**

   ![image-20210119164411764](./images/24.png)

   一个 Job 会对应一个集群，每提交一个作业会根据自身的情况，都会单独向 yarn申请资源，直到作业执行完成，一个作业的失败与否并不会影响下一个作业的正常提交和运行。独享 Dispatcher 和 ResourceManager，按需接受资源申请；适合规模大长时间运行的作业

   每次提交都会创建一个新的 flink 集群，任务之间互相独立，互不影响，方便管理。任务执行完成之后创建的集群也会消失

### 3.2.2 **Session Cluster**

1. *启动* *hadoop* *集群（略）*

2. *启动* *yarn-session*

   ```bash
   ./yarn-session.sh -n 2 -s 2 -jm 1024 -tm 1024 -nm test -d
   ```

   其中：

   -n(--container)：TaskManager 的数量。

   -s(--slots)： 每个 TaskManager 的 slot 数量，默认一个 slot 一个 core，默认每个

   taskmanager 的 slot 的个数为 1，有时可以多一些 taskmanager，做冗余。

   -jm：JobManager 的内存（单位 MB)。 

   -tm：每个 taskmanager 的内存（单位 MB)。 

   -nm：yarn 的 appName(现在 yarn 的 ui 上的名字)。 

   -d：后台执行

   ![image-20210119164648536](./images/25.png)

3. *执行任务*

   ```bash
   ./flink run -c com.atguigu.wc.StreamWordCount FlinkTutorial-1.0-SNAPSHOT-jar-with-dependencies.jar --host lcoalhost –port 7777
   ```

4. *去* *yarn* *控制台查看任务状态*

   ![image-20210119164734223](./images/26.png)

5. *取消* *yarn-session*

   ```bash
   yarn application --kill application_1577588252906_0001
   ```

### 3.2.2  **Per Job Cluster**

1. *启动* *hadoop* *集群（略）*

2. **不启动** **yarn-session***，直接执行* *job*

   ```bash
   ./flink run –m yarn-cluster -c com.atguigu.wc.StreamWordCount FlinkTutorial-1.0-SNAPSHOT-jar-with-dependencies.jar --host lcoalhost –port 7777
   ```

## 3.3 Kubernetes 部署

容器化部署时目前业界很流行的一项技术，基于 Docker 镜像运行能够让用户更加 方 便地 对应 用进 行管 理 和运 维。 容器 管理 工 具中 最为 流行 的就 是 Kubernetes（k8s），而 Flink 也在最近的版本中支持了 k8s 部署模式

1）搭建 *Kubernetes* *集群（略）*

2）配置各组件的 *yaml* *文件*

在 k8s 上构建 Flink Session Cluster，需要将 Flink 集群的组件对应的 docker 镜像分别在 k8s 上启动，包括 JobManager、TaskManager、JobManagerService 三个镜像服务。每个镜像服务都可以从中央镜像仓库中获取

3）启动 *Flink Session Cluster*

```bash
// 启动 jobmanager-service 服务
kubectl create -f jobmanager-service.yaml
// 启动 jobmanager-deployment 服务
kubectl create -f jobmanager-deployment.yaml
// 启动 taskmanager-deployment 服务
kubectl create -f taskmanager-deployment.yaml
```

4）访问 *Flink UI* 页面

集群启动后，就可以通过 JobManagerServicers 中配置的 WebUI 端口，用浏览器输入以下 url 来访问 Flink UI 页面了：

http://*{JobManagerHost:Port}*/api/v1/namespaces/default/services/flink-jobmanager:ui/proxy

# 4. Flink 运行架构

## 4.1 Flink 运行时的组件

Flink 运行时架构主要包括四个不同的组件，它们会在运行流处理应用程序时协同工作：

作业管理器（JobManager）、资源管理器（ResourceManager）、任务管理器（TaskManager），以及分发器（Dispatcher）。因为 Flink 是用 Java 和 Scala 实现的，所以所有组件都会运行在Java 虚拟机上。每个组件的职责如下

1. 作业管理器（JobManager）

   控制一个应用程序执行的主进程，也就是说，每个应用程序都会被一个不同的JobManager 所控制执行。JobManager 会先接收到要执行的应用程序，这个应用程序会包括：作业图（JobGraph）、逻辑数据流图（logical dataflow graph）和打包了所有的类、库和其它资源的 JAR 包。JobManager 会把 JobGraph 转换成一个物理层面的数据流图，这个图被叫做“执行图”（ExecutionGraph），包含了所有可以并发执行的任务。JobManager 会向资源管理器（ResourceManager）请求执行任务必要的资源，也就是任务管理器（TaskManager）上的插槽（slot）。一旦它获取到了足够的资源，就会将执行图分发到真正运行它们的TaskManager 上。而在运行过程中，JobManager 会负责所有需要中央协调的操作，比如说检查点（checkpoints）的协调

2. 资源管理器（ResourceManager）

   主要负责管理任务管理器（TaskManager）的插槽（slot），TaskManger 插槽是 Flink 中定义的处理资源单元。Flink 为不同的环境和资源管理工具提供了不同资源管理器，比如YARN、Mesos、K8s，以及 standalone 部署。当 JobManager 申请插槽资源时，ResourceManager会将有空闲插槽的 TaskManager 分配给 JobManager。如果 ResourceManager 没有足够的插槽来满足 JobManager 的请求，它还可以向资源提供平台发起会话，以提供启动 TaskManager进程的容器。另外，ResourceManager 还负责终止空闲的TaskManager，释放计算资源

3. 任务管理器（TaskManager）

   Flink 中的工作进程。通常在 Flink 中会有多个 TaskManager 运行，每一个 TaskManager都包含了一定数量的插槽（slots）。插槽的数量限制了 TaskManager 能够执行的任务数量。启动之后，TaskManager 会向资源管理器注册它的插槽；收到资源管理器的指令后，TaskManager 就会将一个或者多个插槽提供给 JobManager 调用。JobManager 就可以向插槽分配任务（tasks）来执行了。在执行过程中，一个 TaskManager 可以跟其它运行同一应用程序的 TaskManager 交换数据

4. 分发器（Dispatcher）

   可以跨作业运行，它为应用提交提供了 REST 接口。当一个应用被提交执行时，分发器就会启动并将应用移交给一个 JobManager。由于是 REST 接口，所以 Dispatcher 可以作为集群的一个 HTTP 接入点，这样就能够不受防火墙阻挡。Dispatcher 也会启动一个 Web UI，用来方便地展示和监控作业执行的信息。Dispatcher 在架构中可能并不是必需的，这取决于应用提交运行的方式

## 4.2 任务提交流程

我们来看看当一个应用提交执行时，Flink 的各个组件是如何交互协作的：

![image-20210119182404858](./images/27.png)

上图是从一个较为高层级的视角，来看应用中各组件的交互协作。如果部署的集群环境不同（例如 YARN，Mesos，Kubernetes，standalone 等），其中一些步骤可以被省略，或是有些组件会运行在同一个 JVM 进程中

具体地，如果我们将 Flink 集群部署到 YARN 上，那么就会有如下的提交流程：

![image-20210119182607718](./images/28.png)

Flink 任务提交后，Client 向 HDFS 上传 Flink 的 Jar 包和配置，之后向 Yarn ResourceManager 提交任务，ResourceManager 分配 Container 资源并通知对应的NodeManager 启动 ApplicationMaster，ApplicationMaster 启动后加载 Flink 的 Jar 包和配置构建环境，然后启动 JobManager，之后 ApplicationMaster 向 ResourceManager申请资源启动 TaskManager ， ResourceManager 分 配 Container 资 源 后 ， 由ApplicationMaster 通 知 资 源 所 在 节 点 的 NodeManager 启 动 TaskManager ，NodeManager 加载 Flink 的 Jar 包和配置构建环境并启动 TaskManager，TaskManager启动后向 JobManager 发送心跳包，并等待 JobManager 向其分配任务。



## 4.3 任务调度原理

![image-20210119183110013](./images/29.png)

客户端不是运行时和程序执行 的一部分，但它用于准备并发送dataflow(JobGraph)给 Master(JobManager)，然后，客户端断开连接或者维持连接以等待接收计算结果

当 Flink 集 群 启 动 后 ， 首 先 会 启 动 一 个 JobManger 和一个或多个的TaskManager。由 Client 提交任务给 JobManager，JobManager 再调度任务到各个TaskManager 去执行，然后 TaskManager 将心跳和统计信息汇报给 JobManager。TaskManager 之间以流的形式进行数据的传输。上述三者均为独立的 JVM 进程。

**Client** 为提交 Job 的客户端，可以是运行在任何机器上（与 JobManager 环境连通即可）。提交 Job 后，Client 可以结束进程（Streaming 的任务），也可以不结束并等待结果返回

**JobManager** 主 要 负 责 调 度 Job 并 协 调 Task 做 checkpoint， 职 责 上 很 像Storm 的 Nimbus。从 Client 处接收到 Job 和 JAR 包等资源后，会生成优化后的执行计划，并以 Task 的单元调度到各个 TaskManager 去执行

**TaskManager** 在启动的时候就设置好了槽位数（Slot），每个 slot 能启动一个Task，Task 为线程。从 JobManager 处接收需要部署的 Task，部署启动后，与自己的上游建立 Netty 连接，接收数据并处理



### 4.3.1 **TaskManger** **与** **Slots**

Flink 中每一个 worker(TaskManager)都是一个 **JVM** **进程**，它可能会在独立的线程上执行一个或多个 subtask。为了控制一个 worker 能接收多少个 task，worker 通 过 task slot 来进行控制（一个 worker 至少有一个 task slot）

每个 task slot 表示 TaskManager 拥有资源的**一个固定大小的子集**。假如一个TaskManager 有三个 slot，那么它会将其管理的内存分成三份给各个 slot。资源 slot化意味着一个 subtask 将不需要跟来自其他 job 的 subtask 竞争被管理的内存，取而代之的是它将拥有一定数量的内存储备。需要注意的是，这里不会涉及到 CPU 的隔离，slot 目前仅仅用来隔离 task 的受管理的内存

通过调整 task slot 的数量，允许用户定义 subtask 之间如何互相隔离。如果一个TaskManager 一个 slot，那将意味着每个 task group 运行在独立的 JVM 中（该 JVM可能是通过一个特定的容器启动的），而一个 TaskManager 多个 slot 意味着更多的subtask 可以共享同一个 JVM。而在同一个 JVM 进程中的 task 将共享 TCP 连接（基于多路复用）和心跳消息。它们也可能共享数据集和数据结构，因此这减少了每个task 的负载

![image-20210119183747329](./images/30.png)

默认情况下，Flink 允许子任务共享 slot，即使它们是不同任务的子任务（前提是它们来自同一个 job）。 这样的结果是，一个 slot 可以保存作业的整个管道

**Task Slot** **是静态的概念，是指** **TaskManager** **具有的并发执行能力**，可以通过参数 taskmanager.numberOfTaskSlots 进行配置；而**并行度** **parallelism** 是动态概念，即TaskManager运行程序时实际使用的并发能力，可以通过参数 parallelism.default进行配置

也就是说，假设一共有 3 个 TaskManager，每一个 TaskManager 中的分配 3 个TaskSlot，也就是每个 TaskManager 可以接收 3 个 task，一共 9 个 TaskSlot，如果我们设置 parallelism.default=1，即运行程序默认的并行度为 1，9 个 TaskSlot 只用了 1个，有 8 个空闲，因此，设置合适的并行度才能提高效率

![image-20210119184056225](./images/31.png)

![image-20210119184118040](./images/32.png)



### 4.3 2 程序与数据流（DataFlow）

![image-20210119184246256](./images/33.png)

所有的 Flink 程序都是由三部分组成的： **Source** 、**Transformation** 和 **Sink**。

Source 负责读取数据源，Transformation 利用各种算子进行处理加工，Sink 负责输出。

在运行时，Flink 上运行的程序会被映射成“逻辑数据流”（dataflows），它包含了这三部分。**每一个** **dataflow** **以一个或多个** **sources** **开始以一个或多个** **sinks** **结**束。dataflow 类似于任意的有向无环图（DAG）。在大部分情况下，程序中的转换运算（transformations）跟 dataflow 中的算子（operator）是一一对应的关系，但有时候，一个transformation 可能对应多个 operator。

![image-20210119184430030](./images/34.png)

### 4.3.3 执行图（ExecutionGraph）

​	由 Flink 程序直接映射成的数据流图是 StreamGraph，也被称为逻辑流图，因为它们表示的是计算逻辑的高级视图。为了执行一个流处理程序，Flink 需要将逻辑流图转换为物理数据流图（也叫执行图），详细说明程序的执行方式。

Flink 中的执行图可以分成四层：StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图

**StreamGraph**：是根据用户通过 Stream API 编写的代码生成的最初的图。用来表示程序的拓扑结构

**JobGraph**：StreamGraph 经过优化后生成了 JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件的节点 chain 在一起作为一个节点，这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗。

**ExecutionGraph** ： JobManager 根 据 JobGraph 生 成 ExecutionGraph 。ExecutionGraph 是 JobGraph 的并行化版本，是调度层最核心的数据结构。

**物理执行图**：JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。

![image-20210119184807748](./images/35.png)

### 4.3.4 并行度（Parallelism）

Flink 程序的执行具有**并行、分布式**的特性.

在执行过程中，一个流（stream）包含一个或多个分区（stream partition），而每一个算子（operator）可以包含一个或多个子任务（operator subtask），这些子任务在不同的线程、不同的物理机或不同的容器中彼此互不依赖地执行

一个特定算子的子任务（subtask）的个数被称之为其并行度（parallelism）

一般情况下，一个流程序的并行度，可以认为就是其所有算子中最大的并行度。一个程序中，不同的算子可能具有不同的并行度。

![image-20210119185153347](./images/36.png)

Stream 在算子之间传输数据的形式可以是 one-to-one(forwarding)的模式也可以是 redistributing 的模式，具体是哪一种形式，取决于算子的种类

**One-to-one**：stream(比如在 source 和 map operator 之间)维护着分区以及元素的顺序。那意味着 map 算子的子任务看到的元素的个数以及顺序跟 source 算子的子任务生产的元素的个数、顺序相同，map、fliter、flatMap 等算子都是 one-to-one 的对应关系。类似于 spark 中的**窄依赖**

**Redistributing**：stream(map()跟 keyBy/window 之间或者 keyBy/window 跟 sink之间)的分区会发生改变。每一个算子的子任务依据所选择的 transformation 发送数据到不同的目标任务。例如，keyBy() 基于 hashCode 重分区、broadcast 和 rebalance会随机重新分区，这些算子都会引起 redistribute 过程，而 redistribute 过程就类似于Spark 中的 shuffle 过程。类似于 spark 中的**宽依赖**

### 4.3.5 任务链（Operator Chains）

**相同并行度的** **one to one** **操作**，Flink 这样相连的算子链接在一起形成一个 task，原来的算子成为里面的一部分。将算子链接成 task 是非常有效的优化：它能减少线程之间的切换和基于缓存区的数据交换，在减少时延的同时提升吞吐量。链接的行为可以在编程 API 中进行指定

![image-20210120093931944](./images/37.PNG)



# 5. Flink 流处理 API

![image-20210121093950764](./images/38.png)

## 5.1 Environment

### 5.1.1.**getExecutionEnvironment**

创建一个执行环境，表示当前执行程序的上下文。 如果程序是独立调用的，则此方法返回本地执行环境；如果从命令行客户端调用程序以提交到集群，则此方法返回此集群的执行环境，也就是说，getExecutionEnvironment 会根据查询运行的方式决定返回什么样的运行环境，是最常用的一种创建执行环境的方式

```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

如果没有设置并行度，会以 flink-conf.yaml 中的配置为准，默认是 1

![image-20210121102040858](./images/39.png)

### 5.1.2 **createLocalEnvironment**

返回本地执行环境，需要在调用时指定默认的并行度。

```java
LocalStreamEnvironment env = StreamExecutionEnvironment.createLocalEnvironment(1);
```

### 5.1.3 **createRemoteEnvironment**

返回集群执行环境，将 Jar 提交到远程服务器。需要在调用时指定 JobManager的 IP 和端口号，并指定要在集群中运行的 Jar 包

```java
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.createRemoteEnvironment("jobmanage-hostname", 6123,"YOURPATH//WordCount.jar");
```

## 5.2 Source

### 5.2.1 **从集合读取数据**

```java
public class SourceTest1_Collection {
public static void main(String[] args) throws Exception{
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
// 1.Source:从集合读取数据
DataStream<SensorReading> sensorDataStream = env.fromCollection(
Arrays.asList(
new SensorReading("sensor_1", 1547718199L, 35.8),
new SensorReading("sensor_6", 1547718201L, 15.4),
new SensorReading("sensor_7", 1547718202L, 6.7),
new SensorReading("sensor_10", 1547718205L, 38.1) )
);
// 2.打印
sensorDataStream.print();
// 3.执行
env.execute();
} }
```

### 5.2.2 **从文件读取数据**

```java
DataStream<String> dataStream = env.readTextFile("YOUR_FILE_PATH ");
```

### 5.2.3 **以** **kafka** **消息队列的数据作为来源**

需要引入 kafka 连接器的依赖：

*pom.xml*

```xml
<!--
https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka-0.11 
-->
<dependency> 
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-kafka-0.11_2.12</artifactId> 
  <version>1.10.1</version>
</dependency>
```

具体代码如下：

```java
// kafka 配置项
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");
properties.setProperty("group.id", "consumer-group");
properties.setProperty("key.deserializer", 
"org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("value.deserializer", 
"org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("auto.offset.reset", "latest");
// 从 kafka 读取数据
DataStream<String> dataStream = env.addSource( new 
FlinkKafkaConsumer011<String>("sensor", new SimpleStringSchema(), properties));
```

### 5.2.4 **自定义** **Source**

除了以上的 source 数据来源，我们还可以自定义 source。需要做的，只是传入一个 SourceFunction 就可以。具体调用如下

```java
DataStream<SensorReading> dataStream = env.addSource( new MySensor());
```

我们希望可以随机生成传感器数据，MySensorSource 具体的代码实现如下：

```java
public static class MySensor implements SourceFunction<SensorReading>{
private boolean running = true;
public void run(SourceContext<SensorReading> ctx) throws Exception {
Random random = new Random();
HashMap<String, Double> sensorTempMap = new HashMap<String, Double>();
for( int i = 0; i < 10; i++ ){
sensorTempMap.put("sensor_" + (i + 1), 60 + random.nextGaussian() * 20);
}
while (running) {
  for( String sensorId: sensorTempMap.keySet() ){
Double newTemp = sensorTempMap.get(sensorId) + random.nextGaussian();
sensorTempMap.put(sensorId, newTemp);
ctx.collect( new SensorReading(sensorId, System.currentTimeMillis(), 
newTemp));
}
Thread.sleep(1000L);
} }
public void cancel() {
this.running = false; } }
```

## 5.3 Transform

### 5.3.1 **map**

![image-20210121103239708](./images/40.png)

```java
DataStream<Integer> mapStram = dataStream.map(new MapFunction<String, Integer>() {
public Integer map(String value) throws Exception {
return value.length();
}
});
```

### 5.3.2 **flatMap**

```java
DataStream<String> flatMapStream = dataStream.flatMap(new FlatMapFunction<String, 
String>() {
public void flatMap(String value, Collector<String> out) throws Exception {
String[] fields = value.split(","); 
for( String field: fields )
out.collect(field);
  }
});
```

### 5.3.3  **Filter**

![image-20210121104629729](./images/41.png)

```java
DataStream<Interger> filterStream = dataStream.filter(new FilterFunction<String>() 
{
public boolean filter(String value) throws Exception {
return value == 1; }
});
```

### 5.3.4  **KeyBy**

![image-20210121104708159](./images/42.png)

**DataStream** **→** **KeyedStream**：逻辑地将一个流拆分成不相交的分区，每个分区包含具有相同 key 的元素，在内部以 hash 的形式实现的

### 5.3.5 滚动聚合算子（Rolling Aggregation）

这些算子可以针对 KeyedStream 的每一个支流做聚合

+ sum()
+ min()
+ max()
+ minBy()
+ maxBy()

### 5.3.6 **Reduce**

**KeyedStream** **→** **DataStream**：一个分组数据流的聚合操作，合并当前的元素和上次聚合的结果，产生一个新的值，返回的流中包含每一次聚合的结果，而不是只返回最后一次聚合的最终结果

```java
DataStream<String> inputStream = env.readTextFile("sensor.txt");
// 转换成 SensorReading 类型
DataStream<SensorReading> dataStream = inputStream.map(new 
MapFunction<String, SensorReading>() {
public SensorReading map(String value) throws Exception {
String[] fileds = value.split(",");
return new SensorReading(fileds[0], new Long(fileds[1]), new 
Double(fileds[2]));
}
});
// 分组
KeyedStream<SensorReading, Tuple> keyedStream = dataStream.keyBy("id");
// reduce 聚合，取最小的温度值，并输出当前的时间戳
DataStream<SensorReading> reduceStream = keyedStream.reduce(new 
ReduceFunction<SensorReading>() {
@Override
public SensorReading reduce(SensorReading value1, SensorReading value2) 
throws Exception {
return new SensorReading(
value1.getId(),
value2.getTimestamp(),
Math.min(value1.getTemperature(), value2.getTemperature()));
}
});
```

### 5.3.7 **Split** **和** **Select**

**Split**

![image-20210121105000895](./images/43.png)

**DataStream** **→** **SplitStream**：根据某些特征把一个 DataStream 拆分成两个或者多个 DataStream

**Select**

![image-20210121105043100](./images/44.png)

SplitStream→DataStream：从一个 SplitStream 中获取一个或者多个DataStream。

需求：传感器数据按照温度高低（以 30 度为界），拆分成两个流。

```java
SplitStream<SensorReading> splitStream = dataStream.split(new 
OutputSelector<SensorReading>() {
@Override
public Iterable<String> select(SensorReading value) {
return (value.getTemperature() > 30) ? Collections.singletonList("high") : 
Collections.singletonList("low");
}
});
DataStream<SensorReading> highTempStream = splitStream.select("high");
DataStream<SensorReading> lowTempStream = splitStream.select("low");
DataStream<SensorReading> allTempStream = splitStream.select("high", "low");
```

### 5.3.8  **Connect** **和** **CoMap**

![image-20210121105227029](./images/45.png)

**DataStream,DataStream** **→** **ConnectedStreams**：连接两个保持他们类型的数据流，两个数据流被 Connect 之后，只是被放在了一个同一个流中，内部依然保持各自的数据和形式不发生任何变化，两个流相互独立。

**CoMap,CoFlatMap**

![image-20210121105336419](./images/46.png)

**ConnectedStreams → DataStream**：作用于 ConnectedStreams 上，功能与 map和 flatMap 一样，对 ConnectedStreams 中的每一个 Stream 分别进行 map 和 flatMap处理。

```java
// 合流 connect
DataStream<Tuple2<String, Double>> warningStream = highTempStream.map(new 
MapFunction<SensorReading, Tuple2<String, Double>>() {
@Override
  public Tuple2<String, Double> map(SensorReading value) throws Exception {
return new Tuple2<>(value.getId(), value.getTemperature());
}
});
ConnectedStreams<Tuple2<String, Double>, SensorReading> connectedStreams = 
warningStream.connect(lowTempStream);
DataStream<Object> resultStream = connectedStreams.map(new 
CoMapFunction<Tuple2<String,Double>, SensorReading, Object>() {
@Override
public Object map1(Tuple2<String, Double> value) throws Exception {
return new Tuple3<>(value.f0, value.f1, "warning");
}
@Override
public Object map2(SensorReading value) throws Exception {
return new Tuple2<>(value.getId(), "healthy");
}
});
```

### 5.3.9 **Union**

![image-20210121105444919](./images/47.png)

**DataStream** **→** **DataStream**：对两个或者两个以上的 DataStream 进行 union 操作，产生一个包含所有 DataStream 元素的新 DataStream。

```java
DataStream<SensorReading> unionStream = highTempStream.union(lowTempStream);
```

**Connect** **与** **Union** **区别：**

1.  Union 之前两个流的类型必须是一样，Connect 可以不一样，在之后的 coMap中再去调整成为一样的

2.  Connect 只能操作两个流，Union 可以操作多个

   

## 5.4 支持的数据类型

​		Flink 流应用程序处理的是以数据对象表示的事件流。所以在 Flink 内部，我们需要能够处理这些对象。它们需要被序列化和反序列化，以便通过网络传送它们；或者从状态后端、检查点和保存点读取它们。为了有效地做到这一点，Flink 需要明确知道应用程序所处理的数据类型。Flink 使用类型信息的概念来表示数据类型，并为每个数据类型生成特定的序列化器、反序列化器和比较器。

​		Flink 还具有一个类型提取系统，该系统分析函数的输入和返回类型，以自动获取类型信息，从而获得序列化器和反序列化器。但是，在某些情况下，例如 lambda函数或泛型类型，需要显式地提供类型信息，才能使应用程序正常工作或提高其性能。

Flink 支持 Java 和 Scala 中所有常见数据类型。使用最广泛的类型有以下几种   

### 5.4.1 **基础数据类型**

Flink 支持所有的 Java 和 Scala 基础数据类型，Int, Double, Long, String, …

```java
DataStream<Integer> numberStream = env.fromElements(1, 2, 3, 4);
numberStream.map(data -> data * 2);
```

### 5.4.2 **Java** **和** **Scala** 元组（Tuples）

```java
DataStream<Tuple2<String, Integer>> personStream = env.fromElements(
new Tuple2("Adam", 17),
new Tuple2("Sarah", 23) );
personStream.filter(p -> p.f1 > 18);
```

### 5.4.3 **Scala** 样例类（case classes）

```java
case class Person(name: String, age: Int)
val persons: DataStream[Person] = env.fromElements(
Person("Adam", 17),
Person("Sarah", 23) )
  persons.filter(p => p.age > 18)
```

### 5.4.4 **Java** 简单对象（POJOs）

```java
public class Person {
public String name;
public int age;
public Person() {}
public Person(String name, int age) { 
this.name = name; 
this.age = age; 
} }
DataStream<Person> persons = env.fromElements( 
new Person("Alex", 42), 
new Person("Wendy", 23));
```

### 5.4.5 其它（Arrays, Lists, Maps, Enums,等等）

Flink 对 Java 和 Scala 中的一些特殊目的的类型也都是支持的，比如 Java 的ArrayList，HashMap，Enum 等等

## 5.5 实现 UDF 函数——更细粒度的控制流

### 5.5.1 函数类（Function Classes）

Flink 暴露了所有 udf 函数的接口(实现方式为接口或者抽象类)。例如MapFunction, FilterFunction, ProcessFunction 等

下面例子实现了 FilterFunction 接口：

```java
DataStream<String> flinkTweets = tweets.filter(new FlinkFilter());
public static class FlinkFilter implements FilterFunction<String> {
@Override
public boolean filter(String value) throws Exception {
return value.contains("flink");
  } 
}
```

还可以将函数实现成匿名类

```java
DataStream<String> flinkTweets = tweets.filter(new FilterFunction<String>() {
@Override
public boolean filter(String value) throws Exception {
return value.contains("flink");
}
});
```

我们 filter 的字符串"flink"还可以当作参数传进去。

```java
DataStream<String> tweets = env.readTextFile("INPUT_FILE ");
DataStream<String> flinkTweets = tweets.filter(new KeyWordFilter("flink"));
public static class KeyWordFilter implements FilterFunction<String> {
private String keyWord;
KeyWordFilter(String keyWord) { this.keyWord = keyWord; }
@Override
public boolean filter(String value) throws Exception {
return value.contains(this.keyWord);
} }
```

### 5.5.2 匿名函数（Lambda Functions）

```java
DataStream<String> tweets = env.readTextFile("INPUT_FILE");
DataStream<String> flinkTweets = tweets.filter( tweet -> tweet.contains("flink") );
```

### 5.5.3 富函数（Rich Functions）

“富函数”是 DataStream API 提供的一个函数类的接口，所有 Flink 函数类都有其 Rich 版本。它与常规函数的不同在于，可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能

+ RichMapFunction
+ RichFlatMapFunction
+ RichFilterFunction
+ ...

Rich Function 有一个生命周期的概念。典型的生命周期方法有：

+ open()方法是 rich function 的初始化方法，当一个算子例如 map 或者 filter被调用之前 open()会被调用。
+ close()方法是生命周期中的最后一个调用的方法，做一些清理工作
+ getRuntimeContext()方法提供了函数的 RuntimeContext 的一些信息，例如函数执行的并行度，任务的名字，以及 state 状态

```java
public static class MyMapFunction extends RichMapFunction<SensorReading, 
Tuple2<Integer, String>> {
@Override
public Tuple2<Integer, String> map(SensorReading value) throws Exception {
return new Tuple2<>(getRuntimeContext().getIndexOfThisSubtask(), 
value.getId());
}
@Override
public void open(Configuration parameters) throws Exception {
System.out.println("my map open");
// 以下可以做一些初始化工作，例如建立一个和 HDFS 的连接
}
@Override
public void close() throws Exception {
System.out.println("my map close");
// 以下做一些清理工作，例如断开和 HDFS 的连接
} }
```

## 5.6 Sink

Flink 没有类似于 spark 中 foreach 方法，让用户进行迭代的操作。虽有对外的输出操作都要利用 Sink 完成。最后通过类似如下方式完成整个任务最终输出操作。

```java
stream.addSink(new MySink(xxxx))
```

官方提供了一部分的框架的 sink。除此以外，需要用户自定义实现 sink

![image-20210121111240207](./images/48.png)

![image-20210121111302147](./images/49.png)

### 5.6.1  **Kafka**

pom.xml

```xml
<!--
https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka-0.11 
--><dependency> 
  <groupId>org.apache.flink</groupId> 
  <artifactId>flink-connector-kafka-0.11_2.12</artifactId>
  <version>1.10.1</version>
</dependency>

```

```java
dataStream.addSink(new FlinkKafkaProducer011[String]("localhost:9092", 
"test", new SimpleStringSchema()))
```

### 5.6.2 **Redis**

pom.xml

```xml
<!-- https://mvnrepository.com/artifact/org.apache.bahir/flink-connector-redis 
--><dependency> 
  <groupId>org.apache.bahir</groupId> 
  <artifactId>flink-connector-redis_2.11</artifactId> 
  <version>1.0</version>
</dependency>
```

定义一个 redis 的 mapper 类，用于定义保存到 redis 时调用的命令：

```java
public static class MyRedisMapper implements RedisMapper<SensorReading>{
// 保存到 redis 的命令，存成哈希表
public RedisCommandDescription getCommandDescription() {
return new RedisCommandDescription(RedisCommand.HSET, "sensor_tempe");
}
public String getKeyFromData(SensorReading data) {
return data.getId();
}
public String getValueFromData(SensorReading data) {
return data.getTemperature().toString();
} }
```

在主函数中调用

```java
FlinkJedisPoolConfig config = new FlinkJedisPoolConfig.Builder()
.setHost("localhost")
.setPort(6379)
.build();
dataStream.addSink( new RedisSink<SensorReading>(config, new MyRedisMapper()) );
```

### 5.6.3 **Elasticsearch** 

pom.xml

```xml
<dependency>
  <groupId>org.apache.flink</groupId> 
  <artifactId>flink-connector-elasticsearch6_2.12</artifactId>
  <version>1.10.1</version>
</dependency>
```

在主函数中调用

```java
// es 的 httpHosts 配置
ArrayList<HttpHost> httpHosts = new ArrayList<>();
httpHosts.add(new HttpHost("localhost", 9200));
dataStream.addSink( new ElasticsearchSink.Builder<SensorReading>(httpHosts, new 
MyEsSinkFunction()).build());
```

ElasitcsearchSinkFunction 的实现

```java
public static class MyEsSinkFunction implements 
ElasticsearchSinkFunction<SensorReading>{
@Override
public void process(SensorReading element, RuntimeContext ctx, RequestIndexer 
indexer) {
HashMap<String, String> dataSource = new HashMap<>();
dataSource.put("id", element.getId());
dataSource.put("ts", element.getTimestamp().toString());
dataSource.put("temp", element.getTemperature().toString());
IndexRequest indexRequest = Requests.indexRequest()
.index("sensor")
.type("readingData")
.source(dataSource);
indexer.add(indexRequest);
} }
```

### 5.6.4 **JDBC** **自定义** **sink**

```xml
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java --> <dependency> <groupId>mysql</groupId> 
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.44</version>
</dependency>
```

添加 MyJdbcSink

```java
public static class MyJdbcSink extends RichSinkFunction<SensorReading> {
Connection conn = null;
PreparedStatement insertStmt = null;
PreparedStatement updateStmt = null;
// open 主要是创建连接
@Override
public void open(Configuration parameters) throws Exception {
conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", 
"root", "123456");
// 创建预编译器，有占位符，可传入参数
insertStmt = conn.prepareStatement("INSERT INTO sensor_temp (id, temp) VALUES 
(?, ?)");
updateStmt = conn.prepareStatement("UPDATE sensor_temp SET temp = ? WHERE id 
= ?");
}
// 调用连接，执行 sql
@Override
public void invoke(SensorReading value, Context context) throws Exception {
// 执行更新语句，注意不要留 super
updateStmt.setDouble(1, value.getTemperature());
updateStmt.setString(2, value.getId());
updateStmt.execute();
// 如果刚才 update 语句没有更新，那么插入
if (updateStmt.getUpdateCount() == 0) {
insertStmt.setString(1, value.getId());
  insertStmt.setDouble(2, value.getTemperature());
insertStmt.execute();
} }
@Override
public void close() throws Exception {
insertStmt.close();
updateStmt.close();
conn.close();
} }
```

在 main 方法中增加，把明细保存到 mysql 中

```java
dataStream.addSink(new MyJdbcSink())
```

https://www.bilibili.com/video/BV1qy4y1q728?p=46





# 6. Flink 中的 Window

## 6.1 Window

### 6.1.1 **Window** **概述**

​		streaming 流式计算是一种被设计用于处理无限数据集的数据处理引擎，而无限数据集是指一种不断增长的本质上无限的数据集，而 window 是一种切割无限数据为有限块进行处理的手段。

​		Window 是无限数据流处理的核心，Window 将一个无限的 stream 拆分成有限大小的”buckets”桶，我们可以在这些桶上做计算操作

### 6.1.2 **Window** **类型**

Window 可以分成两类

+ CountWindow：按照指定的数据条数生成一个 Window，与时间无关
+ TimeWindow：按照时间生成 Window

对于 TimeWindow，可以根据窗口实现原理的不同分成三类：滚动窗口（Tumbling Window）、滑动窗口（Sliding Window）和会话窗口（Session Window）

1. 滚动窗口（Tumbling Windows）

   将数据依据固定的窗口长度对数据进行切片

   **特点：时间对齐，窗口长度固定，没有重叠**

   滚动窗口分配器将每个元素分配到一个指定窗口大小的窗口中，滚动窗口有一个固定的大小，并且不会出现重叠。例如：如果你指定了一个 5 分钟大小的滚动窗口，窗口的创建如下图所示

   ![image-20210121195127731](./images/50.png)

   适用场景：适合做 BI 统计等（做每个时间段的聚合计算）

2. 滑动窗口（Sliding Windows）

   滑动窗口是固定窗口的更广义的一种形式，滑动窗口由固定的窗口长度和滑动间隔组成。

   **特点：时间对齐，窗口长度固定，可以有重叠。**

   滑动窗口分配器将元素分配到固定长度的窗口中，与滚动窗口类似，窗口的大小由窗口大小参数来配置，另一个窗口滑动参数控制滑动窗口开始的频率。因此，滑动窗口如果滑动参数小于窗口大小的话，窗口是可以重叠的，在这种情况下元素会被分配到多个窗口中

   例如，你有 10 分钟的窗口和 5 分钟的滑动，那么每个窗口中 5 分钟的窗口里包含着上个 10 分钟产生的数据，如下图所示：

   ![image-20210121195422012](./images/51.png)

   适用场景：对最近一个时间段内的统计（求某接口最近 5min 的失败率来决定是否要报警）。

3. 会话窗口（Session Windows）

   由一系列事件组合一个指定时间长度的 timeout 间隙组成，类似于 web 应用的session，也就是一段时间没有接收到新数据就会生成新的窗口。

   **特点：时间无对齐。**

   session 窗口分配器通过 session 活动来对元素进行分组，session 窗口跟滚动窗口和滑动窗口相比，不会有重叠和固定的开始时间和结束时间的情况，相反，当它在一个固定的时间周期内不再收到元素，即非活动间隔产生，那个这个窗口就会关闭。一个 session 窗口通过一个 session 间隔来配置，这个 session 间隔定义了非活跃

   周期的长度，当这个非活跃周期产生，那么当前的 session 将关闭并且后续的元素将被分配到新的 session 窗口中去。

   ![image-20210121195628853](./images/52.png)

## 6.2 Window API

### 6.2.1 **TimeWindow**

TimeWindow 是将指定时间范围内的所有数据组成一个 window，一次对一个window 里面的所有数据进行计算

1. 滚动窗口

   Flink 默认的时间窗口根据 Processing Time 进行窗口的划分，将 Flink 获取到的数据根据进入 Flink 的时间划分到不同的窗口中

   ```java
   DataStream<Tuple2<String, Double>> minTempPerWindowStream = dataStream
   .map(new MapFunction<SensorReading, Tuple2<String, Double>>() {
   @Override
   public Tuple2<String, Double> map(SensorReading value) throws 
   Exception {
   return new Tuple2<>(value.getId(), value.getTemperature());
   }
   })
   .keyBy(data -> data.f0) 
   .timeWindow( Time.seconds(15) )
   .minBy(1);
   ```

   时间间隔可以通过 Time.milliseconds(x)，Time.seconds(x)，Time.minutes(x)等其中的一个来指定

2. 滑动窗口（SlidingEventTimeWindows）

   滑动窗口和滚动窗口的函数名是完全一致的，只是在传参数时需要传入两个参数，一个是 window_size，一个是 sliding_size

   下面代码中的 sliding_size 设置为了 5s，也就是说，每 5s 就计算输出结果一次，每一次计算的 window 范围是 15s 内的所有元素。

   ```java
   DataStream<SensorReading> minTempPerWindowStream = dataStream
   .keyBy(SensorReading::getId) 
   .timeWindow( Time.seconds(15), Time.seconds(5) )
   .minBy("temperature");
   ```

   时间间隔可以通过 Time.milliseconds(x)，Time.seconds(x)，Time.minutes(x)等其中的一个来指定



### 6.2.2 **CountWindow**

CountWindow 根据窗口中相同 key 元素的数量来触发执行，执行时只计算元素数量达到窗口大小的 key 对应的结果。

注意：CountWindow 的 window_size 指的是相同 Key 的元素的个数，不是输入的所有元素的总数

1. 滚动窗口

   默认的 CountWindow 是一个滚动窗口，只需要指定窗口大小即可，当元素数量达到窗口大小时，就会触发窗口的执行

   ```java
   DataStream<SensorReading> minTempPerWindowStream = dataStream
   .keyBy(SensorReading::getId)
   .countWindow( 5 )
   .minBy("temperature");
   ```

2. 滑动窗口

   滑动窗口和滚动窗口的函数名是完全一致的，只是在传参数时需要传入两个参数，一个是 window_size，一个是 sliding_size。

   下面代码中的 sliding_size 设置为了 2，也就是说，每收到两个相同 key 的数据就计算一次，每一次计算的 window 范围是 10 个元素

   ```java
   DataStream<SensorReading> minTempPerWindowStream = dataStream
   .keyBy(SensorReading::getId)
   .countWindow( 10, 2 )
   .minBy("temperature");
   ```

   ### 6.2.3 **window function**

   window function 定义了要对窗口中收集的数据做的计算操作，主要可以分为两类：

   + 增量聚合函数（incremental aggregation functions）

     每条数据到来就进行计算，保持一个简单的状态。典型的增量聚合函数有ReduceFunction, AggregateFunction

   + 全窗口函数（full window functions）

     先把窗口所有数据收集起来，等到计算的时候会遍历所有数据。ProcessWindowFunction 就是一个全窗口函数。

   ### 6.2.4 **其它可选** **API**

   + .trigger() —— 触发器

     定义 window 什么时候关闭，触发计算并输出结果

   + .evitor() —— 移除器

     定义移除某些数据的逻辑

   + .allowedLateness() —— 允许处理迟到的数据

   + .sideOutputLateData() —— 将迟到的数据放入侧输出流

   + .getSideOutput() —— 获取侧输出流

   ![image-20210122101949929](./images/53.png)

# 7. 时间语义与 Wartermark

## 7.1  Flink 中的时间语义

在 Flink 的流式处理中，会涉及到时间的不同概念，如下图所示：

![image-20210123094350646](./images/54.png)

**Event Time**：是事件创建的时间。它通常由事件中的时间戳描述，例如采集的日志数据中，每一条日志都会记录自己的生成时间，Flink 通过时间戳分配器访问事件时间戳。

**Ingestion Time**：是数据进入 Flink 的时间。

**Processing Time**：是每一个执行基于时间操作的算子的本地系统时间，与机器相关，默认的时间属性就是 Processing Time

一个例子——电影《星球大战》：

![image-20210123094608739](./images/55.png)

例如，一条日志进入 Flink 的时间为 2017-11-12 10:00:00.123，到达 Window 的系统时间为 2017-11-12 10:00:01.234，日志的内容如下：

```bash
2017-11-02 18:37:15.624 INFO Fail over to rm2
```

对于业务来说，要统计 1min 内的故障日志个数，哪个时间是最有意义的？——eventTime，因为我们要根据日志的生成时间进行统计。

## 7.2 EventTime 的引入

**在** **Flink** **的流式处理中，绝大部分的业务都会使用** **eventTime**，一般只在eventTime 无法使用时，才会被迫使用 ProcessingTime 或者 IngestionTime

如果要使用 EventTime，那么需要引入 EventTime 的时间属性，引入方式如下所示：

```java
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment
// 从调用时刻开始给 env 创建的每一个 stream 追加时间特征
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```

## 7.3 Watermark

### 7.3.1 **基本概念**

我们知道，流处理从事件产生，到流经 source，再到 operator，中间是有一个过程和时间的，虽然大部分情况下，流到 operator 的数据都是按照事件产生的时间顺序来的，但是也不排除由于网络、分布式等原因，导致乱序的产生，所谓乱序，就是指 Flink 接收到的事件的先后顺序不是严格按照事件的 Event Time 顺序排列的。

![image-20210123095118900](./images/56.png)

那么此时出现一个问题，一旦出现乱序，如果只根据 eventTime 决定 window 的运行，我们不能明确数据是否全部到位，但又不能无限期的等下去，此时必须要有个机制来保证一个特定的时间后，必须触发 window 去进行计算了，这个特别的机制，就是 Watermark。

+ Watermark 是一种衡量 Event Time 进展的机制。
+ **Watermark** **是用于处理乱序事件的**，而正确的处理乱序事件，通常用Watermark 机制结合 window 来实现。
+ 数据流中的 Watermark 用于表示 timestamp 小于 Watermark 的数据，都已经到达了，因此，window 的执行也是由 Watermark 触发的。
+ Watermark 可以理解成一个延迟触发机制，我们可以设置 Watermark 的延时时长 t，每次系统会校验已经到达的数据中最大的 maxEventTime，然后认定 eventTime小于 maxEventTime - t 的所有数据都已经到达，如果有窗口的停止时间等于maxEventTime – t，那么这个窗口被触发执行

有序流的 Watermarker 如下图所示：（Watermark 设置为 0）

![image-20210123095621486](./images/57.png)

乱序流的 Watermarker 如下图所示：（Watermark 设置为 2）

![image-20210123095714028](./images/58.png)

当 Flink 接收到数据时，会按照一定的规则去生成 Watermark，这条 Watermark就等于当前所有到达数据中的 maxEventTime - 延迟时长，也就是说，Watermark 是基于数据携带的时间戳生成的，一旦 Watermark 比当前未触发的窗口的停止时间要晚，那么就会触发相应窗口的执行。由于 event time 是由数据携带的，因此，如果运行过程中无法获取新的数据，那么没有被触发的窗口将永远都不被触发。

上图中，我们设置的允许最大延迟到达时间为 2s，所以时间戳为 7s 的事件对应的 Watermark 是 5s，时间戳为 12s 的事件的 Watermark 是 10s，如果我们的窗口 1 是 1s~5s，窗口 2 是 6s~10s，那么时间戳为 7s 的事件到达时的 Watermarker 恰好触发窗口 1，时间戳为 12s 的事件到达时的 Watermark 恰好触发窗口 2。

Watermark 就是触发前一窗口的“关窗时间”，一旦触发关窗那么以当前时刻为准在窗口范围内的所有所有数据都会收入窗中。

只要没有达到水位那么不管现实中的时间推进了多久都不会触发关窗。

### 7.3.2 **Watermark** **的引入**

watermark 的引入很简单，对于乱序数据，最常见的引用方式如下：

```java
dataStream.assignTimestampsAndWatermarks( new 
BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.millisecond
s(1000)) {
@Override
  public long extractTimestamp(element: SensorReading): Long = {
return element.getTimestamp() * 1000L; }
} );
```

Event Time 的使用一定要**指定数据源中的时间戳**。否则程序无法知道事件的事件时间是什么(数据源里的数据没有时间戳的话，就只能使用 Processing Time 了)。

我们看到上面的例子中创建了一个看起来有点复杂的类，这个类实现的其实就是分配时间戳的接口。Flink 暴露了 TimestampAssigner 接口供我们实现，使我们可以自定义如何从事件数据中抽取时间戳。

```java
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
// 设置事件时间语义
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
DataStream<SensorReading> dataStream = env.addSource(new SensorSource())
.assignTimestampsAndWatermarks(new MyAssigner());
```

MyAssigner 有两种类型

+ AssignerWithPeriodicWatermarks
+ AssignerWithPunctuatedWatermarks

以上两个接口都继承自 TimestampAssigner。

**Assigner with periodic watermarks**

周期性的生成 watermark：系统会周期性的将 watermark 插入到流中(水位线也是一种特殊的事件!)。默认周期是 200 毫秒。可以使用ExecutionConfig.setAutoWatermarkInterval()方法进行设置。

```java
// 每隔 5 秒产生一个 watermark
env.getConfig.setAutoWatermarkInterval(5000);
```

产生 watermark 的逻辑：每隔 5 秒钟，Flink 会调用

AssignerWithPeriodicWatermarks 的 getCurrentWatermark()方法。如果方法返回一个时间戳大于之前水位的时间戳，新的 watermark 会被插入到流中。这个检查保证了水位线是单调递增的。如果方法返回的时间戳小于等于之前水位的时间戳，则不会产生新的 watermark

例子，自定义一个周期性的时间戳抽取：

```java
// 自定义周期性时间戳分配器
public static class MyPeriodicAssigner implements 
AssignerWithPeriodicWatermarks<SensorReading>{
private Long bound = 60 * 1000L; // 延迟一分钟
private Long maxTs = Long.MIN_VALUE; // 当前最大时间戳
@Nullable
@Override
public Watermark getCurrentWatermark() {
return new Watermark(maxTs - bound);
}@Override
public long extractTimestamp(SensorReading element, long previousElementTimestamp) 
{
maxTs = Math.max(maxTs, element.getTimestamp());
return element.getTimestamp();
} }
```

一种简单的特殊情况是，如果我们事先得知数据流的时间戳是单调递增的，也就是说没有乱序，那我们可以使用 AscendingTimestampExtractor，这个类会直接使用数据的时间戳生成 watermark。

```java
DataStream<SensorReading> dataStream = …
dataStream.assignTimestampsAndWatermarks(
new AscendingTimestampExtractor<SensorReading>() {
@Override
public long extractAscendingTimestamp(SensorReading element) {
return element.getTimestamp() * 1000; }
});
```

而对于乱序数据流，如果我们能大致估算出数据流中的事件的最大延迟时间，就可以使用如下代码：

```java
DataStream<SensorReading> dataStream = …
dataStream.assignTimestampsAndWatermarks(
new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(1)) {
@Override
public long extractTimestamp(SensorReading element) {
return element.getTimestamp() * 1000L; }
});
```

**Assigner with punctuated watermarks**

间断式地生成 watermark。和周期性生成的方式不同，这种方式不是固定时间的，而是可以根据需要对每条数据进行筛选和处理。直接上代码来举个例子，我们只给sensor_1 的传感器的数据流插入 watermark：

```java
public static class MyPunctuatedAssigner implements 
AssignerWithPunctuatedWatermarks<SensorReading>{
private Long bound = 60 * 1000L; // 延迟一分钟
@Nullable
@Override
public Watermark checkAndGetNextWatermark(SensorReading lastElement, long 
extractedTimestamp) {
if(lastElement.getId().equals("sensor_1"))
return new Watermark(extractedTimestamp - bound);
else
return null; }
@Override
public long extractTimestamp(SensorReading element, long previousElementTimestamp) 
{
return element.getTimestamp();
} }
```

## 7.4  EvnetTime 在 window 中的使用（Scala 版）

### 7.4.1 	滚动窗口（TumblingEventTimeWindows）

```scala
def main(args: Array[String]): Unit = {
// 环境
val env: StreamExecutionEnvironment = 
StreamExecutionEnvironment.getExecutionEnvironment
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
env.setParallelism(1)
val dstream: DataStream[String] = env.socketTextStream("localhost",7777)
val textWithTsDstream: DataStream[(String, Long, Int)] = dstream.map 
{ text =>
val arr: Array[String] = text.split(" ")
(arr(0), arr(1).toLong, 1) }
val textWithEventTimeDstream: DataStream[(String, Long, Int)] = 
textWithTsDstream.assignTimestampsAndWatermarks(new 
BoundedOutOfOrdernessTimestampExtractor[(String, Long, 
Int)](Time.milliseconds(1000)) {
override def extractTimestamp(element: (String, Long, Int)): Long = {
return element._2
}
})
val textKeyStream: KeyedStream[(String, Long, Int), Tuple] =
  textWithEventTimeDstream.keyBy(0)
textKeyStream.print("textkey:")
val windowStream: WindowedStream[(String, Long, Int), Tuple, TimeWindow] 
= textKeyStream.window(TumblingEventTimeWindows.of(Time.seconds(2)))
val groupDstream: DataStream[mutable.HashSet[Long]] = 
windowStream.fold(new mutable.HashSet[Long]()) { case (set, (key, ts, count)) 
=>
set += ts
}
groupDstream.print("window::::").setParallelism(1)
env.execute()
} }
  
```

结果是按照 Event Time 的时间窗口计算得出的，而无关系统的时间（包括输入的快慢）。

### 7.4.2 滑动窗口（SlidingEventTimeWindows）

```scala
def main(args: Array[String]): Unit = {
// 环境
val env: StreamExecutionEnvironment = 
StreamExecutionEnvironment.getExecutionEnvironment
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
env.setParallelism(1)
  val dstream: DataStream[String] = env.socketTextStream("localhost",7777)
val textWithTsDstream: DataStream[(String, Long, Int)] = dstream.map { text 
=>
val arr: Array[String] = text.split(" ")
(arr(0), arr(1).toLong, 1) }
val textWithEventTimeDstream: DataStream[(String, Long, Int)] = 
textWithTsDstream.assignTimestampsAndWatermarks(new 
BoundedOutOfOrdernessTimestampExtractor[(String, Long, 
Int)](Time.milliseconds(1000)) {
override def extractTimestamp(element: (String, Long, Int)): Long = {
return element._2
}
})
val textKeyStream: KeyedStream[(String, Long, Int), Tuple] = 
textWithEventTimeDstream.keyBy(0)
textKeyStream.print("textkey:")
val windowStream: WindowedStream[(String, Long, Int), Tuple, TimeWindow] = 
textKeyStream.window(SlidingEventTimeWindows.of(Time.seconds(2),Time.millis
econds(500)))
val groupDstream: DataStream[mutable.HashSet[Long]] = windowStream.fold(new 
mutable.HashSet[Long]()) { case (set, (key, ts, count)) =>
set += ts
  }
groupDstream.print("window::::").setParallelism(1)
env.execute()
}
```

### 7.4.3 会话窗口（EventTimeSessionWindows）

相邻两次数据的 EventTime 的时间差超过指定的时间间隔就会触发执行。如果加入 Watermark， 会在符合窗口触发的情况下进行延迟。到达延迟水位再进行窗口触发。

```scala
def main(args: Array[String]): Unit = {
// 环境
val env: StreamExecutionEnvironment = 
StreamExecutionEnvironment.getExecutionEnvironment
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
env.setParallelism(1)
val dstream: DataStream[String] = env.socketTextStream("localhost",7777)
val textWithTsDstream: DataStream[(String, Long, Int)] = dstream.map { text 
=>
val arr: Array[String] = text.split(" ")
(arr(0), arr(1).toLong, 1) }
val textWithEventTimeDstream: DataStream[(String, Long, Int)] = 
textWithTsDstream.assignTimestampsAndWatermarks(new
                                                BoundedOutOfOrdernessTimestampExtractor[(String, Long, 
Int)](Time.milliseconds(1000)) {
override def extractTimestamp(element: (String, Long, Int)): Long = {
return element._2
}
})
val textKeyStream: KeyedStream[(String, Long, Int), Tuple] = 
textWithEventTimeDstream.keyBy(0)
textKeyStream.print("textkey:")
val windowStream: WindowedStream[(String, Long, Int), Tuple, TimeWindow] 
= 
textKeyStream.window(EventTimeSessionWindows.withGap(Time.milliseconds(500)
) )
windowStream.reduce((text1,text2)=>
( text1._1,0L,text1._3+text2._3)
) .map(_._3).print("windows:::").setParallelism(1)
env.execute()
}	
```

# 8. Flink状态管理

## 8.1 Flink中的状态

![image-20210123102719532](./images/59.png)

+ 由一个任务维护，并且用来计算某个结果的所有数据，都属于这个任务的状态
+ 可以认为状态就是一个本地变量，可以被任务的业务逻辑访问
+ Flink会进行状态管理，包括状态一致性、故障处理以及高效存储和访问，以便开发人员可以专注于应用程序的逻辑
+ 在Flink中，状态始终与特定算子关联
+ 为了使运行时的Flink了解算子的状态，算子需要预先注册其状态

总的说来，有两种类型的状态:

+ 算子状态(Operator State)

  算子状态的作用范围限定为算子任务

+ 键控状态(Keyed  State)

  根据输入数据流中定义的键(key)来维护和访问

### 8.1.1 算子状态

![image-20210123124141195](./images/60.png)

+ 算子状态的作用范围限定为算子任务，由同一个并行任务所处理的所有数据都可以访问到向相同的状态
+ 状态对于同一子任务而言是共享的
+ 算子状态不能由相同或不同算子的另一个子任务访问

### 8.1.2 算子状态数据结构

+ 列表状态(List state)

  将状态表示为一组数据的列表

+ 联合列表状态(Union list state)

  也将数据表示为数据的列表，与常见的列表状态的区别在于，在发生故障时或者从保存点(savepoint)启动应用程序时如何恢复

+ 广播状态(Broadcast state)

  如果一个算子有多项任务，而它的每项任务状态又都相同，那么这中特殊情况最适合应用广播状态

## 8.2 键控状态(Keyed State)

![image-20210123155311059](./images/61.png)

+ 键控状态是根据输入数据流中定义的键(key)来维护和访问
+ Flink为每个key维护一个状态实例，并将具有相同键的所有数据，都分区到同一个同一个算子任务中，这个任务会维护和处理这个key对应的状态
+ 当处理一条数据时，这个任务会维护和处理这个key对应的状态
+ 当任务处理一条数据时，它会自动将状态的访问范围限定为当前数据的key

## 8.3 键控状态数据结构

+ 值状态(Value state)

  将状态表示为单个的值

+ 列表状态(List state)

  将状态表示为一组数据的列表

+ 映射状态(Map state)

  将状态表示为一组key-value对

+ 聚合状态(Reducing state & Aggregating state)

  将状态表示为一个用于聚合操作的列表

## 8.4 键控状态的使用

![image-20210123160916212](./images/62.png)

## 8.4 状态后端(State Backends)

+ 每传入一条数据，有状态的算子任务都会读取和更新状态
+ 由于有效的状态访问对于处理数据的低延迟至关重要，因此每个并行任务都会在本地维护其状态，以确保快速的状态访问
+ 状态的存储、访问以及维护，由一个可插入的组件决定，这个组件就叫做状态后端(state backend)
+ 状态后端主要负责两件事:本地的状态管理以及将检查点(checkpoint)状态写入远程存储

### 8.4.1 选择一个状态

1. MemoryStateBackend

   + 内存级的状态后端，会将键控状态作为内存的对象进行管理，将它们存储在TaskManager的JVM堆上，而将checkpoint存储在JobManager的内存中
   + 特点:快速、低延迟、但不稳定

2. FsStateBackend

   + 将checkpoint存到远程的持久化文件系统(FileSystem)上，而对于本地状态，跟MemoryStateBackend一样，也会存在TaskManager的JVM堆上
   + 同时拥有内存级别的本地访问速度，和更好的容错保证

3. RocksDBStateBackend

   将所有状态序列化后，存入本地的RockDB中存储



# 9. ProcessFunction API（底层 API）

我们之前学习的**转换算子**是无法访问事件的时间戳信息和水位线信息的。而这在一些应用场景下，极为重要。例如 MapFunction 这样的 map 转换算子就无法访问时间戳或者当前事件的事件时间

基于此，DataStream API 提供了一系列的 Low-Level 转换算子。可以访问时间戳、watermark以及注册定时事件**。还可以输出**特定的一些事件，例如超时事件等。Process Function 用来构建事件驱动的应用以及实现自定义的业务逻辑(使用之前的window 函数和转换算子无法实现)。例如，Flink SQL 就是使用 Process Function 实现的。

Flink 提供了 8 个 Process Function： 

+ ProcessFunction
+ KeyedProcessFunction
+ CoProcessFunction
+ ProcessJoinFunction
+ BroadcastProcessFunction
+ KeyedBroadcastProcessFunction
+ ProcessWindowFunction
+ ProcessAllWindowFunction

## 9.1 KeyedProcessFunction

KeyedProcessFunction 用来操作 KeyedStream。KeyedProcessFunction 会处理流的每一个元素，输出为 0 个、1 个或者多个元素。所有的 Process Function 都继承自RichFunction 接口，所以都有 open()、close()和 getRuntimeContext()等方法。而KeyedProcessFunction<K, I, O>还额外提供了两个方法:

+ processElement(I value, Context ctx, Collector<O> out), 流中的每一个元素都会调用这个方法，调用结果将会放在 Collector 数据类型中输出。Context 可以访问元素的时间戳，元素的 key以及 TimerService 时间服务。Context 还可以将结果输出到别的流(side outputs)
+ onTimer(long timestamp, OnTimerContext ctx, Collector<O> out) 是一个回调函数。当之前注册的定时器触发时调用。参数 timestamp 为定时器所设定的触发的时间戳。Collector 为输出结果的集合。OnTimerContext 和processElement 的 Context 参数一样，提供了上下文的一些信息，例如定时器触发的时间信息(事件时间或者处理时间)。

## 9.2  TimerService 和 定时器（Timers）

Context 和 OnTimerContext 所持有的 TimerService 对象拥有以下方法: 

+ long currentProcessingTime() 返回当前处理时间
+ long currentWatermark() 返回当前 watermark 的时间戳
+ void registerProcessingTimeTimer(long timestamp) 会注册当前 key 的processing time 的定时器。当 processing time 到达定时时间时，触发 timer。
+ void registerEventTimeTimer(long timestamp) 会注册当前 key 的 event time 定时器。当水位线大于等于定时器注册的时间时，触发定时器执行回调函数。
+ void deleteProcessingTimeTimer(long timestamp) 删除之前注册处理时间定时器。如果没有这个时间戳的定时器，则不执行。
+ void deleteEventTimeTimer(long timestamp) 删除之前注册的事件时间定时器，如果没有此时间戳的定时器，则不执行。

当定时器 timer 触发时，会执行回调函数 onTimer()。注意定时器 timer 只能在keyed streams 上面使用

下面举个例子说明 KeyedProcessFunction 如何操作 KeyedStream

需求：监控温度传感器的温度值，如果温度值在 10 秒钟之内(processing time)连续上升，则报警

```java
DataStream<String> warningStream = dataStream.keyBy(SensorReading::getId)
.process( new TempIncreaseWarning(10) );
```

看一下 TempIncreaseWarning 如何实现, 程序中使用了 ValueState 状态变量来保存上次的温度值和定时器时间戳

```java
public static class TempIncreaseWarning extends KeyedProcessFunction<String, 
SensorReading, String>{
private Integer interval;
public TempIncreaseWarning(Integer interval) {
this.interval = interval;
}
// 声明状态，保存上次的温度值、当前定时器时间戳
private ValueState<Double> lastTempState;
private ValueState<Long> timerTsState;
@Override
public void open(Configuration parameters) throws Exception {
lastTempState = getRuntimeContext().getState(new 
ValueStateDescriptor<Double>("last-temp", Double.class, Double.MIN_VALUE));
timerTsState = getRuntimeContext().getState(new 
ValueStateDescriptor<Long>("timer-ts", Long.class));
}
@Override
public void processElement(SensorReading value, Context ctx, Collector<String> out) 
throws Exception {
// 取出状态
Double lastTemp = lastTempState.value();
Long timerTs = timerTsState.value();
// 更新温度状态
lastTempState.update(value.getTemperature());
if( value.getTemperature() > lastTemp && timerTs == null ){
long ts = ctx.timerService().currentProcessingTime() + interval * 1000L;
ctx.timerService().registerProcessingTimeTimer(ts);
timerTsState.update(ts); 
}
else if( value.getTemperature() < lastTemp && timerTs != null){
ctx.timerService().deleteProcessingTimeTimer(timerTs);
timerTsState.clear();
} }
  @Override
public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) 
throws Exception {
out.collect( "传感器" + ctx.getCurrentKey() + "的温度连续" + interval + "秒上升
" );
// 清空 timer 状态
timerTsState.clear();
} }
```

## 9.3 侧输出流（SideOutput）

大部分的 DataStream API 的算子的输出是单一输出，也就是某种数据类型的流。除了 split 算子，可以将一条流分成多条流，这些流的数据类型也都相同。process function 的 side outputs 功能可以产生多条流，并且这些流的数据类型可以不一样。一个 side output 可以定义为 OutputTag[X]对象，X 是输出流的数据类型。process function 可以通过 Context 对象发射一个事件到一个或者多个 side outputs。

下面是一个示例程序，用来监控传感器温度值，将温度值低于 30 度的数据输出到 side output

```java
final OutputTag<SensorReading> lowTempTag = new OutputTag<SensorReading>("lowTemp"){};
SingleOutputStreamOperator<SensorReading> highTempStream = dataStream.process(new 
ProcessFunction<SensorReading, SensorReading>() {
@Override
public void processElement(SensorReading value, Context ctx, 
Collector<SensorReading> out) throws Exception {
if( value.getTemperature() < 30 )
ctx.output(lowTempTag, value);
else
out.collect(value);
}
});
DataStream<SensorReading> lowTempStream = highTempStream.getSideOutput(lowTempTag);
highTempStream.print("high");
lowTempStream.print("low");
```

## 9.4  CoProcessFunction

对于两条输入流，DataStream API 提供了 CoProcessFunction 这样的 low-level操作。CoProcessFunction 提供了操作每一个输入流的方法: processElement1()和processElement2()。

类似于 ProcessFunction，这两种方法都通过 Context 对象来调用。这个 Context对象可以访问事件数据，定时器时间戳，TimerService，以及 side outputs。CoProcessFunction 也提供了 onTimer()回调函数

# 10.容错机制

## 10.1 一致性检查点（checkpoint） 

![image-20210125144153501](./images/63.png)

+ Flink 故障恢复机制的核心，就是应用状态的一致性检查点
+ 有状态流应用的一致检查点，其实就是所有任务的状态，在某个时间点的一份拷贝（一份快照）；这个时间点，应该是所有任务都恰好处理完一个相同的输入数据的时候

## 10.2 从检查点恢复状态

![image-20210125144313950](./images/64.png)

+ 在执行流应用程序期间，Flink 会定期保存状态的一致检查点
+ 如果发生故障， Flink 将会使用最近的检查点来一致恢复应用程序的状态，并重新启动处理流程

![image-20210125144441811](./images/65.png)

+ 遇到故障之后，第一步就是重启应用,如上图

![image-20210125144519266](./images/66.png)

+ 第二步是从 checkpoint 中读取状态，将状态重置
+ 从检查点重新启动应用程序后，其内部状态与检查点完成时的状态完全相同

![image-20210125144615875](./images/67.png)

+ 第三步：开始消费并处理检查点到发生故障之间的所有数据
+ 这种检查点的保存和恢复机制可以为应用程序状态提供“精确一次”（exactly-once）的一致性，因为所有算子都会保存检查点并恢复其所有状态，这样一来所有的输入流就都会被重置到检查点完成时的位置

## 10.3 Flink 检查点算法

+ 一种简单的想法

  暂停应用，保存状态到检查点，再重新恢复应用

+ Flink 的改进实现

  基于 Chandy-Lamport 算法的分布式快照

  将检查点的保存和数据处理分离开，不暂停整个应用

### 10.3.1 检查点分界线（Checkpoint Barrier） 

+ Flink 的检查点算法用到了一种称为分界线（barrier）的特殊数据形式，用来把一条流上数据按照不同的检查点分开
+ 分界线之前到来的数据导致的状态更改，都会被包含在当前分界线所属的检查点中；而基于分界线之后的数据导致的所有更改，就会被包含在之后的检查点中

![image-20210125145023777](./images/68.png)

现在是一个有两个输入流的应用程序，用并行的两个 Source 任务来读取

![image-20210125145100376](./images/69.png)

JobManager 会向每个 source 任务发送一条带有新检查点 ID 的消息，通过这种方式来启动检查点

![image-20210125145140870](./images/70.png)

+ 数据源将它们的状态写入检查点，并发出一个检查点 barrier
+ 状态后端在状态存入检查点之后，会返回通知给 source 任务，source 任务就会向 JobManager 确认检查点完成

![image-20210125145300552](./images/71.png)

+ 分界线对齐：barrier 向下游传递，sum 任务会等待所有输入分区的 barrier 到达 

+ 对于barrier已经到达的分区，继续到达的数据会被缓存

+ 而barrier尚未到达的分区，数据会被正常处理

  

![image-20210125145419773](./images/72.png)

当收到所有输入分区的 barrier 时，任务就将其状态保存到状态后端的检查点中,然后将 barrier 继续向下游转发

![image-20210125145741366](./images/73.png)

向下游转发检查点 barrier 后，任务继续正常的数据处理

![image-20210125145808560](./images/74.png)

+ Sink 任务向 JobManager 确认状态保存到 checkpoint 完毕
+ 当所有任务都确认已成功将状态保存到检查点时，检查点就真正完成了

## 10.4 保存点（save points）

+ Flink 还提供了可以自定义的镜像保存功能，就是保存点（savepoints） 
+ 原则上，创建保存点使用的算法与检查点完全相同，因此保存点可以认为就是具有一些额外元数据的检查点
+ Flink不会自动创建保存点，因此用户（或者外部调度程序）必须明确地触发创建操作
+ 保存点是一个强大的功能。除了故障恢复外，保存点可以用于：有计划的手动备份，更新应用程序，版本迁移，暂停和重启应用，等等



# 11. 状态一致性

## 11.1状态一致性

![image-20210125150602559](./images/75.png)

+ 有状态的流处理，内部每个算子任务都可以有自己的状态
+ 对于流处理器内部来说，所谓的状态一致性，其实就是我们所说的计算结果要保证准确
+ 一条数据不应该丢失，也不应该重复计算
+ 在遇到故障时可以恢复状态，恢复以后的重新计算，结果应该也是完全正确的

### 11.1.1 状态一致性分类

+ AT-MOST-ONCE（最多一次）

  当任务故障时，最简单的做法是什么都不干，既不恢复丢失的状态，也不重播丢失的数据。At-most-once 语义的含义是最多处理一次事件

+ AT-LEAST-ONCE（至少一次）

  在大多数的真实应用场景，我们希望不丢失事件。这种类型的保障称为 at least-once，意思是所有的事件都得到了处理，而一些事件还可能被处理多次。

+ EXACTLY-ONCE（精确一次）

  恰好处理一次是最严格的保证，也是最难实现的。恰好处理一次语义不仅仅意味着没有事件丢失，还意味着针对每一个数据，内部状态仅仅更新一次

## 11.2 一致性检查点（checkpoint）

+ Flink 使用了一种轻量级快照机制 —— 检查点（checkpoint）来保证 exactly-once 语义
+ 有状态流应用的一致检查点，其实就是：所有任务的状态，在某个时间点的一份拷贝（一份快照）。而这个时间点，应该是所有任务都恰好处理完一个相同的输入数据的时候
+ 应用状态的一致检查点，是 Flink 故障恢复机制的核心

![image-20210125151643600](./images/76.png)

## 11.3 端到端（end-to-end）状态一致性

+ 目前我们看到的一致性保证都是由流处理器实现的，也就是说都是在Flink 流处理器内部保证的；而在真实应用中，流处理应用除了流处理器以外还包含了数据源（例如 Kafka）和输出到持久化系统
+ 端到端的一致性保证，意味着结果的正确性贯穿了整个流处理应用的始终；每一个组件都保证了它自己的一致性
+ 整个端到端的一致性级别取决于所有组件中一致性最弱的组件

## 11.4 端到端的精确一次（exactly-once）保证

### 11.4.1 内部保证(checkpoint)

### 11.4.2 source 端(可重设数据的读取位置)



### 11.4.3 sink端(从故障恢复时，数据不会重复写入外部系统)

1. 幂等写入(Idempotent Writes)

   所谓幂等操作，是说一个操作，可以重复执行很多次，但只导致一次结果更改，也就是说，后面再重复执行就不起作用了

   ![image-20210125152056195](./images/77.png)

2. 事务写入

   + 事务（Transaction） 

     应用程序中一系列严密的操作，所有操作必须成功完成，否则在每个操作中所作的所有更改都会被撤消

     具有原子性：一个事务中的一系列的操作要么全部成功，要么一个都不做

   + 实现思想

     构建的事务对应着 checkpoint，等到 checkpoint 真正完成的时候，才把所有对应的结果写入 sink 系统中

   + 实现方式

     (1). 预写日志

     (2). 两阶段提交

3. 预写日志（Write-Ahead-Log，WAL） 

   + 把结果数据先当成状态保存，然后在收到 checkpoint 完成的通知时，一次性写入 sink 系统
   + 简单易于实现，由于数据提前在状态后端中做了缓存，所以无论什么sink 系统，都能用这种方式一批搞定
   + DataStream API 提供了一个模板类：GenericWriteAheadSink，来实现这种事务性 sink

4. 两阶段提交（Two-Phase-Commit，2PC） 

   + 对于每个 checkpoint，sink 任务会启动一个事务，并将接下来所有接收的数据添加到事务里
   + 然后将这些数据写入外部 sink 系统，但不提交它们 —— 这时只是“预提交”
   + 当它收到 checkpoint 完成的通知时，它才正式提交事务，实现结果的真正写入

   这种方式真正实现了 exactly-once，它需要一个提供事务支持的外部sink 系统。Flink 提供了 TwoPhaseCommitSinkFunction 接口。

5. 2PC 对外部 sink 系统的要求

   + 外部 sink 系统必须提供事务支持，或者 sink 任务必须能够模拟外部系统上的事务

   + 在 checkpoint 的间隔期间里，必须能够开启一个事务并接受数据写入

   + 在收到 checkpoint 完成的通知之前，事务必须是“等待提交”的状态

     在故障恢复的情况下，这可能需要一些时间。如果这个时候sink系统关闭事务（例如超时了），那么未提交的数据就会丢失

   + sink 任务必须能够在进程失败后恢复事务

   + 提交事务必须是幂等操作

6. 不同 Source 和 Sink 的一致性保证

   ![image-20210125152729999](./images/78.png)

## 11.5 Flink+Kafka 端到端状态一致性的保证

+ 内部 —— 利用 checkpoint 机制，把状态存盘，发生故障的时候可以恢复，保证内部的状态一致性
+ source —— kafka consumer 作为 source，可以将偏移量保存下来，如果后续任务出现了故障，恢复的时候可以由连接器重置偏移量，重新消费数据，保证一致性
+ sink —— kafka producer 作为sink，采用两阶段提交 sink，需要实现一个 TwoPhaseCommitSinkFunction

### 11.5.1 Exactly-once 两阶段提交

![image-20210125153049835](./images/79.png)

+ JobManager 协调各个 TaskManager 进行 checkpoint 存储
+ checkpoint保存在 StateBackend中，默认StateBackend是内存级的，也可以改为文件级的进行持久化保存

![image-20210125153234329](./images/80.png)

+ 当 checkpoint 启动时，JobManager 会将检查点分界线（barrier）注入数据流
+ barrier会在算子间传递下去

![image-20210125153326672](./images/81.png)

+ 每个算子会对当前的状态做个快照，保存到状态后端
+ checkpoint 机制可以保证内部的状态一致性

![image-20210125153415902](./images/82.png)

+ 每个内部的 transform 任务遇到 barrier 时，都会把状态存到 checkpoint 里
+ sink 任务首先把数据写入外部 kafka，这些数据都属于预提交的事务；遇到barrier 时，把状态保存到状态后端，并开启新的预提交事务

![image-20210125153500944](./images/83.png)

+ 当所有算子任务的快照完成，也就是这次的 checkpoint 完成时，JobManager 会向所有任务发通知，确认这次 checkpoint 完成
+ sink 任务收到确认通知，正式提交之前的事务，kafka 中未确认数据改为“已确认”

### 11.5.2 Exactly-once 两阶段提交步骤

+ 第一条数据来了之后，开启一个 kafka 的事务（transaction），正常写入 kafka 分区日志但标记为未提交，这就是“预提交”
+ jobmanager 触发 checkpoint 操作，barrier 从 source 开始向下传递，遇到barrier 的算子将状态存入状态后端，并通知 jobmanager
+ sink 连接器收到 barrier，保存当前状态，存入 checkpoint，通知 jobmanager，并开启下一阶段的事务，用于提交下个检查点的数据
+ jobmanager 收到所有任务的通知，发出确认信息，表示 checkpoint 完成
+ sink 任务收到 jobmanager 的确认信息，正式提交这段时间的数据
+ 外部kafka关闭事务，提交的数据可以正常消费了



# 12. Table API 和 Flink SQL

## 12.1 Table API 和 Flink SQL 是什么

+ Flink 对批处理和流处理，提供了统一的上层 API
+ Table API 是一套内嵌在 Java 和 Scala 语言中的查询API，它允许以非常直观的方式组合来自一些关系运算符的查询
+ Flink 的 SQL 支持基于实现了 SQL 标准的 Apache Calcite

![image-20210125154627657](./images/84.png)

## 12.2 基本程序结构

Table API 和 SQL 的程序结构，与流式处理的程序结构十分类似

```java
StreamTableEnvironment tableEnv = ... // 创建表的执行环境
// 创建一张表，用于读取数据
tableEnv.connect(...).createTemporaryTable("inputTable");
// 注册一张表，用于把计算结果输出
tableEnv.connect(...).createTemporaryTable("outputTable");
// 通过 Table API 查询算子，得到一张结果表
Table result = tableEnv.from("inputTable").select(...);
// 通过 SQL查询语句，得到一张结果表
Table sqlResult = tableEnv.sqlQuery("SELECT ... FROM inputTable ...");
// 将结果表写入输出表中
result.insertInto("outputTable");
```

### 12.2.1 创建 TableEnvironment

+ 创建表的执行环境，需要将 flink 流处理的执行环境传入

  ```java
  StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);
  ```

  

+ TableEnvironment 是 flink 中集成 Table API 和 SQL 的核心概念，所有对表的操作都基于 TableEnvironment 

  1. 注册 Catalog
  2. 在 Catalog 中注册表
  3. 执行 SQL 查询
  4. 注册用户自定义函数（UDF）

### 12.2.2 配置 TableEnvironment

1. 配置老版本 planner 的流式查询

   ```java
   EnvironmentSettings settings = EnvironmentSettings.newInstance()
   .useOldPlanner()
   .inStreamingMode()
   .build();
   StreamTableEnvironment tableEnv = StreamTableEnvironment
   .create(env, settings);
   ```

2. 配置老版本 planner 的批式查询

   ```java
   ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment;
   BatchTableEnvironment batchTableEnv = BatchTableEnvironment.create(batchEnv);
   ```

### 12.2.3 配置 TableEnvironment

1. 配置 blink planner 的流式查询

   ```java
   EnvironmentSettings bsSettings = EnvironmentSettings.newInstance()
   .useBlinkPlanner()
   .inStreamingMode()
   .build();
   StreamTableEnvironment bsTableEnv = StreamTableEnvironment
   .create(env, bsSettings);
   ```

2. 配置 blink planner 的批式查询

   ```java
   EnvironmentSettings bbSettings = EnvironmentSettings.newInstance()
   .useBlinkPlanner()
   .inBatchMode()
   .build();
   TableEnvironment bbTableEnv = TableEnvironment.create(bbSettings);
   ```

## 12.3 表（Table） 

+ TableEnvironment 可以注册目录 Catalog，并可以基于 Catalog 注册表
+ 表（Table）是由一个“标识符”（identifier）来指定的，由3部分组成：Catalog名、数据库（database）名和对象名
+ 表可以是常规的，也可以是虚拟的（视图，View） 
+ 常规表（Table）一般可以用来描述外部数据，比如文件、数据库表或消息队列的数据，也可以直接从 DataStream转换而来
+ 视图（View）可以从现有的表中创建，通常是 table API 或者 SQL 查询的一个结果集

### 12.3.1 创建表 

+ TableEnvironment 可以调用 .connect() 方法，连接外部系统，并调用 .createTemporaryTable() 方法，在 Catalog 中注册表

  ```java
  tableEnv
  .connect(...) // 定义表的数据来源，和外部系统建立连接
  .withFormat(...) // 定义数据格式化方法
  .withSchema(...) // 定义表结构
  .createTemporaryTable("MyTable"); // 创建临时表
  ```

+ 可以创建 Table 来描述文件数据，它可以从文件中读取，或者将数据写入文件

  ```java
  tableEnv
  .connect(
  new FileSystem().path(“YOUR_Path/sensor.txt”)
  ) // 定义到文件系统的连接
  .withFormat(new Csv()) // 定义以csv格式进行数据格式化
  .withSchema( new Schema()
  .field("id", DataTypes.STRING())
  .field("timestamp", DataTypes.BIGINT())
  .field("temperature", DataTypes.DOUBLE())
  ) // 定义表结构
  .createTemporaryTable("sensorTable"); // 创建临时表
  ```

   ### 12.3.2 表的查询 – Table API

+ Table API 是集成在 Scala 和 Java 语言内的查询 API

+ Table API 基于代表“表”的 Table 类，并提供一整套操作处理的方法 API；

  这些方法会返回一个新的 Table 对象，表示对输入表应用转换操作的结果

+ 有些关系型转换操作，可以由多个方法调用组成，构成链式调用结构

  ```java
  Table sensorTable = tableEnv.from("inputTable");
  Table resultTable = sensorTable
  .select("id, temperature")
  .filter("id = 'sensor_1'");
  ```

### 12.3.3 表的查询 – SQL

+ Flink 的 SQL 集成，基于实现 了SQL 标准的 Apache Calcite
+ 在 Flink 中，用常规字符串来定义 SQL 查询语句
+ SQL 查询的结果，也是一个新的 Table

```java
Table resultSqlTable = tableEnv
.sqlQuery("select id, temperature from sensorTable where id ='sensor_1'");
```

### 12.3.4 输出表

+ 表的输出，是通过将数据写入 TableSink 来实现的

+ TableSink 是一个通用接口，可以支持不同的文件格式、存储数据库和消息队列

+ 输出表最直接的方法，就是通过 Table.insertInto() 方法将一个 Table 写入注册 

  ```java
  tableEnv.connect(...)
  .createTemporaryTable("outputTable");
  Table resultSqlTable = ...
  resultTable.insertInto("outputTable");
  ```

### 12.3.5 输出到文件

```java
tableEnv.connect(
new FileSystem().path("output.txt")
) // 定义到文件系统的连接
.withFormat(new Csv()) 
.withSchema(new Schema()
.field("id", DataTypes.STRING())
.field("temp", DataTypes.Double())
) 
.createTemporaryTable("outputTable") ; // 创建临时表
resultTable.insertInto("outputTable"); // 输出表
```

### 12.3.6 输出到 Kafka

可以创建 Table 来描述 kafka 中的数据，作为输入或输出的 TableSink

```java
tableEnv.connect(
new Kafka()
.version("0.11")
.topic("sinkTest")
.property("zookeeper.connect", "localhost:2181")
.property("bootstrap.servers", "localhost:9092") )
.withFormat( new Csv() )
.withSchema( new Schema()
.field("id", DataTypes.STRING())
.field("temp", DataTypes.DOUBLE())
)
.createTemporaryTable("kafkaOutputTable");
resultTable.insertInto("kafkaOutputTable");
```

### 12.3.7 输出到 ES

可以创建 Table 来描述 ES 中的数据，作为输出的 TableSink

```java
tableEnv.connect(
new Elasticsearch()
.version("6")
.host("localhost", 9200, "http")
.index("sensor")
.documentType("temp") )
.inUpsertMode()
.withFormat(new Json())
.withSchema( new Schema()
.field("id", DataTypes.STRING())
.field("count", DataTypes.BIGINT())
)
.createTemporaryTable("esOutputTable");
aggResultTable.insertInto("esOutputTable");
```

### 12.3.8 输出到 MySql

可以创建 Table 来描述 MySql 中的数据，作为输入和输出

```java
String sinkDDL=
"create table jdbcOutputTable (" +
" id varchar(20) not null, " + " cnt bigint not null " + ") with (" + " 'connector.type' = 'jdbc', " + " 'connector.url' = 'jdbc:mysql://localhost:3306/test', " + " 'connector.table' = 'sensor_count', " + " 'connector.driver' = 'com.mysql.jdbc.Driver', " + " 'connector.username' = 'root', " + " 'connector.password' = '123456' )";
tableEnv.sqlUpdate(sinkDDL) // 执行 DDL创建表
aggResultSqlTable.insertInto("jdbcOutputTable");
```

### 12.3.9 将 Table 转换成 DataStream

+ 表可以转换为 DataStream 或 DataSet ，这样自定义流处理或批处理程序就可以继续在 Table API 或 SQL 查询的结果上运行了
+ 将表转换为 DataStream 或 DataSet 时，需要指定生成的数据类型，即要将表的每一行转换成的数据类型
+ 表作为流式查询的结果，是动态更新的
+ 转换有两种转换模式：追加（Append）模式和撤回（Retract）模式

### 12.3.10 将 Table 转换成 DataStream

1. 追加模式（Append Mode） 

   用于表只会被插入（Insert）操作更改的场景

   ```java
   DataStream<Row> resultStream = tableEnv.toAppendStream(resultTable, Row.class);
   ```

2. 撤回模式（Retract Mode） 

   + 用于任何场景。有点类似于更新模式中 Retract 模式，它只有 Insert 和 Delete 两类操作
   + 得到的数据会增加一个 Boolean 类型的标识位（返回的第一个字段），用它来表示到底是新增的数据（Insert），还是被删除的数据（Delete）

   ```java
   DataStream<Tuple2<Boolean, Row>> aggResultStream = tableEnv
   .toRetractStream(aggResultTable , Row.class);
   ```

### 12.3.11 将 DataStream 转换成表

+ 对于一个 DataStream，可以直接转换成 Table，进而方便地调用 Table API做转换操作

  ```java
  DataStream<SensorReading> dataStream = ...
  Table sensorTable = tableEnv.fromDataStream(dataStream);
  ```

+ 默认转换后的 Table schema 和 DataStream 中的字段定义一一对应，也可以单独指定出来

  ```java
  DataStream<SensorReading> dataStream = ...
  Table sensorTable = tableEnv.fromDataStream(dataStream, "id, timestamp as ts, temperature");
  ```

### 12.3.12 创建临时视图（Temporary View） 

1. 基于 DataStream 创建临时视图

   ```java
   tableEnv.createTemporaryView("sensorView", dataStream);
   tableEnv.createTemporaryView("sensorView", 
   dataStream, "id, temperature, timestamp as ts");
   ```

2. 基于 Table 创建临时视图

   ```java
   tableEnv.createTemporaryView("sensorView", sensorTable);
   ```

      

## 12.4 更新模式

+ 对于流式查询，需要声明如何在表和外部连接器之间执行转换
+ 与外部系统交换的消息类型，由更新模式（Update Mode）指定

1. 追加（Append）模式

   表只做插入操作，和外部连接器只交换插入（Insert）消息

2. 撤回（Retract）模式

   表和外部连接器交换添加（Add）和撤回（Retract）消息

   插入操作（Insert）编码为 Add 消息；删除（Delete）编码为 Retract 消息；更新（Update）编码为上一条的 Retract 和下一条的 Add 消息

3. 更新插入（Upsert）模式

   更新和插入都被编码为 Upsert 消息；删除编码为 Delete 消息

## 12.5 查看执行计划

+ Table API 提供了一种机制来解释计算表的逻辑和优化查询计划

+ 查看执行计划，可以通过 TableEnvironment.explain(table) 方法或TableEnvironment.explain() 方法完成，返回一个字符串，描述三个计划

  + 优化的逻辑查询计划
  + 优化后的逻辑查询计划
  + 实际执行计划

  ```java
  String explaination = tableEnv.explain(resultTable);
  System.out.println(explaination);
  ```

## 12.6 流处理和关系代数的区别

![image-20210125161828836](./images/85.png)



## 12.7 动态表（Dynamic Tables）

+ 动态表是 Flink 对流数据的 Table API 和 SQL 支持的核心概念
+ 与表示批处理数据的静态表不同，动态表是随时间变化的

持续查询（Continuous Query） 

+ 动态表可以像静态的批处理表一样进行查询，查询一个动态表会产生持续查询（Continuous Query） 
+ 连续查询永远不会终止，并会生成另一个动态表
+ 查询会不断更新其动态结果表，以反映其动态输入表上的更改

## 12.8 动态表和持续查询

![image-20210125163613526](./images/86.png)

流式表查询的处理过程：

1. 流被转换为动态表
2. 对动态表计算连续查询，生成新的动态表
3. 生成的动态表被转换回流

### 12.8.1 将流转换成动态表

+ 为了处理带有关系查询的流，必须先将其转换为表

+ 从概念上讲，流的每个数据记录，都被解释为对结果表的插入（Insert）修改操作

  ![image-20210125163759075](./images/87.png)

### 12.8.2 持续查询

持续查询会在动态表上做计算处理，并作为结果生成新的动态表

![image-20210125163852564](./images/88.png)

### 12.8.3 将动态表转换成 DataStream

+ 与常规的数据库表一样，动态表可以通过插入（Insert）、更新（Update）和删除（Delete）更改，进行持续的修改
+ 将动态表转换为流或将其写入外部系统时，需要对这些更改进行编码

1. 仅追加（Append-only）流

   仅通过插入（Insert）更改来修改的动态表，可以直接转换为仅追加流

2. 撤回（Retract）流

   撤回流是包含两类消息的流：添加（Add）消息和撤回（Retract）消息

3. Upsert（更新插入）流

   Upsert 流也包含两种类型的消息：Upsert 消息和删除（Delete）消息。

### 12.8.4 将动态表转换成 DataStream

![image-20210125164140519](./images/89.png)



## 12.9 时间特性（Time Attributes） 

+ 基于时间的操作（比如 Table API 和 SQL 中窗口操作），需要定义相关的时间语义和时间数据来源的信息
+ Table 可以提供一个逻辑上的时间字段，用于在表处理程序中，指示时间和访问相应的时间戳
+ 时间属性，可以是每个表schema的一部分。一旦定义了时间属性，它就可以作为一个字段引用，并且可以在基于时间的操作中使用
+ 时间属性的行为类似于常规时间戳，可以访问，并且进行计算

### 12.9.1 定义处理时间（Processing Time） 

+ 处理时间语义下，允许表处理程序根据机器的本地时间生成结果。它是时间的最简单概念。它既不需要提取时间戳，也不需要生成 watermark

1. 由 DataStream 转换成表时指定

+ 在定义Schema期间，可以使用.proctime，指定字段名定义处理时间字段

+ 这个proctime属性只能通过附加逻辑字段，来扩展物理schema。因此，只能在schema定义的末尾定义它

  ```java
  Table sensorTable = tableEnv.fromDataStream(dataStream, 
  "id, temperature, timestamp, pt.proctime");
  ```

  

2. 定义 Table Schema 时指定

   ```java
   .withSchema(new Schema()
   .field("id", DataTypes.STRING())
   .field("timestamp", DataTypes.BIGINT())
   .field("temperature", DataTypes.DOUBLE())
   .field("pt", DataTypes.TIMESTAMP(3))
   .proctime()
   )
   ```

3. 在创建表的 DDL 中定义

   ```java
   String sinkDDL =
   "create table dataTable (" + " id varchar(20) not null, " + " ts bigint, " + " temperature double, " + " pt AS PROCTIME() " + ") with (" + " 'connector.type' = 'filesystem', " + " 'connector.path' = '/sensor.txt', " + " 'format.type' = 'csv')";
   tableEnv.sqlUpdate(sinkDDL);
   ```

### 12.9.2 定义事件时间（Event Time） 

+ 事件时间语义，允许表处理程序根据每个记录中包含的时间生成结果。这样即使在有乱序事件或者延迟事件时，也可以获得正确的结果

+ 为了处理无序事件，并区分流中的准时和迟到事件；Flink 需要从事件数据中，提取时间戳，并用来推进事件时间的进展

+ 定义事件时间，同样有三种方法：

  + 由 DataStream 转换成表时指定
  + 定义 Table Schema 时指定
  + 在创建表的 DDL 中定义

1. 由 DataStream 转换成表时指定

   在 DataStream 转换成 Table，使用 .rowtime 可以定义事件时间属性

   ```java
   // 将 DataStream转换为 Table，并指定时间字段
   Table sensorTable = tableEnv.fromDataStream(dataStream, 
   "id, timestamp.rowtime, temperature");
   // 或者，直接追加时间字段
   Table sensorTable = tableEnv.fromDataStream(dataStream, 
   " id, temperature, timestamp, rt.rowtime");
   ```

2. 定义 Table Schema 时指定

   ```java
   .withSchema(new Schema()
   .field("id", DataTypes.STRING())
   .field("timestamp", DataTypes.BIGINT())
   .rowtime(
   new Rowtime()
   .timestampsFromField("timestamp") // 从字段中提取时间戳
   .watermarksPeriodicBounded(1000) // watermark延迟1秒 )
   .field("temperature", DataTypes.DOUBLE())
   )
   ```

3. 在创建表的 DDL 中定义

   ```java
   String sinkDDL=
   "create table dataTable (" + " id varchar(20) not null, " + " ts bigint, " + " temperature double, " + " rt AS TO_TIMESTAMP( FROM_UNIXTIME(ts) ), " + " watermark for rt as rt - interval '1' second" + ") with (" + " 'connector.type' = 'filesystem', " + " 'connector.path' = '/sensor.txt', " + " 'format.type' = 'csv')";
   tableEnv.sqlUpdate(sinkDDL);
   ```

## 12.10 窗口

+ 时间语义，要配合窗口操作才能发挥作用
+ 在 Table API 和 SQL 中，主要有两种窗口

1. Group Windows（分组窗口）

   根据时间或行计数间隔，将行聚合到有限的组（Group）中，并对每个组的数据执行一次聚合函数

2. Over Windows

   针对每个输入行，计算相邻行范围内的聚合

### 12.10.1 Group Windows

+ Group Windows 是使用 window（w:GroupWindow）子句定义的，并且必须由as子句指定一个别名

+ 为了按窗口对表进行分组，窗口的别名必须在 group by 子句中，像常规的分组字段一样引用

  ```java
  Table table = input
  .window([w: GroupWindow] as "w") // 定义窗口，别名为 w
  .groupBy("w, a") // 按照字段 a和窗口 w分组
  .select("a, b.sum"); // 聚合
  ```

  

+ Table API 提供了一组具有特定语义的预定义 Window 类，这些类会被转换

  为底层 DataStream 或 DataSet 的窗口操作

### 12.10.2 滚动窗口（Tumbling windows） 

滚动窗口要用 Tumble 类来定义

```java
// Tumbling Event-time Window
.window(Tumble.over("10.minutes").on("rowtime").as("w"))
// Tumbling Processing-time Window
.window(Tumble.over("10.minutes").on("proctime").as("w"))
// Tumbling Row-count Window
.window(Tumble.over("10.rows").on("proctime").as("w"))
```

### 12.10.3 滑动窗口（Sliding windows） 

滑动窗口要用 Slide 类来定义

```java
// Sliding Event-time Window
.window(Slide.over("10.minutes").every("5.minutes").on("rowtime").as("w"))
// Sliding Processing-time window 
.window(Slide.over("10.minutes").every("5.minutes").on("proctime").as("w"))
// Sliding Row-count window
.window(Slide.over("10.rows").every("5.rows").on("proctime").as("w"))
```

### 12.10.4 会话窗口（Session windows）

会话窗口要用 Session 类来定义

```java
// Session Event-time Window
.window(Session.withGap("10.minutes").on("rowtime").as("w"))
// Session Processing-time Window
.window(Session.withGap("10.minutes").on(“proctime").as("w"))
```

### 12.10.5 SQL 中的 Group Windows

Group Windows 定义在 SQL 查询的 Group By 子句

1. TUMBLE(time_attr, interval)

   定义一个滚动窗口，第一个参数是时间字段，第二个参数是窗口长度

2. HOP(time_attr, interval, interval)

   定义一个滑动窗口，第一个参数是时间字段，第二个参数是窗口滑动步长，第三个是窗口长度

3. SESSION(time_attr, interval)

   定义一个会话窗口，第一个参数是时间字段，第二个参数是窗口间隔

+ 用 Over 做窗口聚合时，所有聚合必须在同一窗口上定义，也就是说必须是相同的分区、排序和范围

+ 目前仅支持在当前行范围之前的窗口

+ ORDER BY 必须在单一的时间属性上指定

  ```sql
  SELECT COUNT(amount) OVER (
  PARTITION BY user
  ORDER BY proctime
  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
  FROM Orders
  ```

  

### 12.10.6 Over Windows

+ Over window 聚合是标准 SQL 中已有的（over 子句），可以在查询的SELECT 子句中定义

+ Over window 聚合，会针对每个输入行，计算相邻行范围内的聚合

+ Over windows 使用 window（w:overwindows*）子句定义，并在 select（）方法中通过别名来引用

  ```java
  Table table = input
  .window([w: OverWindow] as "w")
  .select("a, b.sum over w, c.min over w");
  ```

+ Table API 提供了 Over 类，来配置 Over 窗口的属性

+ Table API 提供了 Over 类，来配置 Over 窗口的属性

### 12.10.7 无界 Over Windows

+ 可以在事件时间或处理时间，以及指定为时间间隔、或行计数的范围内，定义 Over windows

+ 无界的 over window 是使用常量指定的

  ```java
  // 无界的事件时间 over window
  .window(Over.partitionBy("a").orderBy("rowtime").preceding(UNBOUNDED_RANGE).as("w"))
  //无界的处理时间 over window
  .window(Over.partitionBy("a").orderBy("proctime").preceding(UNBOUNDED_RANGE).as("w"))
  // 无界的事件时间 Row-count over window
  .window(Over.partitionBy("a").orderBy("rowtime").preceding(UNBOUNDED_ROW).as("w"))
  //无界的处理时间 Row-count over window
  .window(Over.partitionBy("a").orderBy("proctime").preceding(UNBOUNDED_ROW).as("w"))
  ```

### 12.10.8 有界 Over Windows

有界的 over window 是用间隔的大小指定的

```java
// 有界的事件时间 over window
.window(Over.partitionBy("a").orderBy("rowtime").preceding("1.minutes").as("w"))
// 有界的处理时间 over window
.window(Over.partitionBy("a").orderBy("proctime").preceding("1.minutes").as("w"))
// 有界的事件时间 Row-count over window
.window(Over.partitionBy("a").orderBy("rowtime").preceding("10.rows").as("w"))
// 有界的处理时间 Row-count over window
.window(Over.partitionBy("a").orderBy("procime").preceding("10.rows").as("w"))
```

## 12.11 函数（Functions）

+ Flink Table API 和 SQL 为用户提供了一组用于数据转换的内置函数
+ SQL 中支持的很多函数，Table API 和 SQL 都已经做了实现

![image-20210125170737280](./images/90.png)

![image-20210125170810158](./images/91.png)



### 12.11.1 用户自定义函数（UDF） 

+ 用户定义函数（User-defined Functions，UDF）是一个重要的特性，它们显著地扩展了查询的表达能力
+ 在大多数情况下，用户定义的函数必须先注册，然后才能在查询中使用
+ 函数通过调用 registerFunction（）方法在 TableEnvironment 中注册。当用户定义的函数被注册时，它被插入到 TableEnvironment 的函数目录中，这样Table API 或 SQL 解析器就可以识别并正确地解释它

### 12.11.2 标量函数（Scalar Functions） 

+ 用户定义的标量函数，可以将0、1或多个标量值，映射到新的标量值 

+ 为了定义标量函数，必须在 org.apache.flink.table.functions 中扩展基类Scalar Function，并实现（一个或多个）求值（eval）方法

+ 标量函数的行为由求值方法决定，求值方法必须公开声明并命名为 eval

  ```java
  public static class HashCode extends ScalarFunction {
  private int factor = 13;
  public HashCode(int factor) {
  this.factor = factor;
  }
  public int eval(String s) {
  return s.hashCode() * factor; } }
  ```

### 12.11.3 表函数（Table Functions） 

+ 用户定义的表函数，也可以将0、1或多个标量值作为输入参数；与标量函数不同的是，它可以返回任意数量的行作为输出，而不是单个值

+ 为了定义一个表函数，必须扩展 org.apache.flink.table.functions 中的基类TableFunction 并实现（一个或多个）求值方法

+ 表函数的行为由其求值方法决定，求值方法必须是 public 的，并命名为 eval

  ```java
  public static class Split extends TableFunction<Tuple2<String, Integer>> {
  private String separator = ",";
  public Split(String separator) {
  this.separator = separator;
  }
  public void eval(String str) {
  for (String s : str.split(separator)) {
  collect(new Tuple2<String, Integer>(s, s.length()));
  } } }
  ```

### 12.11.4 聚合函数（Aggregate Functions）

+ 用户自定义聚合函数（User-Defined Aggregate Functions，UDAGGs）可以把一个表中的数据，聚合成一个标量值

+ 用户定义的聚合函数，是通过继承 AggregateFunction 抽象类实现的

  ![image-20210125171446220](./images/92.png)

+ AggregationFunction要求必须实现的方法

  – createAccumulator()

  – accumulate()

  – getValue()

+ AggregateFunction 的工作原理如下

  – 首先，它需要一个累加器（Accumulator），用来保存聚合中间结果的数据结构；

  可以通过调用 createAccumulator() 方法创建空累加器

  – 随后，对每个输入行调用函数的 accumulate() 方法来更新累加器

  – 处理完所有行后，将调用函数的 getValue() 方法来计算并返回最终结果

### 12.11.5 表聚合函数（Table Aggregate Functions） 

+ 用户定义的表聚合函数（User-Defined Table Aggregate Functions，UDTAGGs），可以把一个表中数据，聚合为具有多行和多列的结果表

+ 用户定义表聚合函数，是通过继承 TableAggregateFunction 抽象类来实现的

  ![image-20210125171752927](./images/93.png)

  + AggregationFunction 要求必须实现的方法

    – createAccumulator()

    – accumulate()

    – emitValue()

  + TableAggregateFunction 的工作原理如下

    – 首先，它同样需要一个累加器（Accumulator），它是保存聚合中间结果的数据

    结构。通过调用 createAccumulator() 方法可以创建空累加器。

    – 随后，对每个输入行调用函数的 accumulate() 方法来更新累加器。

    – 处理完所有行后，将调用函数的 emitValue() 方法来计算并返回最终结果。

# 13 .CEP

## 13.1 什么是 CEP

+ 复杂事件处理（Complex Event Processing，CEP）
+ Flink CEP是在 Flink 中实现的复杂事件处理（CEP）库 
+ CEP 允许在无休止的事件流中检测事件模式，让我们有机会掌握数据中重要的部分
+ 一个或多个由简单事件构成的事件流通过一定的规则匹配，然后输出用户想得到的数据 —— 满足规则的复杂事件

## 13.2 CEP 的特点

![image-20210125172553420](./images/94.png)

+ 目标：从有序的简单事件流中发现一些高阶特征
+ 输入：一个或多个由简单事件构成的事件流
+ 处理：识别简单事件之间的内在联系，多个符合一定规则的简单事件构成复杂事件
+ 输出：满足规则的复杂事件

## 13.3 Pattern API

+ 处理事件的规则，被叫做“模式”（Pattern） 

+ Flink CEP 提供了 Pattern API，用于对输入流数据进行复杂事件规则定义，用来提取符合规则的事件序列

  ![image-20210125173125612](./images/95.png)

1. 个体模式（Individual Patterns） 

   组成复杂规则的每一个单独的模式定义，就是“个体模式”

   ![image-20210125173240969](./images/06.png)

2. 组合模式（Combining Patterns，也叫模式序列）

   – 很多个体模式组合起来，就形成了整个的模式序列

   – 模式序列必须以一个“初始模式”开始：

   ![image-20210125173324974](./images/96.png)

3. 模式组（Groups of patterns） 

   将一个模式序列作为条件嵌套在个体模式里，成为一组模式

### 13.3.1 个体模式（Individual Patterns） 

+ 个体模式可以包括“单例（singleton）模式”和“循环（looping）模式”
+ 单例模式只接收一个事件，而循环模式可以接收多个

量词（Quantifier）

可以在一个个体模式后追加量词，也就是指定循环次数

![image-20210125173509359](./images/97.png)

### 13.3.2 个体模式的条件

1. 条件（Condition） 

   – 每个模式都需要指定触发条件，作为模式是否接受事件进入的判断依据

   – CEP 中的个体模式主要通过调用 .where() .or() 和 .until() 来指定条件

   – 按不同的调用方式，可以分成以下几类

   + 简单条件（Simple Condition） 

     通过 .where() 方法对事件中的字段进行判断筛选，决定是否接受该事件

     ![image-20210125173850003](./images/98.png)

   + 组合条件（Combining Condition）

     将简单条件进行合并；.or() 方法表示或逻辑相连，where 的直接组合就是 AND

     ![image-20210125173928707](./images/99.png)

   + 终止条件（Stop Condition） 

     如果使用了 oneOrMore 或者 oneOrMore.optional，建议使用 .until() 作为终止条件，以便清理状态

   + 迭代条件（Iterative Condition） 

     – 能够对模式之前所有接收的事件进行处理

     – 可以调用 ctx.getEventsForPattern(“name”)

     ![image-20210125174027509](./images/100.png)

   

   ## 13.4 模式序列

   ### 13.4.1 不同的“近邻”模式

   ![image-20210125174127961](./images/101.png)

   

1. 严格近邻（Strict Contiguity） 
   + 所有事件按照严格的顺序出现，中间没有任何不匹配的事件，由 .next() 指定
   + 例如对于模式”a next b”，事件序列 [a, c, b1, b2] 没有匹配
2. 宽松近邻（ Relaxed Contiguity ） 
   + 允许中间出现不匹配的事件，由 .followedBy() 指定
   + 例如对于模式”a followedBy b”，事件序列 [a, c, b1, b2] 匹配为 {a, b1}
3. 非确定性宽松近邻（ Non-Deterministic Relaxed Contiguity ） 
   + 进一步放宽条件，之前已经匹配过的事件也可以再次使用，由 .followedByAny() 指定
   + 例如对于模式”a followedByAny b”，事件序列 [a, c, b1, b2] 匹配为 {a, b1}，{a, b2}

除以上模式序列外，还可以定义“不希望出现某种近邻关系”：

+ .notNext() —— 不想让某个事件严格紧邻前一个事件发生
+ .notFollowedBy() —— 不想让某个事件在两个事件之间发生

需要注意：

+ 所有模式序列必须以 .begin() 开始

+ 模式序列不能以 .notFollowedBy() 结束

+ “not” 类型的模式不能被 optional 所修饰

+ 此外，还可以为模式指定时间约束，用来要求在多长时间内匹配有效

  ![image-20210125174723374](./images/102.png)

## 13.4 模式的检测

+ 指定要查找的模式序列后，就可以将其应用于输入流以检测潜在匹配

+ 调用 CEP.pattern()，给定输入流和模式，就能得到一个PatternStream

  ![image-20210125174841725](./images/103.png)

## 13.5 匹配事件的提取

+ 创建 PatternStream 之后，就可以应用 select 或者 flatselect 方法，从检测到的事件序列中提取事件了

+ select() 方法需要输入一个 select function 作为参数，每个成功匹配的事件序列都会调用它

+ select() 以一个 Map<String，List <IN>> 来接收匹配到的事件序列，其中 key 就是每个模式的名称，而 value 就是所有接收到的事件的 List 类型

  ![image-20210125175633079](./images/104.png)

## 13.6 超时事件的提取

+ 当一个模式通过 within 关键字定义了检测窗口时间时，部分事件序列可能因为超过窗口长度而被丢弃；为了能够处理这些超时的部分匹配，select 和 flatSelect API 调用允许指定超时处理程序

+ 超时处理程序会接收到目前为止由模式匹配到的所有事件，由一个 OutputTag 定义接收到的超时事件序列

  ![image-20210125175754798](./images/105.png)







