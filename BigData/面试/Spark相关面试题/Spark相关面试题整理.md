[TOC]

# 1. Spark中创建RDD的4中方式

## 1.1 **从集合（内存）中创建** **RDD**

从集合中创建 RDD，Spark 主要提供了两个方法：parallelize 和 makeRDD

```scala
val sparkConf =
new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val rdd1 = sparkContext.parallelize(
List(1,2,3,4)
)
val rdd2 = sparkContext.makeRDD(
List(1,2,3,4)
)
rdd1.collect().foreach(println)
rdd2.collect().foreach(println)
sparkContext.stop()
```

从底层代码实现来讲，makeRDD 方法其实就是 parallelize 方法

```scala
def makeRDD[T: ClassTag](
seq: Seq[T],
numSlices: Int = defaultParallelism): RDD[T] = withScope {
parallelize(seq, numSlices)
}
```

## 1.2 **从外部存储（文件）创建** **RDD**

由外部存储系统的数据集创建 RDD 包括：本地的文件系统，所有 Hadoop 支持的数据集，比如 HDFS、HBase 等。

```scala
val sparkConf =
new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val fileRDD: RDD[String] = sparkContext.textFile("input")
fileRDD.collect().foreach(println)
sparkContext.stop()
```

## 1.3 **从其他** **RDD** **创建**

主要是通过一个 RDD 运算完后，再产生新的 RDD

## 1.4 **直接创建** RDD（new）

使用 new 的方式直接构造 RDD，一般由 Spark 框架自身使用





# 2. Spark中创建DataFrame的3种方式

## 2.1 通过 Spark 的数据源进行创建

Spark 支持创建文件的数据源格式包括：

```bash
csv format jdbc json load option options orc parquet schema 
table text textFile
```

读取 json 文件创建 DataFrame

```scala
scala> val df = spark.read.json("data/user.json")
df: org.apache.spark.sql.DataFrame = [age: bigint， username: string]
```



## 2.2 从一个存在的 RDD 进行转换



## 2. 3 从 Hive Table 进行查询返回



# 3. Spark是什么

Spark 是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。

# 4. Spark有哪些模块

+ Spark SQL

  Spark SQL 是 Spark 用来操作结构化数据的组件。通过 Spark SQL，用户可以使用 SQL或者 Apache Hive 版本的 SQL 方言（HQL）来查询数据

+ Spark Streaming

  Spark Streaming 是 Spark 平台上针对实时数据进行流式计算的组件，提供了丰富的处理数据流的 API。

+ Spark MLib

  MLlib 是 Spark 提供的一个机器学习算法库。MLlib 不仅提供了模型评估、数据导入等额外的功能，还提供了一些更底层的机器学习原语

+ Spark Graphx

  GraphX 是 Spark 面向图计算提供的框架与算法库

# 5. Spark运行模式

## 5.1 Local模式

不需要其他任何节点资源就可以在本地执行 Spark 代码的环境

提交应用示例：

![image-20210223154522032](./images/1.png)

## 5.2 Standalone模式

Spark 的 Standalone 模式体现了经典的 master-slave 模式。

提交应用示例

![image-20210223154611024](./images/2.png)



## 5.3 Yarn模式

独立部署（Standalone）模式由 Spark 自身提供计算资源，无需其他框架提供资源。这种方式降低了和其他第三方资源框架的耦合性，独立性非常强。但是你也要记住，Spark 主要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以还是和其他专业的资源调度框架集成会更靠谱一些。所以接下来我们来学习在强大的 Yarn 环境下 Spark 是如何工作的（其实是因为在国内工作中，Yarn 使用的非常多）。

提交应用示例

![image-20210223154854411](./images/3.png)

## 5.4 **K8S & Mesos** **模式**

国内使用不多

## 5.5 Windows模式



## 5.6 部署模式对比

![image-20210223155202766](./images/4.png)

# 6. Spark常用端口号

+ Spark 查看当前 Spark-shell 运行任务情况端口号：4040（计算） 
+ Spark Master 内部通信服务端口号：7077
+ Standalone 模式下，Spark Master Web 端口号：8080（资源）
+ Spark 历史服务器端口号：18080
+ Hadoop YARN 任务运行情况查看端口号：8088

# 7. Spark核心组件

## 7.1 **Driver**

Spark 驱动器节点，用于执行 Spark 任务中的 main 方法，负责实际代码的执行工作

Driver 在 Spark 作业执行时主要负责：

+ 将用户程序转化为作业（job） 
+ 在 Executor 之间调度任务(task)
+ 跟踪 Executor 的执行情况
+ 通过 UI 展示查询运行情况

## 7.2 **Executor**

Spark Executor 是集群中工作节点（Worker）中的一个 JVM 进程，负责在 Spark 作业中运行具体任务（Task），任务彼此之间相互独立。Spark 应用启动时，Executor 节点被同时启动，并且始终伴随着整个 Spark 应用的生命周期而存在。如果有 Executor 节点发生了故障或崩溃，Spark 应用也可以继续执行，会将出错节点上的任务调度到其他Executor 节点上继续运行。

Executor 有两个核心功能：

+ 负责运行组成 Spark 应用的任务，并将结果返回给驱动器进程
+ 它们通过自身的块管理器（Block Manager）为用户程序中要求缓存的 RDD 提供内存式存储。RDD 是直接缓存在 Executor 进程内的，因此任务可以在运行时充分利用缓存数据加速运算

## 7.3 **Master & Worker**

Spark 集群的独立部署环境中，不需要依赖其他的资源调度框架，自身就实现了资源调度的功能，所以环境中还有其他两个核心组件：Master 和 Worker，这里的 Master 是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于 Yarn 环境中的 RM, 而Worker 呢，也是进程，一个 Worker 运行在集群中的一台服务器上，由 Master 分配资源对数据进行并行的处理和计算，类似于 Yarn 环境中 NM



## 7.4 **ApplicationMaster**

Hadoop 用户向 YARN 集群提交应用程序时,提交程序中应该包含 ApplicationMaster，用于向资源调度器申请执行任务的资源容器 Container，运行用户自己的程序任务 job，监控整个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。

说的简单点就是，ResourceManager（资源）和 Driver（计算）之间的解耦合靠的就是ApplicationMaster



# 8. 提交流程

## 8.1 **Yarn Client** **模式**

Client 模式将用于监控和调度的 Driver 模块在客户端执行，而不是在 Yarn 中，所以一般用于测试

1. Driver 在任务提交的本地机器上运行
2. ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster，负责向 ResourceManager 申请 Executor 内存
3. ResourceManager 接到 ApplicationMaster 的资源申请后会分配 container，然后ApplicationMaster 在资源分配指定的 NodeManager 上启动 Executor 进程
4. Executor 进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行main 函数
5. 之后执行到 Action 算子时，触发一个 Job，并根据宽依赖开始划分 stage，每个 stage 生成对应的 TaskSet，之后将 task 分发到各个 Executor 上执行

## 8.2 **Yarn Cluster** **模式**

Cluster 模式将用于监控和调度的 Driver 模块启动在 Yarn 集群资源中执行。一般应用于实际生产环境

1. 在 YARN Cluster 模式下，任务提交后会和 ResourceManager 通讯申请启动ApplicationMaster
2. 随后 ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster，此时的 ApplicationMaster 就是 Driver
3. Driver 启动后向 ResourceManager 申请 Executor 内存，ResourceManager 接到ApplicationMaster 的资源申请后会分配 container，然后在合适的 NodeManager 上启动Executor 进程
4. Executor 进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行main 函数
5. 之后执行到 Action 算子时，触发一个 Job，并根据宽依赖开始划分 stage，每个 stage 生成对应的 TaskSet，之后将 task 分发到各个 Executor 上执行

# 9. Spark 3大数据结构

## 9.1 RDD : 弹性分布式数据集



## 9.2 累加器：分布式共享只写变量





## 9.3 广播变量：分布式共享只读变量





# 10. map 和 mapPartitions 的区别？

## 10.1 数据处理角度

Map 算子是分区内一个数据一个数据的执行，类似于串行操作。而 mapPartitions 算子是以分区为单位进行批处理操作

## 10.2 功能的角度

Map 算子主要目的将数据源中的数据进行转换和改变。但是不会减少或增多数据。

MapPartitions 算子需要传一个迭代器，返回一个迭代器，没有要求的元素的个数保持不变，所以可以增加或减少数据

## 10.3 性能的角度

Map 算子因为类似于串行操作，所以性能比较低，而是 mapPartitions 算子类似于批处理，所以性能较高。但是 mapPartitions 算子会长时间占用内存，那么这样会导致内存可能不够用，出现内存溢出的错误。所以在内存有限的情况下，不推荐使用。使用 map 操作。



# 11. zip的几个问题(待解决)

如果两个 RDD 数据类型不一致怎么办？

如果两个 RDD 数据分区不一致怎么办？

如果两个 RDD 分区数据数量不一致怎么办？





# 12. **partitionBy**的几个问题(待解决)

如果重分区的分区器和当前 RDD 的分区器一样怎么办？

Spark 还有其他分区器吗？

如果想按照自己的方法进行数据分区怎么办



# 13. reduceByKey 和 groupByKey 的区别？

1. **从** **shuffle** **的角度**

   reduceByKey 和 groupByKey 都存在 shuffle 的操作，但是 reduceByKey可以在 shuffle 前对分区内相同 key 的数据进行预聚合（combine）功能，这样会减少落盘的数据量，而 groupByKey 只是进行分组，不存在数据量减少的问题，reduceByKey 性能比较高

2. **从功能的角度**

   reduceByKey 其实包含分组和聚合的功能。GroupByKey 只能分组，不能聚合，所以在分组聚合的场合下，推荐使用 reduceByKey，如果仅仅是分组而不需要聚合。那么还是只能使用 groupByKey

# 14. reduceByKey、foldByKey、aggregateByKey、combineByKey 的区别

reduceByKey: 相同 key 的第一个数据不进行任何计算，分区内和分区间计算规则相同

FoldByKey: 相同 key 的第一个数据和初始值进行分区内计算，分区内和分区间计算规则相同

AggregateByKey：相同 key 的第一个数据和初始值进行分区内计算，分区内和分区间计算规则可以不相同

CombineByKey:当计算时，发现数据结构不满足要求时，可以让第一个数据转换结构。分区内和分区间计算规则不相同



# 15. join操作如果 key 存在不相等的情况会是什么结果(待解决)









# 16. RDD转换算子有哪些？

### 9.1.2 RDD转换算子有哪些？

根据数据处理方式的不同分为3类：Value类型、双Value类型、Key-Value类型

1. Value类型

   + map

     将处理的数据逐条进行映射转换

   + **mapPartitions**

     将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据

   + **mapPartitionsWithIndex**

     将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据，在处理时同时可以获取当前分区索引。

   + **flatMap**

     将处理的数据进行扁平化后再进行映射处理

   + **glom**

     将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变

   + **groupBy**

     将数据根据指定的规则进行分组, 分区默认不变，但是数据会被打乱重新组合，我们将这样的操作称之为 shuffle。极限情况下，数据可能被分在同一个分区中一个组的数据在一个分区中，但是并不是说一个分区中只有一个组

   + **filter**

     将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。

     当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现数据倾斜

   + **sample**

     根据指定的规则从数据集中抽取数据

   + **distinct**

     将数据集中重复的数据去重

   + **coalesce**

     根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率

     当 spark 程序中，存在过多的小任务的时候，可以通过 coalesce 方法，收缩合并分区，减少分区的个数，减小任务调度成本

   + **repartition**

     该操作内部其实执行的是 coalesce 操作，参数 shuffle 的默认值为 true。无论是将分区数多的RDD 转换为分区数少的 RDD，还是将分区数少的 RDD 转换为分区数多的 RDD，repartition操作都可以完成，因为无论如何都会经 shuffle 过程

   + **sortBy**

     该操作用于排序数据。在排序之前，可以将数据通过 f 函数进行处理，之后按照 f 函数处理的结果进行排序，默认为升序排列。排序后新产生的 RDD 的分区数与原 RDD 的分区数一致。中间存在 shuffle 的过程

     

2. 双Value类型

   + **intersection**

     对源 RDD 和参数 RDD 求交集后返回一个新的 RDD

   + **union**

     对源 RDD 和参数 RDD 求并集后返回一个新的 RDD

   + **subtract**

     以一个 RDD 元素为主，去除两个 RDD 中重复元素，将其他元素保留下来。求差集

   + **zip**

     将两个 RDD 中的元素，以键值对的形式进行合并。其中，键值对中的 Key 为第 1 个 RDD中的元素，Value 为第 2 个 RDD 中的相同位置的元素。

     

3. Key-Value类型

   + **partitionBy**

     将数据按照指定 Partitioner 重新进行分区。Spark 默认的分区器是 HashPartitioner

   + **reduceByKey**

     可以将数据按照相同的 Key 对 Value 进行聚合

   + **groupByKey**

     将数据源的数据根据 key 对 value 进行分组

   + **aggregateByKey**

     将数据根据不同的规则进行分区内计算和分区间计算

   + **foldByKey**

     当分区内计算规则和分区间计算规则相同时，aggregateByKey 就可以简化为 foldByKey

   + **combineByKey**

     最通用的对 key-value 型 rdd 进行聚集操作的聚集函数（aggregation function）。类似于

     aggregate()，combineByKey()允许用户返回值的类型与输入不一致

   + **sortByKey**

     在一个(K,V)的 RDD 上调用，K 必须实现 Ordered 接口(特质)，返回一个按照 key 进行排序的

   + **join**

     在类型为(K,V)和(K,W)的 RDD 上调用，返回一个相同 key 对应的所有元素连接在一起的(K,(V,W))的 RDD

   + **leftOuterJoin**

     类似于 SQL 语句的左外连接

   + **cogroup**

     在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的 RDD



# 17.什么是RDD?

RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合



# 18. RDD的行动算子有哪些?

1. **reduce**

   聚集 RDD 中的所有元素，先聚合分区内数据，再聚合分区间数据

2. **collect**

   在驱动程序中，以数组 Array 的形式返回数据集的所有元素

3. **count**

   返回 RDD 中元素的个数

4. **first**

   返回 RDD 中的第一个元素

5. **take**

   返回一个由 RDD 的前 n 个元素组成的数组

6. **takeOrdered**

   返回该 RDD 排序后的前 n 个元素组成的数组

7. **aggregate**

   分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合

8. **fold**

   折叠操作，aggregate 的简化版操作

9. **countByKey**

   统计每种 key 的个数

10. **save** **相关算子**

    +  saveAsTextFile
    + saveAsObjectFile
    + saveAsSequenceFile

11. **foreach**

    分布式遍历 RDD 中的每一个元素，调用指定函数



# 19. 描述一下RDD序列化的闭包检测

从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor端执行。那么在 scala 的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就形成了闭包的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给 Executor端执行，就会发生错误，所以需要在执行任务计算前，检测闭包内的对象是否可以进行序列化，这个操作我们称之为闭包检测。Scala2.12 版本后闭包编译方式发生了改变



# 20. 描述一下**RDD** **血缘关系**

RDD 只支持粗粒度转换，即在大量记录上执行的单个操作。将创建 RDD 的一系列 Lineage（血统）记录下来，以便恢复丢失的分区。RDD 的 Lineage 会记录 RDD 的元数据信息和转换行为，当该 RDD 的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。



# 21. 描述一下**RDD** **依赖关系**

这里所谓的依赖关系，其实就是两个相邻 RDD 之间的关系



# 22. **RDD** **窄依赖**

窄依赖表示每一个父(上游)RDD 的 Partition 最多被子（下游）RDD 的一个 Partition 使用

# 23. **RDD** **宽依赖**

宽依赖表示同一个父（上游）RDD 的 Partition 被多个子（下游）RDD 的 Partition 依赖，会引起 Shuffle



# 24. RDD任务切分

RDD 任务切分中间分为：Application、Job、Stage 和 Task

+ Application：初始化一个 SparkContext 即生成一个 Application
+ Job：一个 Action 算子就会生成一个 Job
+ Stage：Stage 等于宽依赖(ShuffleDependency)的个数加 1
+ Task：一个 Stage 阶段中，最后一个 RDD 的分区个数就是 Task 的个数

注意：Application->Job->Stage->Task 每一层都是 1 对 n 的关系



# 25. **RDD Cache** **缓存**

RDD 通过 Cache 或者 Persist 方法将前面的计算结果缓存，默认情况下会把数据以缓存在 JVM 的堆内存中。但是并不是这两个方法被调用时立即缓存，而是触发后面的 action 算子时，该 RDD 将会被缓存在计算节点的内存中，并供后面重用

**存储级别**

![image-20210223172755493](./images/5.png)



# 26. **RDD CheckPoint** **检查点**

所谓的检查点其实就是通过将 RDD 中间结果写入磁盘



# 27. 缓存和检查点的区别

1. Cache 缓存只是将数据保存起来，不切断血缘依赖。Checkpoint 检查点切断血缘依赖
2. Cache 缓存的数据通常存储在磁盘、内存等地方，可靠性低。Checkpoint 的数据通常存储在 HDFS 等容错、高可用的文件系统，可靠性高

# 28. RDD有哪些分区器

Spark 目前支持 Hash 分区和 Range 分区，和用户自定义分区。Hash 分区为当前的默认分区。分区器直接决定了 RDD 中分区的个数、RDD 中每条数据经过 Shuffle 后进入哪个分区，进而决定了 Reduce 的个数。

+ 只有 Key-Value 类型的 RDD 才有分区器，非 Key-Value 类型的 RDD 分区的值是 None
+ 每个 RDD 的分区 ID 范围：0 ~ (numPartitions - 1)，决定这个值是属于那个分区的

## 28.1 **Hash** **分区**

对于给定的 key，计算其 hashCode,并除以分区个数取余

## 28.2 **Range** **分区**

将一定范围内的数据映射到一个分区中，尽量保证每个分区数据均匀，而且分区间有序

# 29. 累加器的实现原理

累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后，传回 Driver 端进行 merge。



# 30. 广播变量的实现原理

广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个 Spark 操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表，广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark 会为每个任务分别发送











