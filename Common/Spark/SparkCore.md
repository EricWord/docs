[TOC]



# 1. Spark概述

## 1.1 Spark是什么

![image-20201217210328239](./images/1-spark是什么.png)

+ Spark是一种基于内存的快速、通用、可扩展的大数据分析计算引擎，使用Scala语言编写
+ Spark Core中提供了Spark最基础与核心的功能
+ Spark SQL是Spark用来操作结构化数据的组件。通过spark SQL，用户可以使用SQL或者Apache Hive版本的SQL方言(HQL)来查询数据
+ Spark Streaming 是Spark平台上针对实时数据进行流式计算的组件，提供了丰富的处理数据流的API

## 1.2 一次性数据计算

框架在处理数据的时候，会从存储设备中读取数据，进行逻辑操作，然后将处理的结果重新存储到介质中

![image-20201217211313329](./images/2-一次性数据计算.png)

## 1.3 Spark VS Hadoop

+ Spark和Hadoop的根本差异是多个作业之间的数据通信问题:Spark多个作业之间数据通信是基于内存，而Hadoop是基于磁盘
+ Hadoop MapReduce由于其设计初衷并不是为了满足循环迭代数据流处理，因此在多并行运行的数据可复用场景(比如机器学习、图挖掘算法、交互式数据挖掘算法)中存在诸多计算效率等问题。所以Spark应运而生，Spark就是在传统的MapReduce计算框架的基础上，利用其计算过程的优化，从而大大加快了数据分析、挖掘的运行和读写速度，并将计算单元缩小到更适合并行计算和重复使用的RDD计算模型
+ 机器学习中ALS、凸优化梯度下降等，这些都需要基于数据集或者数据集的衍生数据反复查询反复操作。MR这种模式不太适合，即使多MR串行处理，性能和时间也是一个问题。数据的共享依赖于磁盘。另外一种是交互式数据挖掘，MR显然不擅长。而Spark所基于的Scala语言恰恰擅长函数的处理
+ Spark是一个分布式数据快速分析项目。它的核心技术是弹性分布式数据集(Resilient Distributed Datasets),提供了比MapReduce丰富的模型，可以快速在内存中对数据集进行多次迭代，来支持复杂的数据挖掘算法和图形计算算法
+ Spark Task的启动时间快。Spark采用fork线程的方式，而Hadoop采用创建新的进程的方式
+ Spark只有在shuffle的时候将数据写入磁盘，而Hadoop中多个MR作业之间的数据交互都要依赖于磁盘交互
+ Spark的缓存机制比HDFS的缓存机制高效

在绝大多数的数据计算场景中，Spar确实会比MapReduce更有优势，但是Spark是基于内存的，所以在实际的生产环境中，由于内存的限制，可能会由于内存资源不足导致Job执行失败，此时MapReduce其实是一个更好的选择，所以Spark并不能完全替代MR

## 1.4 Spark核心模块

![image-20201218093630749](./images/3-SparkCore.png)

### 1.4.1 Spark Core

Spark Core中提供了Spark最基础与最核心的功能，Spark其他功能如:Spark SQL，Spark Streaming,GraphX,MLlib都是在Spark Core的基础上进行扩展的

### 1.4.2 Spark SQL

Spark SQL是Spark用来操作结构化数据的组件。通过Spark SQL，用户可以使用SQL或者Apache Hive版本的SQL方言(HQL)来查询数据

### 1.4.3 Spark Streaming

Spark Streaming是Spark平台上针对实时数据进行流式计算的组件，提供了丰富的处理数据流的API

### 1.4.4 Spark MLlib

MLlib 是Spark提供的一个机器学习算法库。MLlib不仅提供了模型评估、数据导入等额外的功能，还提供了一些更底层的机器学习原语

### 1.4.5 Spark GraphX

GraphX是Spark面向图计算提供的框架与算法库



# 2. Spark 快速上手

## 2.1 创建Maven项目

### 2.1.1增加scala插件

![image-20201218100818259](./images/4-scala_plugin.png)

### 2.1.2 增加依赖关系



### 2.1.3 WordCount

![image-20201218101501782](./images/5-wordCount.png)

# 3. Spark运行环境

Spark作为一个数据处理框架和计算引擎，被设计在所有常见的集群环境中运行，在国内工作中主流的环境为Yarn,不过容器式环境也慢慢流行起来。

![image-20201219152836282](./images/6-spark_env.png)

## 3.1 Local模式

所谓Local模式，就是不需要其他任何节点资源就可以在本地执行Spark代码的环境

### 3.1.1 解压缩文件

![image-20201219160103511](./images/7-unzip_file.png)

### 3.1.2 启动Local环境

![image-20201219160202123](./images/8-enter.png)

![image-20201219160230114](./images/9-webui.png)

### 3.1.3 命令行工具

在解压缩文件夹下的data目录中添加word.txt文件。在命令行工具中执行如下代码指令(和IDEA中代码简化版一致)

![image-20201219161442318](./images/10-command.png)

### 3.1.4 退出本地模式

Ctrl + c 或者输入Scala命令

```scala
:quit
```

### 3.1.5 提交应用

```scala
./spark-submit  \
 --class org.apache.spark.examples.SparkPi \
 --master local[2] \
 ../examples/jars/spark-examples_2.12-3.0.0.jar \
 10
```

+ --class 表示要执行的程序的主类，此处可以替换成自己写的程序
+ --master local[2] 部署模式，默认为本地模式，数字表示分配的虚拟CPU核数量
+ spark-examples_2.12-3.0.0.jar 运行的应用类所在的jar包，实际使用时，可以设定为自己打的Jar包
+ 数字10 表示程序的入口参数，用于设定当前应用的任务数量

![image-20201219162257145](./images/11-console.png)

## 3.2 Standalone模式

Spark的Standalone模式体现了经典的master-slave模式

集群规划:

![image-20201219164225567](./images/12-cluster_setting.png)

### 3.2.1 解压缩文件

![image-20201219164322624](/Users/cuiguangsong/go/src/docs/BigData/Spark/images/13-unzip2.png)

### 3.2.2 修改配置文件

+ 进入解压缩后路径的conf目录，修改slaves.template文件名为slaves

  ```shell
  mv slaves.template slaves
  ```

+ 修改slaves文件，添加work节点

  ```shell
  Hadoop02
  Hadoop03
  Hadoop04
  ```

+ 修改spark-env.sh.template文件名为spark-env.sh

  ```shell
  mv spark-env.sh.template spark-env.sh
  ```

+ 在spark-env.sh文件中追加JAVA_HOME环境变量和集群对应的master节点

  ```shell
  export JAVA_HOME=/opt/jdk1.8.0_261
  SPARK_MASTER_HOST=Hadoop02
  SPARK_MASTER_PORT=7077
  ```

  注意:7077端口，相当于Hadoop3内部通信的8020端口，此处的端口需要确定自己的Hadoop配置

+  分发spark-standlone目录

  ```shell
  xsync.sh  spark-standalone
  ```

  

### 3.2.3 启动集群

+ 执行脚本命令

  ```shell
  sbin/start-all.sh
  ```

  ![image-20201219165250215](./images/14-console.png)

+ 查看三台服务器的运行进程

  ![image-20201219165339550](/Users/cuiguangsong/go/src/docs/BigData/Spark/images/15-jps.png)

![image-20201219165408012](./images/16-jps.png)

![image-20201219165442453](./images/17-jps.png)



+ 查看Master资源监控Web UI界面:http://hadoop02:8080/

  ![image-20201219165621093](./images/18-webui.png)

### 3.2.4 提交应用

```shell
./spark-submit  \
 --class org.apache.spark.examples.SparkPi \
 --master spark://Hadoop02:7077  \
 ../examples/jars/spark-examples_2.12-3.0.0.jar \
 10
```

+ --class 表示要执行程序的主类

+ --master spark://Hadoop02:7077 独立部署模式，连接到Spark集群

+ spark-examples_2.12-3.0.0.jar 运行类所在的Jar包

+ 数字10表示程序的入口参数，用于设定当前应用的任务数量

  ![image-20201219170116397](./images/19.png)

+ 执行任务时会产生多个Java进程

  ![image-20201219170208322](./images/20.png)

+ 执行任务时，默认采用服务器集群节点的总核数，每个节点内存 1024M

  ![image-20201219170416035](./images/21.png)

### 3.2.5 提交参数说明

在提交应用中一般会同时提交一些参数

```shell
bin/spark-submit \
--class <main-class>
--master <master-url> \
... # other options
<application-jar> \
[application-arguments]
```

![image-20201219171202577](./images/22.png)

![image-20201219171222708](./images/23.png)

### 3.2.6 配置历史服务器

由于spark-shell停止后，集群监控Hadoop02:4040页面就看不到历史任务的运行情况，所以开发时都配置历史服务器记录任务运行情况

+ 修改spark-defaults.conf.template文件名为spark-defaults.conf

+ 修改spark-defaults.conf文件，配置日志存储路径

  ```shell
  spark.eventLog.enabled true
  spark.eventLog.dir hdfs://Hadoop02:9000/directory
  ```

注意:需要启动Hadoop集群，HDFS上的directory目录需要提前存在

```shell
sbin/start-dfs.sh
hadoop fs -mkdir /directory
```

+ 修改spark-env.sh文件，添加日志配置

  ```shell
  export SPARK_HISTORY_OPTS="
  -Dspark.history.ui.port=18080
  -Dspark.history.fs.logDirectory=hdfs://Hadoop02:9000/directory
  -Dspark.history.retainedApplications=30"
  ```

  1. 参数1含义:WEB UI访问的端口为18080
  2. 参数2含义:指定历史服务器日志存储路径
  3. 参数3含义:指定保存Application历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数

+ 分发配置文件

  ```shell
  xsync.sh  spark-defaults.conf
  ```

+ 重新执行任务

  ```shell
  ./spark-submit  \
   --class org.apache.spark.examples.SparkPi \
   --master spark://Hadoop02:7077  \
   ../examples/jars/spark-examples_2.12-3.0.0.jar \
   10
  ```

  ![image-20201219203138714](./images/24.png)

+ 查看历史服务:Hadoop02:18080

  ![image-20201219203241314](./images/25.png)

### 3.2.7 配置高可用(HA)

所谓的高可用是因为当前集群中的Master节点只有一个，所以会存在单点故障问题。所以为了解决单点故障问题，需要在集群中配置多个Master节点，一旦处于活动状态的Master发生故障时，由备用Masterr提供服务，保证作业可以继续执行。这里的高可用一般采用Zookeeper设置。

集群规划:

![image-20201219205959104](./images/26.png)

+ 停止集群

  ```shell
  sbin/stop-all.sh
  ```

+ 启动Zookeeper

  ```shell
  sh zkServer.sh start
  ```

+ 修改spark-env.sh文件添加如下配置

  ```shell
  #注释掉下面这两行
  # SPARK_MASTER_HOST=Hadoop02
  # SPARK_MASTER_PORT=7077
  
  #Master监控页面默认访问端口为8080，但是可能会和Zookeeper冲突，所以改成8989，也可以自定义
  SPARK_MASTER_WEBUI_PORT=8989
  export SPARK_DAEMON_JAVA_OPTS="
  -Dspark.deploy.recoveryMode=ZOOKEEPER
  -Dspark.deploy.zookeeper.url=Hadoop02,Hadoop03,Hadoop04
  -Dspark.deploy.zookeeper.dir=/spark"
  ```

+  分发配置文件

  ```shell
  xsync.sh spark-env.sh
  ```

+ 启动集群

  ```shell
  sbin/start-all.sh
  ```

  ![image-20201219210656777](./images/27.png)

+ 启动Hadoop03的单独的Master节点，此时Hadoop02节点Master状态处于备用状态

  ```shell
  sbin/start-master.sh
  ```

  ![image-20201219210847897](/Users/cuiguangsong/go/src/docs/BigData/Spark/images/28.png)

+ 提交应用到高可用集群

  ```shell
  ./spark-submit  \
   --class org.apache.spark.examples.SparkPi \
   --master spark://Hadoop02:7077,Hadoop03:7077  \
   ../examples/jars/spark-examples_2.12-3.0.0.jar \
   10
  ```

+ 停止Hadoop02的Master资源监控进程

  ![image-20201219211019193](./images/29.png)

+ 查看Hadoop03的Master资源监控Web UI,稍等一段时间后，Hadoop03节点的Master状态提升为活动状态

  ![image-20201219211207646](./images/30.png)

## 3.3 Yarn模式

独立部署(Standalone)模式由Spark自身提供资源，无需其他框架提供资源。这种方式降低了和其他第三方资源框架的耦合性，独立性非常强。但是Spark主要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以还是和其他专业的资源调度框架集成会更靠谱一些。

### 3.3.1 解压缩文件

将spark-3.0.0-bin-hadoop3.2.tgz文件上传到Linux并解压缩，放到指定位置

```shell
tar -zvxf spark-3.0.0-bin-hadoop3.2.tgz  -C ../module/
```

### 3.3.2 修改配置文件

1. 修改 /opt/module/hadoop-3.3.0/etc/hadoop/yarn-site.xml，并分发

   ```xml
   <!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认
   是 true -->
   <property>
   <name>yarn.nodemanager.pmem-check-enabled</name>
   <value>false</value>
   </property>
   <!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认
   是 true -->
   <property>
   <name>yarn.nodemanager.vmem-check-enabled</name>
   <value>false</value>
   </property>
   ```

2. 修改/opt/module/spark-yarn/conf/spark-env.sh,添加如下配置

   ```shell
   YARN_CONF_DIR=/opt/module/hadoop-3.3.0/etc/hadoop
   JAVA_HOME=/opt/jdk1.8.0_261
   ```

### 3.3.3 启动HDFS以及Yarn集群

+ 启动Zookeeper集群
+ 启动Hadoop集群
+ 启动Spark集群

### 3.3.4 提交应用

```shell
./spark-submit  \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode cluster \
/opt/module/spark-yarn/examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

### 3.3.5 配置历史服务器

1. 修改spark-defaults.conf.template文件名为spark-defaults.conf

2. 修改spark-defaults.conf文件，配置日志存储路径

   ```shell
   spark.eventLog.enabled true
   spark.eventLog.dir hdfs://Hadoop02:9000/directory
   ```

   注意:需要启动hadoop集群，HDFS上的目录需要提前存在

   ```shell
   sbin/start-dfs.sh
   hadoop fs -mkdir /directory
   ```

3. 修改spark-env.sh，添加日志配置

   ```shell
   export SPARK_HISTORY_OPTS="
   -Dspark.history.ui.port=18080
   -Dspark.history.fs.logDirectory=hdfs://Hadoop02:9000/directory
   -Dspark.history.retainedApplications=30"
   ```

   + 参数1含义:WEB UI访问的端口好为18080
   + 参数2含义:指定历史服务器日志存储路径
   + 参数3含义指定保存Application历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数

   

4. 修改spark-defaults.conf

   ```shell
   spark.yarn.historyServer.address=Hadoop02:18080
   spark.history.ui.port=18080
   ```

   

5. 启动历史服务

   ```shell
   sbin/start-history-server.sh
   ```

   

6. 重新提交应用

   ```shell
   ./spark-submit  \
   --class org.apache.spark.examples.SparkPi \
   --master yarn \
   --deploy-mode client \
   /opt/module/spark-yarn/examples/jars/spark-examples_2.12-3.0.0.jar \
   10
   ```

   

7. Web页面查看日志

   ![image-20201221123045644](./images/32.png)

## 3.4 K8S & Mesos模式

Mesos是Apache下的开源分布式资源管理框架，它被称为是分布式系统的内核，在Twitter得到广泛应用，管理着Twitter超过30，0000台服务器上的应用部署，但是在国内，依然使用着传统的Hadoop大数据框架，所以国内使用Mesos框架的并不多，但是原理其实都是差不多。

![image-20201221124231709](/Users/cuiguangsong/go/src/docs/BigData/Spark/images/33.png)

容器化部署是目前业界很流行的一项技术，基于Docker镜像运行能够让用户更加方便地对应用进行管理和运维。容器管理工具最为流行的就是Kubernetes(K8S),而Spark也在最近的版本中支持了K8S部署模式

https://spark.apache.org/docs/latest/running-on-kubernetes.html

![image-20201221124809894](./images/34.png)

## 3.5 Windows模式

Spark提供了可以在windows系统下启动本地集群的方式。

### 3.5.1 解压缩文件

将文件 spark-3.0.0-bin-hadoop3.2.tgz 解压缩到无中文无空格的路径中

### 3.5.2 启动本地环境

1. 执行解压缩文件路径下 bin 目录中的 spark-shell.cmd 文件，启动 Spark 本地环境
   ![image-20201221125028606](./images/35.png)
2. 在 bin 目录中创建 input 目录，并添加 word.txt 文件, 在命令行中输入脚本代码
   ![image-20201221125104297](./images/36.png)

### 3.5.3 命令行提交应用

在 DOS 命令行窗口中执行提交指令

```shell
spark-submit --class org.apache.spark.examples.SparkPi --master 
local[2] ../examples/jars/spark-examples_2.12-3.0.0.jar 10
```

![image-20201221125220422](./images/37.png)

## 3.6 部署对比

![image-20201221125806903](/Users/cuiguangsong/go/src/docs/BigData/Spark/images/38.png)

## 3.7 端口号

+ Spark查看当前Spark-shell运行任务情况端口号:4040(计算)
+ Spark Master内部通信服务端口号:7077
+ Standalone模式下，Spark Master Web端口号：8080(资源)
+ Spark历史服务器端口号:18080
+ Hadoop Yarn任务运行情况查看端口号：8088

# 4. Spark运行架构

## 4.2 运行架构

Spark框架的核心是一个计算引擎，整体来说，它采用了标准master-slave的结构

如下图所示，展示了一个Spark执行时的基本结构。图中的Driver表示master,负责管理整个集群中的作业任务调度。图中的Executor则是slave,负责实际执行任务。

![image-20201221143744232](./images/39.png)

## 4.2 核心组件

从上图可以看出，对于Spark框架有两个核心组件:

### 4.2.1 Driver

Spark驱动器节点，用于执行Spark任务中的main方法，负责实际代码的执行工作。

Driver在Spark作业执行时主要负责:

+ 将用户程序转化为作业(job)
+ 在Executor之间调度任务(task)
+ 跟踪Exexutor的执行情况
+ 通过UI展示查询运行情况

实际上，无法准确地描述Driver的定义，因为在整个的编程过程中没有看到任何有关Driver的字眼。所以简单理解，所谓的Driver就是驱使整个应用运行起来的程序，也称之为Driver类

### 4.2.2 Executor

Spark Executor是集群中工作节点(Worker)中的一个JVM 进程，负责在Spark作业中运行具体任务(Task),彼此之间相互独立。Spark应用启动时，Executor节点被同时启动，并且始终伴随着整个Spark应用的生命周期而存在。如果有Executor节点发生了故障或崩溃，Spark应用也可以继续执行，会将出错节点上的任务调度到其他Executor节点上继续运行

Executor有两个核心功能:

+ 负责运行组成Spark应用的任务，并将结果返回给驱动器进程
+ 它们通过自身的块管理器(Block Manager)为用户程序中要求缓存的RDD提供内存式存储。RDD是直接缓存在Executor进程内的，因此任务可以在运行时充分利用缓存数据加速运算

### 4.2.3 Master & Worker

Spark集群的独立部署环境中，不需要依赖其他的资源调度框架，自身就实现了资源调度的功能，所以环境中还有其他两个核心组件：Master和Worker,这里的Master是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于Yarn环境中的RM，而Worker呢，也是进程，一个Worker运行在集群中的一台服务器上，由Master分配资源对数据进行并行的处理和计算，类似于Yarn环境中NM

### 4.2.4 ApplicationMaster

Hadoop用户向YARN集群提交应用程序时，提交程序中应该包含ApplicationMaster,用于向资源调度器申请执行任务的资源容器Container,运行用户自己的程序任务job,监控整个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。

说的简单点就是ResourceManager(资源)和Driver(计算)之间的解耦合靠的就是ApplicationMaster.

## 4.3核心概念

### 4.3.1 Executor与Core

Spark Executor是集群中运行在工作节点(Worker)中的一个JVM进程，是整个集群中的专门用于计算的节点。在提交应用时，可以提供参数指定计算节点的个数以及对应的资源。这里的资源一般指的是工作节点Executor的内存大小和使用的虚拟CPU核(Core)数量。

应用程序相关启动参数如下:

![image-20201221211502736](./images/40.png)

### 4.3.2 并行度(Parallelism)

在分布式计算框架中一般都是多个任务同时执行，由于任务分布在不同的计算节点进行计算，所以能够真正地实现多任务并行执行，注意这里是并行而不是并发。我们将整个集群并行执行任务的数量称之为并行度。一个作业到底并行度是多少取决于框架的默认配置也可以在应用程序运行过程中动态地修改。

### 4.3.3 有向无环图(DAG)

![image-20201221211852364](./images/41.png)

大数据计算引擎框架我们根据使用方式的不同一般会分为四类，其中第一类就是Hadoop所承载的MapReduce,它将计算分为两个阶段，分别为Map阶段和Reduce阶段。对于上层应用来说，就不得不想方设法去拆分算法，甚至于不得不在上层应用实现多个Job的串联，以完成一个完整的算法，例如迭代计算。由于这样的弊端，催生了支持DAG的框架。因此支持DAG的框架被划分为第二代计算引擎。如Tez以及更上层的Oozie。这里不去细究各种DAG的实现之间的区别，不过对于当时的Tez和Oozie来说，大多还是批处理的任务。接下来就是以Spark为代表的第三代的计算引擎。第三代计算引擎的主要特点是Job内部的DAG支持(不跨越Job),以及实时计算。

这里所谓的有向无环图，并不是真正意义上的图形，而是由Spark程序直接映射成的数据流的高级抽象模型。简单理解就是将整个程序的执行过程用图形表示出来，这样更直观，更便于理解，可以用于表示程序的拓扑结构。

DAG(Directed Acyclic Graph)有向无环图是由点和线组成的拓扑图形，该图形具有方向，不会闭环。

## 4.4 提交流程

所谓的提交流程，其实就是我们开发人员根据需求写的应用程序通过Spark客户端提交给Spark运行环境执行计算的流程。在不同的部署环境中，这个提交过程基本相同，但是又有细微的差别，这里不进行详细的比较。但是因为在国内工作中，将Spark应用部署到Yarn环境中会更多一些，所以下面所讲的提交流程是基于Yarn环境的。

![image-20201222095327596](./images/42.png)

Spark应用程序提交到Yarn环境中执行的时候，一般会有两种部署执行的方式:Client和Cluster。两种模式主要区别在于:Driver程序的运行节点位。

### 4.2.1 Yarn Client模式

Client模式将用于监控和调度的Driver模块在客户端执行，而不是在Yarn中，所以一般用于测试。

+ Driverz在任务提交的本地机器上运行
+ Driver启动后会和ResourceManager通讯申请启动ApplicationMaster
+ ResourceManager分配container，在合适的NodeManager上启动ApplicationMaster,负责向ResourceManager申请Executor内存
+ ResourceManager接到ApplicationMaster的资源申请后会分配container,然后ApplicationMaster在资源分配指定的NodeManager上启动Executor进程。
+ Executor进程启动后会向Driver反向注册，Executor全部注册完成后Driver开始执行Main函数
+ 之后执行到Action算子时，触发一个Job,并根据宽依赖开始划分stage，每个stage生成对应的TaskSet，之后将task分发到各个Executor上执行。

### 4.2.2 Yarn Cluster模式

Cluster模式将用于监控和调度的Driver模块启动在Yarn集群资源中执行。一般应用于实际生产环境。

+ 在Yarn Cluster模式下，任务提交后会和ResourceManager通讯申请启动ApplicationMaster
+ 随后ResourceManager分配container,在合适的NodeManager上启动ApplicationMaster,此时的ApplicationMaster就是Driver
+ Driver启动后ResourceManager申请Executor内存，ResourceManager接到ApplicationMaster的资源申请会分配container,然后在合适的NodeManager上启动Executor进程
+ Executor进程启动后会向Driver反向注册，Executor全部注册完成后Driver开始执行Main函数
+ 之后执行到Actioin算子时，触发一个Job,并根据宽依赖开始划分stage,每个stage生成对应的TaskSet,之后将task分发到各个Executor上执行。

# 5. Spark核心编程

Spark计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。三大数据结构分别是:

+ RDD:弹性分布式数据集
+ 累加器:分布式共享只写变量
+ 广播变量:分布式共享只读变量

## 5.1 RDD

### 5.1.1 什么是RDD

RDD(Resilient Distributed Dataset)叫做弹性分布式数据集，是Spark中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。

1. 弹性
   + 存储的弹性:内存与磁盘的自动切换 
   + 容错的弹性:数据丢失可以自动恢复
   + 计算的弹性:计算出错重试机制
   + 分片的弹性:可根据需要重新分片
2. 分布式:数据存储在大数据集群不同节点上
3. 数据集:RDD封装了计算逻辑，并不保存数据
4. 数据抽象：RDD是一个抽象类，需要子类具体实现
5. 不可变：RDD封装了计算逻辑，是不可以改变的，想要改变，只能产生新的RDD，在新的RDD里面封装计算逻辑
6. 可分区、并行计算

### 5.1.2 核心属性

![image-20201222203415842](./images/43.png)

+ 分区列表
  RDD数据结构中存在分区列表，用于执行任务时并行计算，是实现分布式计算的重要属性
  ![image-20201222203552276](./images/44.png)
+ 分区计算函数
  Spark在计算时，是使用分区函数对每一个分区进行计算
  ![image-20201222203703636](./images/45.png)
+ RDD之间的依赖关系
  RDD是计算模型的封装，当需求中需要将多个计算模型进行组合时，就需要将多个RDD建立依赖关系
  ![image-20201222203915429](/Users/cuiguangsong/go/src/docs/BigData/Spark/images/46.png)

+ 分区器(可选)
  当数据为KV类型数据时，可以通过设定分区器自定义数据的分区
  ![image-20201222204021163](./images/47.png)

+ 首选位置(可选)

  计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算
  ![image-20201222204135230](/Users/cuiguangsong/go/src/docs/BigData/Spark/images/48.png)

### 5.1.3执行原理

从计算的角度来讲，数据处理过程中需要计算资源(内存&CPU)和计算模型(逻辑)，执行时，需要将计算资源和计算模型进行协调和整合。

Spark框架在执行时，先申请资源，然后将应用程序的数据处理逻辑分解成一个一个的计算任务。然后将任务发到已经分配资源的计算节点上，按照指定的计算模型进行数据的计算。最后得到计算结果。

RDD是Spark框架中用于数据处理的核心模型，下面是Yarn环境中，RDD的工作原理：

1. 启动YARN集群环境
   ![image-20201222204751985](./images/49.png)
2. Spark通过申请资源创建调度节点和计算节点
   ![image-20201222204837634](./images/50.png)

3. Spark根据需求将计算逻辑根据分区划分成不同的任务
   ![image-20201222205007785](./images/51.png)

4. 调度节点将任务根据节点状态发送到对应的计算节点进行计算
   ![image-20201222205105124](./images/52.png)

   从以上流程可以看出RDD在整个流程中主要用于将逻辑进行封装，并生成Task发送给Executor节点执行计算

   

### 5.1.4 基础编程

#### 5.1.4.1 RDD创建

在Spark中创建RDD的方式分为4种:

1. 从集合(内存)中创建RDD
   从集合中创建RDD,Spark主要提供了两个方法parallelize和makeRDD

   ```scala
   package net.codeshow.spark.core.rdd.builder
   
   import org.apache.spark.{SparkConf, SparkContext}
   
   object Spark01_RDD_Memory {
     def main(args: Array[String]): Unit = {
       //@todo 准备环境
       val sparkConf = new SparkConf().setMaster("local[*]").setAppName("RDD")
       val sc = new SparkContext(sparkConf)
   
       //@todo 创建RDD
       //从内存中创建RDD,将内存中的集合的数据作为处理的数据源
       val seq = Seq[Int](1, 2, 3, 4)
       //parallelize : 并行
       //    val rdd = sc.parallelize(seq)
       //上行代码的等价写法
       //makeRDD 在底层实现时，还是调用的parallelize
       val rdd = sc.makeRDD(seq)
       rdd.collect().foreach(println)
       //@todo 关闭环境
       sc.stop()
     }
   }
   ```

2. 从外部存储(文件)创建RDD
   由外部存储系统的数据创建RDD包括本地的文件系统、所有Hadoop支持的数据集比如HDFS、HBase等

   ```scala
   package net.codeshow.spark.core.rdd.builder
   
   import org.apache.spark.{SparkConf, SparkContext}
   
   object Spark02_RDD_File {
     def main(args: Array[String]): Unit = {
       //@todo 准备环境
       val sparkConf = new SparkConf().setMaster("local[*]").setAppName("RDD")
       val sc = new SparkContext(sparkConf)
   
       //@todo 创建RDD
       //从文件中创建RDD,将文件中的数据作为处理的数据源
       //路径默认是以当前环境的根路径为基准
       //可以写绝对路径，也可以写相对路径
       //    val rdd = sc.textFile("datas/1.txt")
       //除了可以像上面那样，指定文件的路径，也可以指定多个文件所在的目录
       //    val rdd = sc.textFile("datas")
       //路径还可以使用通配符
       //    val rdd = sc.textFile("datas/1*.txt")
       //path还可以是分布式文件系统路径:HDFS
       val rdd = sc.textFile("hdfs://Hadoop02:9000/test.txt")
       rdd.collect().foreach(println)
       //@todo 关闭环境
       sc.stop()
     }
   }
   ```

3. 从其他RDD创建
   主要是通过一个RDD运算完后，再产生新的RDD

4. 直接创建RDD(new)
   使用new的方式直接构造RDD,一般由Spark框架自身使用。

#### 5.1.4.2 RDD并行度与分区

默认情况下，Spark可以将一个作业切分多个任务后,发送给Executor节点并行计算，而能够并行计算的任务数量我们称之为并行度。这个数量可以在构建RDD时指定。记住，这里的并行执行的任务数量，并不是指的切分任务的数量，不要混淆。

```scala
package net.codeshow.spark.core.rdd.builder

import org.apache.spark.{SparkConf, SparkContext}

object Spark01_RDD_Memory_par {
  def main(args: Array[String]): Unit = {
    //@todo 准备环境
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("RDD")
    sparkConf.set("spark.default.parallelism","3")
    val sc = new SparkContext(sparkConf)

    //@todo 创建RDD
    //RDD的并行度 & 分区
    //makeRDD可以传递两个参数，第二个参数表示分区的数量
    //第二个参数如果不传递会使用默认值
    //scheduler.conf.getInt("spark.default.parallelism", totalCores)
    //    val rdd = sc.makeRDD(List(1, 2, 3, 4), 2)
    //Spark在默认情况下，从配置对象中获取配置参数spark.default.parallelism
    //如果获取不到，则使用totalCores属性，这个属性取值为当前环境的最大核数
    val rdd = sc.makeRDD(List(1, 2, 3, 4))

    //将处理的数据保存成分区文件
    rdd.saveAsTextFile("output")
    //@todo 关闭环境
    sc.stop()
  }
}
```

+ 读取内存数据时，数据可以按照并行度的设定进行数据的分区操作，数据分区规则的Spark核心源码如下:

  ```scala
  def positions(length: Long, numSlices: Int): Iterator[(Int, Int)] = {
  (0 until numSlices).iterator.map { i =>
  val start = ((i * length) / numSlices).toInt
  val end = (((i + 1) * length) / numSlices).toInt
  (start, end)
   }
  }
  ```

+ 读取文件数据时，数据时按照Hadoop文件读取的规则进行切片分区，而切片规则和数据读取的规则有些差异，具体Spark核心源码如下:

  ```scala
  public InputSplit[] getSplits(JobConf job, int numSplits)
  throws IOException {
  long totalSize = 0; // compute total size
  for (FileStatus file: files) { // check we have valid files
  if (file.isDirectory()) {
  throw new IOException("Not a file: "+ file.getPath());
  }
  totalSize += file.getLen();
  }
  long goalSize = totalSize / (numSplits == 0 ? 1 : numSplits);
  long minSize = Math.max(job.getLong(org.apache.hadoop.mapreduce.lib.input.
  FileInputFormat.SPLIT_MINSIZE, 1), minSplitSize);
  ...
  for (FileStatus file: files) {
  ...
  if (isSplitable(fs, path)) {
  long blockSize = file.getBlockSize();
  long splitSize = computeSplitSize(goalSize, minSize, blockSize);
  ...
  }
  protected long computeSplitSize(long goalSize, long minSize,
  long blockSize) {
  return Math.max(minSize, Math.min(goalSize, blockSize));
  }
  ```

#### 5.1.4.3 RDD转换算子

RDD根据数据处理方式的不同将算子整体上分为Value类型、双Value类型和Key-Value类型

+ Value类型	

  1. map

     + 函数签名

       ```scala
       def map[U: ClassTag](f: T => U): RDD[U]
       ```

     + 函数说明
       将处理的数据逐条进行映射转换，这里的转换可以是类型的转换，也可以是值的转换

       ```scala
       val dataRDD: RDD[Int] = sparkContext.makeRDD(List(1,2,3,4))
       val dataRDD1: RDD[Int] = dataRDD.map(
       num => {
         num * 2
       } )
       val dataRDD2: RDD[String] = dataRDD1.map(
       num => {
       "" + num
       } )
       ```

       小功能:从服务器日志数据apache.log中获取用户请求URL资源路径

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark01_RDD_Operator_Transform_test {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
       
           //@todo 算子-map
           val rdd = sc.textFile("datas/apache.log")
           //长的字符串转换成短的字符串
           val mapRDD = rdd.map(
             line => {
               val datas = line.split(" ")
               datas(6)
             }
           )
           mapRDD.collect().foreach(println)
           sc.stop()
         }
       }
       ```

  2. mapPartitions

     + 函数签名

       ```scala
       def mapPartitions[U: ClassTag](
       f: Iterator[T] => Iterator[U],
       preservesPartitioning: Boolean = false): RDD[U]
       ```

     + 函数说明
       将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过来数据

       ```scala
       val dataRDD1: RDD[Int] = dataRDD.mapPartitions(
       datas => {
       datas.filter(_==2)
       } )
       ```

       小功能:获取每个数据分区的最大值

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark02_RDD_Operator_Transform {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
       
           //@todo 算子-map
           val rdd = sc.makeRDD(List(1, 2, 3, 4), 2)
       
           val mpRDD = rdd.mapPartitions(
             iter => {
               List(iter.max).iterator
             }
           )
       
           mpRDD.collect().foreach(println)
       
           sc.stop()
         }
       }
       ```

     + map和mapPartitions的区别
       ***数据处理角度***
       Map算子是分区内一个数据一个数据的执行，类似于串行操作。而mapPartitions算子是以分区为单位进行批处理操作
       ***功能的角度***
       map算子主要目的是将数据源中的数据进行转换和改变。但是不会减少或增多数据。mapPartitions算子需要传递一个迭代器，返回一个迭代器，没有要求元素的个数保持不变，所以可以增加或减少数据
       ***性能的角度***
       map算子因为类似于串行操作，所以性能比较低，而是mapPartitions算子类似于批处理，所以性能较高。但是mapPartitions算子会长时间占用内存，那么这样会导致内存可能不够用，出现内存溢出的错误。所以在内存有限的情况下，不推荐使用。

  3. mapPartitionsWithIndex

     + 函数签名

       ```scala
       def mapPartitionsWithIndex[U: ClassTag](
       f: (Int, Iterator[T]) => Iterator[U],
       preservesPartitioning: Boolean = false): RDD[U]
       ```

     + 函数说明
       将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据，在处理时同时可以获取当前分区索引

       ```scala
       val dataRDD1 = dataRDD.mapPartitionsWithIndex( (index, datas) => {
       datas.map(index, _)
       } )
       ```

       小功能:获取第二个数据分区的数据

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark03_RDD_Operator_Transform {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
           //@todo 算子-map
           val rdd = sc.makeRDD(List(1, 2, 3, 4), 2)
           val mpRDD = rdd.mapPartitionsWithIndex(
             (index, iter) => {
               if (index == 1) {
                 iter
               } else {
                 Nil.iterator
               }
             }
           )
           mpRDD.collect().foreach(println)
           sc.stop()
         }
       }
       ```

  4. flatMap

     + 函数签名

       ```scala
       def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U]
       ```

     + 函数说明
       将处理的数据进行扁平化后再进行映射处理，所以算子也称之为扁平映射

       ```scala
       val dataRDD = sparkContext.makeRDD(List(
       List(1,2),List(3,4)
       ),1)
       val dataRDD1 = dataRDD.flatMap(
       list => list
       )
       ```

       小功能:将List(List(1,2),3,List(4,5))进行扁平化操作

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark04_RDD_Operator_Transform2 {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
       
           //@todo 算子-map
           val rdd = sc.makeRDD(List(List(1, 2), 3, List(4, 5)))
       
           val flatRDD = rdd.flatMap {
             case list: List[_] => list
             case dat => List(dat)
           }
           flatRDD.collect().foreach(println)
           sc.stop()
         }
       }
       ```

  5. glom

     + 函数签名

       ```scala
       def glom(): RDD[Array[T]]
       ```

     + 函数说明
       将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark05_RDD_Operator_Transform {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
       
           //@todo 算子-map
           val rdd = sc.makeRDD(List(1, 2, 3, 4), 2)
           val glomRDD = rdd.glom()
           glomRDD.collect().foreach(data => println(data.mkString(",")))
           sc.stop()
         }
       }
       ```

       小功能:计算所有分区最大值求和(分区内取最大值，分区间最大值求和)

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark05_RDD_Operator_Transform_Test {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
       
           //@todo 算子
           val rdd = sc.makeRDD(List(1, 2, 3, 4), 2)
       
           val glomRDD = rdd.glom()
       
           val maxRDD = glomRDD.map(
             array => {
               array.max
             }
           )
           println(maxRDD.collect().sum)
           sc.stop()
         }
       }
       ```

       

  6. groupBy

     + 函数签名

       ```scala
       def groupBy[K](f: T => K)(implicit kt: ClassTag[K]): RDD[(K, Iterable[T])]
       ```

     + 函数说明
       将数据根据指定的规则进行分组，分区默认不变，但是数据会被打乱重新组合，我们将这样的操作称之为shuffle。极限情况下，数据可能被分在同一个分区中
       一个组的数据在一个分区中，但是并不是说一个分区中只有一个组

       ```scala
       val dataRDD = sparkContext.makeRDD(List(1,2,3,4),1)
       val dataRDD1 = dataRDD.groupBy(
       _%2
       )
       ```

       ***小功能:将List("Hello","Scala","Spark","Hadoop")根据单词首字母进行分组***

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark06_RDD_Operator_Transform1 {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
       
           //@todo 算子
           val rdd = sc.makeRDD(List("Hello", "Spark", "Scala", "Hadoop"), 2)
       
           //分组和分区没有必然的关系
           val groupRDD = rdd.groupBy(_.charAt(0))
           groupRDD.collect().foreach(println)
           sc.stop()
         }
       }
       ```

       小功能:从服务器日志数据apache.log中获取每个时间段访问量

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import java.text.SimpleDateFormat
       import java.util.logging.SimpleFormatter
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark06_RDD_Operator_Transform_Test {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
       
           //@todo 算子
       
           val rdd = sc.textFile("datas/apache.log")
           val timeRDD = rdd.map(
             line => {
               val datas = line.split(" ")
               val time = datas(3)
               //        time.substring(0,)
               val sdf = new SimpleDateFormat("dd/MM/yyyy:HH:mm:ss")
               val date = sdf.parse(time)
       
               val sdf1 = new SimpleDateFormat("HH")
               val hour = sdf1.format(date)
               (hour, 1)
       
             }
           ).groupBy(_._1)
       
           timeRDD.map {
             case (hour, iter) => {
               (hour, iter.size)
             }
           }.collect().foreach(println)
           sc.stop()
         }
       }
       ```

       

  7. filter

     + 函数签名

       ```scala
       def filter(f: T => Boolean): RDD[T]
       ```

     + 函数说明
       将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现数据倾斜。

       ```scala
       val dataRDD = sparkContext.makeRDD(List(
       1,2,3,4
       ),1)
       val dataRDD1 = dataRDD.filter(_%2 == 0)
       ```

       ***小功能:从服务器日志数据apache.log中获取2015年5月17日的请求路径***

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark07_RDD_Operator_Transform_Test {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
       
           //@todo 算子
           val rdd = sc.textFile("datas/apache.log")
           rdd.filter(
             line => {
               val datas = line.split(" ")
               val time = datas(3)
               time.startsWith("17/05/2015")
             }
           ).collect().foreach(println)
           sc.stop()
         }
       }
       ```

       

  8. sample

     + 函数签名

       ```scala
       def sample(
       withReplacement: Boolean,
       fraction: Double,
       seed: Long = Utils.random.nextLong): RDD[T]
       ```

     + 函数说明
       根据指定的规则从数据集中抽取数据

       ```scala
       val dataRDD = sparkContext.makeRDD(List(
       1,2,3,4
       ),1)
       // 抽取数据不放回（伯努利算法）
       // 伯努利算法：又叫 0、1 分布。例如扔硬币，要么正面，要么反面。
       // 具体实现：根据种子和随机算法算出一个数和第二个参数设置几率比较，小于第二个参数要，大于不
       要
       // 第一个参数：抽取的数据是否放回，false：不放回
       // 第二个参数：抽取的几率，范围在[0,1]之间,0：全不取；1：全取；
       // 第三个参数：随机数种子
       val dataRDD1 = dataRDD.sample(false, 0.5)
       // 抽取数据放回（泊松算法）
       // 第一个参数：抽取的数据是否放回，true：放回；false：不放回
       // 第二个参数：重复数据的几率，范围大于等于 0.表示每一个元素被期望抽取到的次数
       // 第三个参数：随机数种子
       val dataRDD2 = dataRDD.sample(true, 2)
       ```

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark08_RDD_Operator_Transform {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
       
           //@todo 算子
           val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10))
           //sample算子需要传递三个参数
           //1.第一个参数表示抽取数据后是否将数据返回true(放回)，false(丢弃)
           //2.第二个参数表示
           // 抽取不放回的场合:数据源中每条数据被抽取的概率
           //抽取放回的场合:
           //基准值的概念
           //3.第三个参数表示，抽取数据时随机算法的种子，如果不传递第三个参数，那么使用的是当前系统时间
           println(rdd.sample(
             true,
             2,
             //      1
           ).collect().mkString(","))
           sc.stop()
         }
       }
       ```

       

  9. distinct

     + 函数签名

       ```scala
       def distinct()(implicit ord: Ordering[T] = null): RDD[T]
       def distinct(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
       ```

     + 函数说明
       将数据集中重复的数据去重

       ```scala
       val dataRDD = sparkContext.makeRDD(List(
       1,2,3,4,1,2
       ),1)
       val dataRDD1 = dataRDD.distinct()
       val dataRDD2 = dataRDD.distinct(2)
       ```

       ```scala
       package net.codeshow.spark.core.rdd.operator.transform
       
       import org.apache.spark.{SparkConf, SparkContext}
       
       object Spark09_RDD_Operator_Transform {
         def main(args: Array[String]): Unit = {
           //@todo 准备环境
           val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
           val sc = new SparkContext(sparkConf)
           //@todo 算子
           val rdd = sc.makeRDD(List(1, 2, 3, 4, 1, 2, 3, 4))
           val rdd1 = rdd.distinct()
           rdd1.collect().foreach(println)
           sc.stop()
         }
       }
       ```

       

  10. coalesce

      + 函数签名

        ```scala
        def coalesce(numPartitions: Int, shuffle: Boolean = false,
        partitionCoalescer: Option[PartitionCoalescer] = Option.empty)
        (implicit ord: Ordering[T] = null)
        : RDD[T]
        ```

      + 函数说明
        根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率，当Spark程序中，存在过多的小任务的时候，可以通过coalesce方法，收缩合并分区，减少分区的个数，减小任务调度成本

        ```scala
        val dataRDD = sparkContext.makeRDD(List(
        1,2,3,4,1,2
        ),6)
        val dataRDD1 = dataRDD.coalesce(2)
        ```

        ```scala
        package net.codeshow.spark.core.rdd.operator.transform
        
        import org.apache.spark.{SparkConf, SparkContext}
        
        object Spark10_RDD_Operator_Transform {
          def main(args: Array[String]): Unit = {
            //@todo 准备环境
            val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
            val sc = new SparkContext(sparkConf)
            //@todo 算子
            val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6), 3)
            //coalesce方法默认情况下不会将分区的数据打乱重新组合
            //这种情况下的缩减分区可能会导致数据不均衡，出现数据倾斜
            //如果想要让数据均衡，可以进行shuffle处理
            //    val newRDD = rdd.coalesce(2)
            val newRDD = rdd.coalesce(2, shuffle = true)
            newRDD.saveAsTextFile("output")
            sc.stop()
          }
        }
        ```

        

  11. repartition

      + 函数签名

        ```scala
        def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
        ```

      + 函数说明
        该操作内部其实执行的是coalesce操作，参数shuffle的默认值为true。无论是将分区数多的RDD转换为分区数少的RDD,还是将分区数少的RDD转换为分区数多的RDD,repartition操作都可以完成，因为无论如何都会经过shuffle过程

        ```scala
        package net.codeshow.spark.core.rdd.operator.transform
        
        import org.apache.spark.{SparkConf, SparkContext}
        
        object Spark11_RDD_Operator_Transform {
          def main(args: Array[String]): Unit = {
            //@todo 准备环境
            val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
            val sc = new SparkContext(sparkConf)
            //@todo 算子
            val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6), 2)
            //coalesce算子可以扩大分区，但是如果不进行shuffle操作是不起作用的
            //所以如果想要实现扩大分区的效果，需要使用shuffle操作
            //Spark提供了一个简化的操作
            //缩减分区:coalesce,如果想要数据均衡，可以采用shuffle
            //扩大分区:repartition
            val newRDD = rdd.repartition(3)
            newRDD.saveAsTextFile("output")
            sc.stop()
          }
        }
        ```

        

  12. sortBy

      + 函数签名

        ```scala
        def sortBy[K](
        f: (T) => K,
          ascending: Boolean = true,
        numPartitions: Int = this.partitions.length)
        (implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T]
        ```

      + 函数说明
        该操作用于排序数据。在排序之前，可以将数据通过f函数进行处理，之后按照f函数处理的结果进行排序，默认为升序排列。排序后新产生的RDD的分区数与原RDD的分区数一致。中间存在shuffle的过程。

        ```scala
        package net.codeshow.spark.core.rdd.operator.transform
        
        import org.apache.spark.{SparkConf, SparkContext}
        
        object Spark12_RDD_Operator_Transform1 {
          def main(args: Array[String]): Unit = {
            //@todo 准备环境
            val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
            val sc = new SparkContext(sparkConf)
            //@todo 算子
            val rdd = sc.makeRDD(List(("1", 1), ("11", 2), ("2", 3)), 2)
            //sortBy方法可以根据指定的规则对数据源中的数据进行排序，默认为升序
            //第二个参数可以改变排序的方式
            //sortBy默认情况下不会改变分区，但是中间存在shuffle操作
            val newRDD = rdd.sortBy(t => t._1.toInt, ascending = false)
            newRDD.collect().foreach(println)
            sc.stop()
          }
        }
        ```

        

  13. 双Value类型

      ***intersection***

      + 函数签名

        ```scala
        def intersection(other: RDD[T]): RDD[T]
        ```

      + 函数说明
        对源RDD和参数RDD求交集后返回一个新的RDD

      ***union***

      + 函数签名

        ```scala
        def union(other: RDD[T]): RDD[T]
        ```

      + 函数说明
        对源RDD和参数RDD求并集后返回一个新的RDD

      ***subtract***

      + 函数签名

        ```scala
        def subtract(other: RDD[T]): RDD[T]
        ```

      + 函数说明
        以一个RDD元素为主，去除两个RDD中重复元素，将其他元素保留下来，求差集

      ***zip***

      + 函数签名

        ```scala
        def zip[U: ClassTag](other: RDD[U]): RDD[(T, U)]
        ```

      + 函数说明
        将两个RDD中的元素，以键值对的形式进行合并。其中，键值对中的key为第1个RDD中的元素，value为第2个RDD中的相同位置的元素

      ```scala
      package net.codeshow.spark.core.rdd.operator.transform
      
      import org.apache.spark.{SparkConf, SparkContext}
      
      object Spark13_RDD_Operator_Transform {
        def main(args: Array[String]): Unit = {
          //@todo 准备环境
          val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
          val sc = new SparkContext(sparkConf)
          //@todo 算子
          val rdd1 = sc.makeRDD(List(1, 2, 3, 4), 2)
          val rdd2 = sc.makeRDD(List(3, 4, 5, 6), 2)
          //交集、并集和差集要求两个数据源数据类型保持一致
          //拉链操作两个数据源的类型可以不一致
          //交集
          val rdd3 = rdd1.intersection(rdd2)
          println(rdd3.collect().mkString(","))
          // 并集
          val rdd4 = rdd1.union(rdd2)
          println(rdd4.collect().mkString(","))
      
          // 差集
          val rdd5 = rdd1.subtract(rdd2)
          println(rdd5.collect().mkString(","))
      
          // 拉链
          //zip要求
          // 两个数据源的分区数要保持一致，否则报 Can't zip RDDs with unequal numbers of partitions
          //两个数据源的每个分区中的数据数量要保持一致，否则报 Can only zip RDDs with same number of elements in each partition
          val rdd6 = rdd1.zip(rdd2)
          println(rdd6.collect().mkString(","))
          sc.stop()
        }
      }
      ```

      

  14. Key-Value类型
      ***partitionBy***

      + 函数签名

        ```scala
        def partitionBy(partitioner: Partitioner): RDD[(K, V)]
        ```

      + 函数说明
        将数据按照指定Partitioner重新进行分区。Spark默认的分区器是HashPartitioner

        ```scala
        package net.codeshow.spark.core.rdd.operator.transform
        
        import org.apache.spark.{HashPartitioner, SparkConf, SparkContext}
        
        object Spark14_RDD_Operator_Transform {
          def main(args: Array[String]): Unit = {
            //@todo 准备环境
            val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
            val sc = new SparkContext(sparkConf)
            //@todo 算子
            val rdd = sc.makeRDD(List(1, 2, 3, 4), 2)
            val mapRDD = rdd.map((_, 1))
        
            //partitionBy 根据指定的分区规则对数据进行重分区
            mapRDD.partitionBy(new HashPartitioner(2)).saveAsTextFile("output")
        
            sc.stop()
          }
        }
        ```

      ***reduceByKey***

      + 函数签名

        ```scala
        def reduceByKey(func: (V, V) => V): RDD[(K, V)]
        def reduceByKey(func: (V, V) => V, numPartitions: Int): RDD[(K, V)]
        ```

      + 函数说明
        可以将数据按照相同的key对value进行聚合

        ```scala
        package net.codeshow.spark.core.rdd.operator.transform
        
        import org.apache.spark.{HashPartitioner, SparkConf, SparkContext}
        
        object Spark15_RDD_Operator_Transform {
          def main(args: Array[String]): Unit = {
            //@todo 准备环境
            val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
            val sc = new SparkContext(sparkConf)
            //@todo 算子
            val rdd = sc.makeRDD(List(
              ("a", 1), ("a", 2), ("a", 3), ("b", 4)
            ))
        
            //reduceByKey:相同的key的数据进行value数据的聚合操作
            //scala语言中一般的聚合操作都是两两聚合，spark基于scala开发,所以spark的聚合也是两两聚合
            //如果key的数据只有1个，是不会参与运算的
            val reduceRDD = rdd.reduceByKey(_ + _)
            reduceRDD.collect().foreach(println)
            sc.stop()
          }
        }
        ```

      ***groupByKey***

      + 函数签名

        ```scala
        def groupByKey(): RDD[(K, Iterable[V])]
        def groupByKey(numPartitions: Int): RDD[(K, Iterable[V])]
        def groupByKey(partitioner: Partitioner): RDD[(K, Iterable[V])]
        ```

      + 函数说明
        将数据源的数据根据key对value进行分组

        ```scala
        package net.codeshow.spark.core.rdd.operator.transform
        
        import org.apache.spark.{SparkConf, SparkContext}
        
        object Spark16_RDD_Operator_Transform {
          def main(args: Array[String]): Unit = {
            //@todo 准备环境
            val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
            val sc = new SparkContext(sparkConf)
            //@todo 算子
            val rdd = sc.makeRDD(List(
              ("a", 1), ("a", 2), ("a", 3), ("b", 4)
            ))
        
            //groupByKey:将数据源中的数据，相同key的数据分在一组，形成一个对偶元组
            //元组中的第一个元素就是key
            //元组中的第二个元素就是相同key的value的集合
            val groupRDD = rdd.groupByKey()
            groupRDD.collect().foreach(println)
            val groupRDD1 = rdd.groupBy(_._1)
            groupRDD1.collect().foreach(println)
            sc.stop()
          }
        }
        ```

        ***reduceByKey和groupByKey的区别***

        ***从shuffle的角度:***reduceByKey和groupByKey都存在shuffle的操作，但是reduceByKey可以在shuffle前对分区内相同key的数据进行预聚合(combine)功能，这样会减少落盘的数据量，而groupByKey只是进行分组，不存在数据量减少的问题，reduceByKey性能比较高
        ***从功能的角度:***reduceByKey其实包含分组和聚合的功能。groupByKey只能分组，不能聚合，所以在分组聚合的场合下，推荐使用reduceByKey,如果仅仅是分组而不需要聚合，那么还是只能使用groupByKey

      ***aggregateByKey***

      + 函数签名

        ```scala
        def aggregateByKey[U: ClassTag](zeroValue: U)(seqOp: (U, V) => U,
        combOp: (U, U) => U): RDD[(K, U)]
        ```

      + 函数说明
        将数据根据不同的规则进行分区内计算和分区间计算

        ```scala
        val dataRDD1 =
        sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
        val dataRDD2 =
        dataRDD1.aggregateByKey(0)(_+_,_+_)
        ```

        取出每个分区内相同key的最大值然后分区间相加

        ```scala
        // TODO : 取出每个分区内相同 key 的最大值然后分区间相加
        // aggregateByKey 算子是函数柯里化，存在两个参数列表
        // 1. 第一个参数列表中的参数表示初始值
        // 2. 第二个参数列表中含有两个参数
        // 2.1 第一个参数表示分区内的计算规则
        // 2.2 第二个参数表示分区间的计算规则
        val rdd =
        sc.makeRDD(List(
        ("a",1),("a",2),("c",3),
        ("b",4),("c",5),("c",6)
        ),2)
        // 0:("a",1),("a",2),("c",3) => (a,10)(c,10)
        // => (a,10)(b,10)(c,20)
        // 1:("b",4),("c",5),("c",6) => (b,10)(c,10)
        val resultRDD =
        rdd.aggregateByKey(10)(
        (x, y) => math.max(x,y),
        (x, y) => x + y
        )
        resultRDD.collect().foreach(println)
        ```

      ***foldByKey***

      + 函数签名

        ```scala
        def foldByKey(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]
        ```

      + 函数说明
        当分区内计算规则和分区间计算规则相同时，aggregateByKey就可以简化为foldByKey

        ```scala
        val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
        val dataRDD2 = dataRDD1.foldByKey(0)(_+_)
        ```

      ***combineByKey***

      + 函数签名

        ```scala
        def combineByKey[C](
        createCombiner: V => C,
        mergeValue: (C, V) => C,
        mergeCombiners: (C, C) => C): RDD[(K, C)]
        ```

      + 函数说明
        最通用的对key-value型rdd进行聚集操作的聚集函数(aggregation function)。 类似于aggregate(),combineByKey()允许用户返回值的类型与输入不一致

      ***小练习***

      将数据 List(("a", 88), ("b", 95), ("a", 91), ("b", 93), ("a", 95), ("b", 98))求每个 key 的平

      均值

      ```scala
      val list: List[(String, Int)] = List(("a", 88), ("b", 95), ("a", 91), ("b", 93), 
      ("a", 95), ("b", 98))
      val input: RDD[(String, Int)] = sc.makeRDD(list, 2)
      val combineRdd: RDD[(String, (Int, Int))] = input.combineByKey(
      (_, 1),
      (acc: (Int, Int), v) => (acc._1 + v, acc._2 + 1),
      (acc1: (Int, Int), acc2: (Int, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2)
      )
      ```

      ***reduceByKey、foldByKey、aggregateByKey、combineByKey的区别***

      reduceByKey:相同Key的第一个数据不进行任何计算，分区内和分区间计算规则相同
      foldByKey:相同key的第一个数据和初始值进行分区内计算，分区内和分区间计算规则相同
      aggregateByKey:相同key的第一个数据和初始值进行分区内计算，分区内和分区间计算规则可以不同
      combineByKey:当计算时，发现数据结构不满足要求时，可以让第一个数据转换结构。分区内和分区间计算规则不同。

      ***sortByKey***

      + 函数签名

        ```scala
        def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length)
        : RDD[(K, V)]
        ```

      + 函数说明
        在一个(k,v)的RDD上调用，K必须实现Ordered接口(特质)，返回一个按照key进行排序的

        ```scala
        val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
        val sortRDD1: RDD[(String, Int)] = dataRDD1.sortByKey(true)
        val sortRDD1: RDD[(String, Int)] = dataRDD1.sortByKey(false)
        ```

      ***join***

      + 函数签名

        ```scala
        def join[W](other: RDD[(K, W)]): RDD[(K, (V, W))]
        ```

      + 函数说明
        在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素连接在一起的(K,(v,W))的RDD

      ***leftOuterJoin***

      + 函数签名

        ```scala
        def leftOuterJoin[W](other: RDD[(K, W)]): RDD[(K, (V, Option[W]))]
        ```

      + 函数说明
        类似于SQL语句的左外连接

        ```scala
        package net.codeshow.spark.core.rdd.operator.transform
        
        import org.apache.spark.{SparkConf, SparkContext}
        
        object Spark22_RDD_Operator_Transform {
          def main(args: Array[String]): Unit = {
            //@todo 准备环境
            val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
            val sc = new SparkContext(sparkConf)
            //@todo 算子
            val rdd1 = sc.makeRDD(List(
              ("a", 1), ("b", 2) //, ("c", 3),
            ))
        
        
            val rdd2 = sc.makeRDD(List(
              ("a", 4), ("b", 5), ("c", 6),
            ))
        
            val leftJoinRDD = rdd1.leftOuterJoin(rdd2)
            val rightJoinRDD = rdd1.rightOuterJoin(rdd2)
        
            leftJoinRDD.collect().foreach(println)
            //    rightJoinRDD.collect().foreach(println)
            sc.stop()
          }
        }
        ```

      ***cogroup***

      + 函数签名

        ```scala
        def cogroup[W](other: RDD[(K, W)]): RDD[(K, (Iterable[V], Iterable[W]))]
        ```

      + 函数说明
        在类型为(K,V)和(K,W)的RDD上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的RDD

        ```scala
        package net.codeshow.spark.core.rdd.operator.transform
        
        import org.apache.spark.{SparkConf, SparkContext}
        
        object Spark23_RDD_Operator_Transform {
          def main(args: Array[String]): Unit = {
            //@todo 准备环境
            val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
            val sc = new SparkContext(sparkConf)
            //@todo 算子
            val rdd1 = sc.makeRDD(List(
              ("a", 1), ("b", 2) //, ("c", 3),
            ))
            val rdd2 = sc.makeRDD(List(
              ("a", 4), ("b", 5), ("c", 6),
            ))
            //cogroup: connect + group
            val cgRDD = rdd1.cogroup(rdd2)
            cgRDD.collect().foreach(println)
            sc.stop()
          }
        }
        ```

#### 5.1.4.4 案例实操

1. 数据准备
   agent.log:时间戳、省份、城市、用户、广告，中间字段使用空格分隔
2. 需求描述
   统计出每一个省份每个广告被点击数量排行的Top3
3. 需求分析
4. 功能实现
   

#### 5.1.4.5 RDD行动算子

1. reduce 

   + 函数签名

     ```scala
     def reduce(f: (T, T) => T): T
     ```

   + 函数说明
     聚集RDD中的所有元素，先聚合分区内数据，再聚合分区间数据

     ```scala
     val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
     // 聚合数据
     val reduceResult: Int = rdd.reduce(_+_)
     ```

     

2. collect

   + 函数签名

     ```scala
     def collect(): Array[T]
     ```

   + 函数说明
     在驱动程序中，以数组Array的形式返回数据集的所有元素

     ```scala
     val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
     // 收集数据到 Driver
     rdd.collect().foreach(println)
     ```

     

3. count

   + 函数签名

     ```scala
     def count(): Long
     ```

   + 函数说明
     返回RDD中元素的个数

     ```scala
     val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
     // 返回 RDD 中元素的个数
     val countResult: Long = rdd.count()
     ```

     

4. first

   + 函数签名

     ```scala
     def first(): T
     ```

   + 函数说明
     返回RDD中的第一个元素

     ```scala
     val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
     // 返回 RDD 中元素的个数
     val firstResult: Int = rdd.first()
     println(firstResult)
     ```

     

5. take

   + 函数签名

     ```scala
     def take(num: Int): Array[T]
     ```

   + 函数说明
     返回一个由RDD的前n个元素组成的数组

     ```scala
     vval rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
     // 返回 RDD 中元素的个数
     val takeResult: Array[Int] = rdd.take(2)
     println(takeResult.mkString(","))
     ```

     

6. takeOrdered

   + 函数签名

     ```scala
     def takeOrdered(num: Int)(implicit ord: Ordering[T]): Array[T]
     ```

   + 函数说明
     返回该RDD排序后的前n个元素组成的数组

     ```scala
     val rdd: RDD[Int] = sc.makeRDD(List(1,3,2,4))
     // 返回 RDD 中元素的个数
     val result: Array[Int] = rdd.takeOrdered(2)
     ```

     

7. aggregate

   + 函数签名

     ```scala
     def aggregate[U: ClassTag](zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U): U
     ```

   + 函数说明
     分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合

     ```scala
     val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4), 8)
     // 将该 RDD 所有元素相加得到结果
     //val result: Int = rdd.aggregate(0)(_ + _, _ + _)
     val result: Int = rdd.aggregate(10)(_ + _, _ + _)
     ```

     

8. fold

   + 函数签名

     ```scala
     def fold(zeroValue: T)(op: (T, T) => T): T
     ```

   + 函数说明
     折叠操作，aggregate的简化版操作

     ```scala
     val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4))
     val foldResult: Int = rdd.fold(0)(_+_)
     ```

     

9. countByKey

   + 函数签名

     ```scala
     def countByKey(): Map[K, Long]
     ```

   + 函数说明
     统计每种key的个数

     ```scala
     val rdd: RDD[(Int, String)] = sc.makeRDD(List((1, "a"), (1, "a"), (1, "a"), (2, 
     "b"), (3, "c"), (3, "c")))
     // 统计每种 key 的个数
     val result: collection.Map[Int, Long] = rdd.countByKey()
     ```

     

10. save相关算子

    + 函数签名 

      ```scala
      def saveAsTextFile(path: String): Unit
      def saveAsObjectFile(path: String): Unit
      def saveAsSequenceFile(
      path: String,
      codec: Option[Class[_ <: CompressionCodec]] = None): Unit
      ```

    + 函数说明
      将数据保存到不同格式的文件中

      ```scala
      // 保存成 Text 文件
      rdd.saveAsTextFile("output")
      // 序列化成对象保存到文件
      rdd.saveAsObjectFile("output1")
      // 保存成 Sequencefile 文件
      rdd.map((_,1)).saveAsSequenceFile("output2")
      ```

      

11. foreach

    + 函数签名

      ```scala
      def foreach(f: T => Unit): Unit = withScope {
      val cleanF = sc.clean(f)
      sc.runJob(this, (iter: Iterator[T]) => iter.foreach(cleanF))
      }
      ```

    + 函数说明
      分布式遍历RDD中的每一个元素，调用指定函数

      ```scala
      val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
      // 收集后打印
      rdd.map(num=>num).collect().foreach(println)
      println("****************")
      // 分布式打印
      rdd.foreach(println)
      ```



#### 5.1.4.6 RDD序列化

1. 闭包检查
   从计算的角度，算子以外的代码都是在Driver端执行，算子里面的代码都是在Executor端执行。那么在scala的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就形成了闭包的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给Executor端执行，就会发生错误，所以需要在执行任务计算之前，检测闭包内的对象是否可以进行序列化，这个操作我们称之为闭包检测。scala2.12版本后闭包编译方式发生了改变

2. 序列化方法和属性

   

   ```scala
   package net.codeshow.spark.core.rdd.serial
   
   import org.apache.spark.rdd.RDD
   import org.apache.spark.{SparkConf, SparkContext}
   
   object Spark01_RDD_Serial {
     def main(args: Array[String]): Unit = {
       val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
       val sc = new SparkContext(sparkConf)
   
       val rdd = sc.makeRDD(Array("hello world", "hello spark", "hive", "atguigu"))
       val search = new Search("h")
       //    search.getMatch1(rdd).collect().foreach(println)
       search.getMatch2(rdd).collect().foreach(println)
     }
   
     //查询对象
     //类的构造参数其实是类的属性
     //构造参数需要进行闭包就检测，其实就等同于类进行闭包检测
     class Search(query: String) {
   
       def isMatch(s: String): Boolean = {
         s.contains(query)
       }
   
       def getMatch1(rdd: RDD[String]): RDD[String] = {
         rdd.filter(isMatch)
       }
   
       def getMatch2(rdd: RDD[String]): RDD[String] = {
         val s = query
         rdd.filter(x => x.contains(s))
       }
     }
   }
   ```

3. Kryo序列化框架
   参考地址: https://github.com/EsotericSoftware/kryo
   Java的序列化能够序列化任何的类，但是比较重(序列化后字节多)，序列化后，对象的提交也比较大。Spark出于性能的考虑，Spark2.0开始支持Kryo序列化机制。Kryo速度是Serializable的10倍。当RDD在shuffle数据的时候，简单数据类型、数组和字符串类型已经在Spark内部使用Kryo来序列化。
   ***注意:***即使使用Kryo序列化，也要继承Serializable接口。

   ```scala
   object serializable_Kryo {
   def main(args: Array[String]): Unit = {
   val conf: SparkConf = new SparkConf()
   .setAppName("SerDemo")
   .setMaster("local[*]")
   // 替换默认的序列化机制
   .set("spark.serializer", 
   "org.apache.spark.serializer.KryoSerializer")
   // 注册需要使用 kryo 序列化的自定义类
   .registerKryoClasses(Array(classOf[Searcher]))
   val sc = new SparkContext(conf)
   val rdd: RDD[String] = sc.makeRDD(Array("hello world", "hello atguigu", 
   "atguigu", "hahah"), 2)
   val searcher = new Searcher("hello")
   val result: RDD[String] = searcher.getMatchedRDD1(rdd)
   result.collect.foreach(println)
   } }
   case class Searcher(val query: String) {
   def isMatch(s: String) = {
   s.contains(query)
   }
   def getMatchedRDD1(rdd: RDD[String]) = {
   rdd.filter(isMatch) 
   }
   def getMatchedRDD2(rdd: RDD[String]) = {
   val q = query
   rdd.filter(_.contains(q))
   } }
   ```

#### 5.1.4.7 RDD依赖关系

1. RDD血缘关系
   RDD只支持粗粒度转换，即在大量记录上执行的单个操作。将创建RDD的一系列Lineage(血统)记录下来，以便恢复丢失的分区。RDD的Lineage会记录RDD的元数据信息和转换行为，当该RDD的部分数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。

   ```scala
   val fileRDD: RDD[String] = sc.textFile("input/1.txt")
   println(fileRDD.toDebugString)
   println("----------------------")
   val wordRDD: RDD[String] = fileRDD.flatMap(_.split(" "))
   println(wordRDD.toDebugString)
   println("----------------------")
   val mapRDD: RDD[(String, Int)] = wordRDD.map((_,1))
   println(mapRDD.toDebugString)
   println("----------------------")
   val resultRDD: RDD[(String, Int)] = mapRDD.reduceByKey(_+_)
   println(resultRDD.toDebugString)
   resultRDD.collect()
   ```

   

2. RDD依赖关系
   这里所谓的依赖关系，其实就是两个相邻RDD之间的关系

   ```scala
   val sc: SparkContext = new SparkContext(conf)
   val fileRDD: RDD[String] = sc.textFile("input/1.txt")
   println(fileRDD.dependencies)
   println("----------------------")
   val wordRDD: RDD[String] = fileRDD.flatMap(_.split(" "))
   println(wordRDD.dependencies)
   println("----------------------")
   val mapRDD: RDD[(String, Int)] = wordRDD.map((_,1))
   println(mapRDD.dependencies)
   println("----------------------")
   val resultRDD: RDD[(String, Int)] = mapRDD.reduceByKey(_+_)
   println(resultRDD.dependencies)
   resultRDD.collect()
   ```

   

3. RDD窄依赖
   窄依赖表示每一个父(上游)RDD的Partition最多被子(下游)RDD的一个Partition使用，窄依赖我们形象地比喻为独生子女

   ```scala
   class OneToOneDependency[T](rdd: RDD[T]) extends NarrowDependency[T](rdd)
   ```

4. RDD宽依赖
   宽依赖表示同一个父(上游)RDD的Partition被多个子(下游)RDD的Partition依赖，会引起Shufflem,总结:宽依赖我们形象地比喻为多生。

   ```scala
   class ShuffleDependency[K: ClassTag, V: ClassTag, C: ClassTag](
   @transient private val _rdd: RDD[_ <: Product2[K, V]],
   val partitioner: Partitioner,
   val serializer: Serializer = SparkEnv.get.serializer,
   val keyOrdering: Option[Ordering[K]] = None,
   val aggregator: Option[Aggregator[K, V, C]] = None,
   val mapSideCombine: Boolean = false)
   extends Dependency[Product2[K, V]]
   ```

   

5. RDD阶段划分
   DAG(Directed Acyclic Graph)有向无环图是由点和线组成的拓扑图形，该图形具有方向，不会闭环。例如DAG记录了RDD的转换过程和任务的阶段
   ![image-20201225141258897](./images/53.png)

6. RDD阶段划分源码

   ```scala
   try {
   // New stage creation may throw an exception if, for example, jobs are run on 
   a
   // HadoopRDD whose underlying HDFS files have been deleted.
   finalStage = createResultStage(finalRDD, func, partitions, jobId, callSite)
   } catch {
   case e: Exception =>
   logWarning("Creating new stage failed due to exception - job: " + jobId, e)
   listener.jobFailed(e)
   return
   }
   ……
   private def createResultStage(
   rdd: RDD[_],
   func: (TaskContext, Iterator[_]) => _,
   partitions: Array[Int],
   jobId: Int,
   callSite: CallSite): ResultStage = {
   val parents = getOrCreateParentStages(rdd, jobId)
   val id = nextStageId.getAndIncrement()
   val stage = new ResultStage(id, rdd, func, partitions, parents, jobId, callSite)
   stageIdToStage(id) = stage
   updateJobIdStageIdMaps(jobId, stage)
   stage
   }
   ……
   private def getOrCreateParentStages(rdd: RDD[_], firstJobId: Int): List[Stage] 
   = {
   getShuffleDependencies(rdd).map { shuffleDep =>
   getOrCreateShuffleMapStage(shuffleDep, firstJobId)
   }.toList
   }
   ……
   private[scheduler] def getShuffleDependencies(
   rdd: RDD[_]): HashSet[ShuffleDependency[_, _, _]] = {
   val parents = new HashSet[ShuffleDependency[_, _, _]]
   val visited = new HashSet[RDD[_]]
   val waitingForVisit = new Stack[RDD[_]]
   waitingForVisit.push(rdd)
   while (waitingForVisit.nonEmpty) {
   val toVisit = waitingForVisit.pop()
   if (!visited(toVisit)) {
   visited += toVisit
   toVisit.dependencies.foreach {
   case shuffleDep: ShuffleDependency[_, _, _] =>
   parents += shuffleDep
   case dependency =>
   waitingForVisit.push(dependency.rdd)
   } } }
   parents
   }
   ```

   

7. 任务划分
   RDD任务切分中间分为:Application、Job、Stage和Task

   + Application:初始化一个SparkContext即生成一个Application
   + Job:一个Action算子就会生成一个Job
   + Stage:Stage等于宽依赖(ShuffleDependency)的个数加1
   + Task:一个Stage阶段中，最后一个RDD的分区个数就是Task的个数

   注意:Application->Job->Stage->Task每一层都是1对n的关系。

   ![image-20201225143004704](./images/54.png)

8. RDD任务划分源码

   ```scala
   val tasks: Seq[Task[_]] = try {
   stage match {
   case stage: ShuffleMapStage =>
   partitionsToCompute.map { id =>
   val locs = taskIdToLocations(id)
   val part = stage.rdd.partitions(id)
   new ShuffleMapTask(stage.id, stage.latestInfo.attemptId,
   taskBinary, part, locs, stage.latestInfo.taskMetrics, properties, 
   Option(jobId),
   Option(sc.applicationId), sc.applicationAttemptId)
   }
   case stage: ResultStage =>
   partitionsToCompute.map { id =>
   val p: Int = stage.partitions(id)
   val part = stage.rdd.partitions(p)
   val locs = taskIdToLocations(id)
   new ResultTask(stage.id, stage.latestInfo.attemptId,
   taskBinary, part, locs, id, properties, stage.latestInfo.taskMetrics,
   Option(jobId), Option(sc.applicationId), sc.applicationAttemptId)
   } }
   ……
   val partitionsToCompute: Seq[Int] = stage.findMissingPartitions()
   ……
   override def findMissingPartitions(): Seq[Int] = {
   mapOutputTrackerMaster
   .findMissingPartitions(shuffleDep.shuffleId)
   .getOrElse(0 until numPartitions) }
   ```

#### 5.1.4.8 RDD持久化

1. RDD Cache缓存
   RDD通过Cache或者Persist方法将前面的计算结果缓存，默认情况下会把数据缓存在JVM的堆内存中。但是并不是这两个方法被调用时立即缓存，而是触发后面的action算子时，该RDD将会被缓存在计算节点的内存中，并供后面重用

   ```scala
   // cache 操作会增加血缘关系，不改变原有的血缘关系
   println(wordToOneRdd.toDebugString)
   // 数据缓存。
   wordToOneRdd.cache()
   // 可以更改存储级别
   //mapRdd.persist(StorageLevel.MEMORY_AND_DISK_2)
   ```

   ***存储级别***

   ```scala
   object StorageLevel {
   val NONE = new StorageLevel(false, false, false, false)
   val DISK_ONLY = new StorageLevel(true, false, false, false)
   val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
   val MEMORY_ONLY = new StorageLevel(false, true, false, true)
   val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
   val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
   val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
   val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
   val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
   val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
   val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
   val OFF_HEAP = new StorageLevel(true, true, true, false, 1)
   ```

   ![image-20201225173001405](./images/55.png)

   ​	缓存有可能丢失，或者存储于内存的数据由于内存不足儿被删除，RDD的缓存容错机制保证了即使缓存丢失也能保证计算的正确执行。通过基于RDD的一系列转换，丢失的数据会被重算，由于RDD的各个Partition是相对独立的，因此只需要计算丢失的那部分即可，并不需要重算全部Partition

   ​	Spark会自动对一些Shuffle操作的中间数据做持久化操作(比如:reduceByKey)。这样做的目的是为了当一个节点Shuffle失败了避免重新计算整个输入。但是，在实际使用的时候，如果想重用数据，仍然建议调用persist或cache

   

2. RDD CheckPoint 检查点
   所谓的检查点其实就是通过将RDD中间结果写入磁盘。由于血缘关系依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点之后有节点出现问题，可以从检查点开始重做血缘，减少了开销。
   ***注意:***对RDD进行checkpoint操作并不会马上被执行，必须执行Action操作才能触发

   ```scala
   // 设置检查点路径
   sc.setCheckpointDir("./checkpoint1")
   // 创建一个 RDD，读取指定位置文件:hello atguigu atguigu
   val lineRdd: RDD[String] = sc.textFile("input/1.txt")
   // 业务逻辑
   val wordRdd: RDD[String] = lineRdd.flatMap(line => line.split(" "))
   val wordToOneRdd: RDD[(String, Long)] = wordRdd.map {
   word => {
   (word, System.currentTimeMillis())
   } }
   // 增加缓存,避免再重新跑一个 job 做 checkpoint
   wordToOneRdd.cache()
   // 数据检查点：针对 wordToOneRdd 做检查点计算
   wordToOneRdd.checkpoint()
   // 触发执行逻辑
   wordToOneRdd.collect().foreach(println)
   ```

   

3. 缓存和检查点区别

   + Cache缓存只是将数据保存起来，不切断血缘依赖。checkpoint检查点切断血缘依赖。
   + cache缓存的数据通常存储在磁盘、内存等地方，可靠性低。checkpoint的数据通常存储在HDFS等容错、高可用的文件系统，可靠性高。
   + 建议对checkpoint()的RDD使用cache缓存，这样checkpoint的job只需从cache缓存中读取数据即可，否则需要再从头计算一次RDD。



#### 5.1.4.9 RDD分区器

Spark目前支持Hash分区、Range分区和用户自定义分区。Hash分区为当前的默认分区。分区器直接决定了RDD中分区的个数、RDD中每条数据经过Shuffle后进入哪个分区，进而决定了Reduce的个数。

+ 只有Key-Value类型的RDD才有分区器，非Key-Value类型的RDD分区的值是None
+ 每个RDD的分区ID范围:0~(numPartitions-1)，决定这个值是属于哪个分区的。

1. Hash分区:对于给定的key，计算其hashCode,并除以分区个数取余

   ```scala
   class HashPartitioner(partitions: Int) extends Partitioner {
   require(partitions >= 0, s"Number of partitions ($partitions) cannot be 
   negative.")
   def numPartitions: Int = partitions
   def getPartition(key: Any): Int = key match {
   case null => 0
   case _ => Utils.nonNegativeMod(key.hashCode, numPartitions)
   }
   override def equals(other: Any): Boolean = other match {
   case h: HashPartitioner =>
   h.numPartitions == numPartitions
   case _ =>
   false
   }
   override def hashCode: Int = numPartitions
   }
   ```

   

2. Range分区：将一定范围内的数据映射到一个分区中，尽量保证每个分区数据均匀，而且分区间有序

   ```scala
   class RangePartitioner[K : Ordering : ClassTag, V](
   partitions: Int,
   rdd: RDD[_ <: Product2[K, V]],
   private var ascending: Boolean = true)
   extends Partitioner {
   // We allow partitions = 0, which happens when sorting an empty RDD under the 
   default settings.
   require(partitions >= 0, s"Number of partitions cannot be negative but found 
   $partitions.")
   private var ordering = implicitly[Ordering[K]]
   // An array of upper bounds for the first (partitions - 1) partitions
   private var rangeBounds: Array[K] = {
   ...
   }
   def numPartitions: Int = rangeBounds.length + 1
   private var binarySearch: ((Array[K], K) => Int) = 
   CollectionsUtils.makeBinarySearch[K]
   def getPartition(key: Any): Int = {
   val k = key.asInstanceOf[K]
   var partition = 0
   if (rangeBounds.length <= 128) {
   // If we have less than 128 partitions naive search
   while (partition < rangeBounds.length && ordering.gt(k, 
   rangeBounds(partition))) {
   partition += 1
   }
   } else {
   // Determine which binary search method to use only once.
   partition = binarySearch(rangeBounds, k)
   // binarySearch either returns the match location or -[insertion point]-1
   if (partition < 0) {
   partition = -partition-1 }
   if (partition > rangeBounds.length) {
   partition = rangeBounds.length
   } }
   if (ascending) {
   partition
   } else {
   rangeBounds.length - partition
   } }
   override def equals(other: Any): Boolean = other match {
   ...
   }
   override def hashCode(): Int = {
   ...
   }
     @throws(classOf[IOException])
   private def writeObject(out: ObjectOutputStream): Unit = 
   Utils.tryOrIOException {
   ...
   }
   @throws(classOf[IOException])
   private def readObject(in: ObjectInputStream): Unit = Utils.tryOrIOException 
   {
   ...
   } }
   ```

#### 5.1.4.10 RDD文件读取与保存

Spark的数据读取及数据保存可以从两个维度来作区分:文件格式以及文件系统

文件格式分为:text文件、csv文件、sequence文件以及Object文件

文件系统分为:本地文件系统、HDFS、HABSE以及数据库

+ text文件

  ```scala
  // 读取输入文件
  val inputRDD: RDD[String] = sc.textFile("input/1.txt")
  // 保存数据
  inputRDD.saveAsTextFile("output")
  ```

  

+ sequence文件
  SequenceFile文件是Hadoop用来存储二进制形式的key-value对而设计的一种平面文件(Flat File)。在SparkContext中，可以调用sequenceFile\[keyClass,valueClass](path)。

  ```scala
  // 保存数据为 SequenceFile
  dataRDD.saveAsSequenceFile("output")
  // 读取 SequenceFile 文件
  sc.sequenceFile[Int,Int]("output").collect().foreach(println)
  ```

  

+ object对象文件
  对象文件是将对象序列化后保存的文件，采用Java的序列化机制。可以通过objectFile\[T:ClassTag](path)函数接收一个路径，读取对象文件，返回对应的RDD,也可以通过调用saveAsObjectFile()实现对对象文件的输出。因为是序列化所以要指定类型

  ```scala
  // 保存数据
  dataRDD.saveAsObjectFile("output")
  // 读取数据
  sc.objectFile[Int]("output").collect().foreach(println)
  ```

  

##  5.2 累加器

###  5.2.1实现原理

累加器用来把Executor端变量信息聚合到Driver端。在Driver程序中定义的变量，在Executor端的每个Task都会得到这个变量的一份新的副本，每个task更新这些副本的值后，传回Driver端进行merge

### 5.2.2基础编程

#### 5.2.2.1 系统累加器

```scala
val rdd = sc.makeRDD(List(1,2,3,4,5))
// 声明累加器
var sum = sc.longAccumulator("sum");
rdd.foreach(
 num => {
 // 使用累加器
 sum.add(num)
 } )
// 获取累加器的值
println("sum = " + sum.value)
```

#### 5.2.2.2 自定义累加器

```scala
// 自定义累加器
// 1. 继承 AccumulatorV2，并设定泛型
// 2. 重写累加器的抽象方法
class WordCountAccumulator extends AccumulatorV2[String, mutable.Map[String, 
Long]]{
var map : mutable.Map[String, Long] = mutable.Map()
// 累加器是否为初始状态
override def isZero: Boolean = {
 map.isEmpty
}
// 复制累加器
override def copy(): AccumulatorV2[String, mutable.Map[String, Long]] = {
 new WordCountAccumulator
}
// 重置累加器
override def reset(): Unit = {
 map.clear()
}
// 向累加器中增加数据 (In)
override def add(word: String): Unit = {
 // 查询 map 中是否存在相同的单词
 // 如果有相同的单词，那么单词的数量加 1
 // 如果没有相同的单词，那么在 map 中增加这个单词
 map(word) = map.getOrElse(word, 0L) + 1L
}
    // 合并累加器
override def merge(other: AccumulatorV2[String, mutable.Map[String, Long]]): 
Unit = {
 val map1 = map
 val map2 = other.value
 // 两个 Map 的合并
 map = map1.foldLeft(map2)(
 ( innerMap, kv ) => {
 innerMap(kv._1) = innerMap.getOrElse(kv._1, 0L) + kv._2
 innerMap
 }
 ) }
// 返回累加器的结果 （Out）
override def value: mutable.Map[String, Long] = map
}
```

## 5.3 广播变量

### 5.3.1 实现原理

广播变量用来高效地分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个Spark操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表，广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是Spark会为每个任务分别发送。

### 5.3.2 基础编程

```scala
val rdd1 = sc.makeRDD(List( ("a",1), ("b", 2), ("c", 3), ("d", 4) ),4)
val list = List( ("a",4), ("b", 5), ("c", 6), ("d", 7) )
// 声明广播变量
val broadcast: Broadcast[List[(String, Int)]] = sc.broadcast(list)
val resultRDD: RDD[(String, (Int, Int))] = rdd1.map {
 case (key, num) => {
 var num2 = 0
 // 使用广播变量
 for ((k, v) <- broadcast.value) {
 if (k == key) {
 num2 = v
 }
 }
 (key, (num, num2))
 } }
```

#  6. Spark案例实操

![image-20201227092653276](.\images\56.png) 

上面的数据图是从数据文件中截取的一部分内容，表示为电商网站的用户行为数据，主要包含用户的4种行为:搜索、点击、下单、支付。数据规则如下:

+ 数据文件中每行数据采用下划线分割数据
+ 每一行数据表示用户的一次行为，这个行为只能是4种行为的一种
+ 如果搜索关键字为null,表示不是搜索数据
+ 如果点击的品类ID和产品ID为-1，表示数据不是点击数据
+ 针对于下单行为，一次可以下单多个商品，所以品类ID和产品ID可以是多个，id之间采用逗号分隔，如果本次不是下单行为，则数据采用Null表示
+ 支付行为和下单行为类似

***详细字段说明***

![image-20201227094527036](.\images\57.png)

![image-20201227094611491](.\images\58.png)

***样例类***

```scala
//用户访问动作表
case class UserVisitAction(
 date: String,//用户点击行为的日期
 user_id: Long,//用户的 ID
 session_id: String,//Session 的 ID
 page_id: Long,//某个页面的 ID
 action_time: String,//动作的时间点
 search_keyword: String,//用户搜索的关键词
 click_category_id: Long,//某一个商品品类的 ID
 click_product_id: Long,//某一个商品的 ID
 order_category_ids: String,//一次订单中所有品类的 ID 集合
 order_product_ids: String,//一次订单中所有商品的 ID 集合
 pay_category_ids: String,//一次支付中所有品类的 ID 集合
 pay_product_ids: String,//一次支付中所有商品的 ID 集合
 city_id: Long
)//城市 id
```

## 6.1 需求1：Top10热门品类

![image-20201227094812831](.\images\59.png)

### 6.1.1 需求说明

品类是指产品的分类，大型电商网站品类分多级，本项目中品类只有1级，不同的公司可能对热门的定义不一样。我们按照每个品类的点击、下单、支付的量来统计热门品类。

![image-20201227095045227](.\images\60.png)

![image-20201227095117292](.\images\61.png)



例如，综合排名 = 点击数 * 20% + 下单数 * 30 % + 支付数 * 50%

本项目需求优化为:先按照点击数排名，靠前的就排名高；如果点击数相同，再比较下单数；下单数相同，就比较支付数

### 6.1.2 实现方案1

#### 6.1.2.1 需求分析

分别统计每个品类点击的次数，下单的次数和支付的次数：

(品类,点击总数) (品类,下单总数) (品类,支付总数)

#### 6.1.2.2 需求实现

```scala
package net.codeshow.spark.core.req

import org.apache.spark.{SparkConf, SparkContext}

object Spark01_Req_HotCategoryTop10Analysis {
  def main(args: Array[String]): Unit = {
    //    todo top10热门品类
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis")
    val sc = new SparkContext(sparkConf)
    //    1.读取原始日志数据
    val actionRDD = sc.textFile("datas/user_visit_action.txt")
    //    2.统计品类的点击数量:(品类ID,点击数量)
    val clickActionRDD = actionRDD.filter(
      action => {
        val datas = action.split("_")
        datas(6) != "-1"
      }
    )
    val clickCountRDD = clickActionRDD.map(
      action => {
        val datas = action.split("_")
        (datas(6), 1)
      }
    ).reduceByKey(_ + _)

    //    3.统计品类的下单数量:(品类ID,下单数量)
    val orderActionRDD = actionRDD.filter(
      action => {
        val datas = action.split("_")
        datas(8) != "null"
      }
    )

    val orderCountRDD = orderActionRDD.flatMap(
      action => {
        val datas = action.split("_")
        val cid = datas(8)
        val cids = cid.split(",")
        cids.map(id => (id, 1))
      }
    ).reduceByKey(_ + _)

    //    4.统计品类的支付数量:(品类ID,支付数量)
    val payActionRDD = actionRDD.filter(
      action => {
        val datas = action.split("_")
        datas(10) != "null"
      }
    )

    val payCountRDD = payActionRDD.flatMap(
      action => {
        val datas = action.split("_")
        val cid = datas(10)
        val cids = cid.split(",")
        cids.map(id => (id, 1))
      }
    ).reduceByKey(_ + _)

    //    5.将品类进行排序，并且取前10名
    //点击数量排序，下单数量排序，支付数量排序
    //    元组排序:先比较第一个，如果第一个相同，再比较第二个，如果第二个相同，再比较第三个，以此类推
    //    (品类ID,(点击数量,下单数量,支付数量))

    val cogroupRDD = clickCountRDD.cogroup(orderCountRDD, payCountRDD)
    val analysisRDD = cogroupRDD.mapValues {
      case (clickIter, orderIter, payIter) => {
        var clickCnt = 0
        val iter1 = clickIter.iterator
        if (iter1.hasNext) {
          clickCnt = iter1.next()
        }
        var orderCnt = 0
        val iter2 = orderIter.iterator
        if (iter2.hasNext) {
          orderCnt = iter2.next()
        }
        var payCnt = 0
        val iter3 = payIter.iterator
        if (iter3.hasNext) {
          payCnt = iter3.next()
        }
        (clickCnt, orderCnt, payCnt)

      }
    }

    val resultRDD = analysisRDD.sortBy(_._2, ascending = false).take(10)

    //    6.将结果采集到控制台打印出来
    resultRDD.foreach(println)
    sc.stop()
  }
}

```

### 6.1.3 实现方案2

#### 6.1.3.1 需求分析

一次性统计每个品类点击的次数、下单的次数和支付的次数

(品类,(点击总数,下单总数,支付总数))

#### 6.1.3.2 需求实现

```scala
package net.codeshow.spark.core.req

import org.apache.spark.{SparkConf, SparkContext}

object Spark02_Req_HotCategoryTop10Analysis {
  def main(args: Array[String]): Unit = {
    //    todo top10热门品类
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis")
    val sc = new SparkContext(sparkConf)
    //    1.读取原始日志数据
    val actionRDD = sc.textFile("datas/user_visit_action.txt")
    //    actionRDD重复使用
    actionRDD.cache()
    //    2.统计品类的点击数量:(品类ID,点击数量)
    val clickActionRDD = actionRDD.filter(
      action => {
        val datas = action.split("_")
        datas(6) != "-1"
      }
    )

    val clickCountRDD = clickActionRDD.map(
      action => {
        val datas = action.split("_")
        (datas(6), 1)
      }
    ).reduceByKey(_ + _)

    //    3.统计品类的下单数量:(品类ID,下单数量)
    val orderActionRDD = actionRDD.filter(
      action => {
        val datas = action.split("_")
        datas(8) != "null"
      }
    )

    val orderCountRDD = orderActionRDD.flatMap(
      action => {
        val datas = action.split("_")
        val cid = datas(8)
        val cids = cid.split(",")
        cids.map(id => (id, 1))
      }
    ).reduceByKey(_ + _)

    //    4.统计品类的支付数量:(品类ID,支付数量)
    val payActionRDD = actionRDD.filter(
      action => {
        val datas = action.split("_")
        datas(10) != "null"
      }
    )

    val payCountRDD = payActionRDD.flatMap(
      action => {
        val datas = action.split("_")
        val cid = datas(10)
        val cids = cid.split(",")
        cids.map(id => (id, 1))
      }
    ).reduceByKey(_ + _)
    //    5.将品类进行排序，并且取前10名
    //点击数量排序，下单数量排序，支付数量排序
    //    元组排序:先比较第一个，如果第一个相同，再比较第二个，如果第二个相同，再比较第三个，以此类推
    //    (品类ID,(点击数量,下单数量,支付数量))
    var rdd1 = clickCountRDD.map {
      case (cid, cnt) => {
        (cid, (cnt, 0, 0))
      }
    }
    var rdd2 = orderCountRDD.map {
      case (cid, cnt) => {
        (cid, (0, cnt, 0))
      }
    }
    var rdd3 = payCountRDD.map {
      case (cid, cnt) => {
        (cid, (0, 0, cnt))
      }
    }

    //    将三个数据源合并在一起，统一进行聚合计算
    val sourceRDD = rdd1.union(rdd2).union(rdd3)
    val analysisRDD = sourceRDD.reduceByKey(
      (t1, t2) => {
        (t1._1 + t2._1, t1._2 + t2._2, t1._3 + t2._3)
      }
    )
    val resultRDD = analysisRDD.sortBy(_._2, ascending = false).take(10)
    //    6.将结果采集到控制台打印出来
    resultRDD.foreach(println)
    sc.stop()
  }
}
```

### 6.1.4 实现方案3

#### 6.1.4.1 需求分析

使用累加器的方式聚合数据

#### 6.1.4.2 需求实现

```scala
package net.codeshow.spark.core.req

import org.apache.spark.util.AccumulatorV2
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable

object Spark04_Req_HotCategoryTop10Analysis {
  def main(args: Array[String]): Unit = {
    //    todo top10热门品类
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis")
    val sc = new SparkContext(sparkConf)
    //    1.读取原始日志数据
    val actionRDD = sc.textFile("datas/user_visit_action.txt")
    val acc = new HotCategoryAccumulator
    sc.register(acc, "hotCategory")
    //   2.将数据转换结构
    //    点击的场景:(品类ID,(1,0,0))
    //    下单的场景:(品类ID,(0,1,0))
    //    支付的场景:(品类ID,(0,0,1))
    actionRDD.foreach {
      action => {
        val datas = action.split("_")
        if (datas(6) != "-1") {
          //          点击的场景
          acc.add((datas(6), "click"))
        } else if (datas(8) != "null") {
          //          下单的场景
          val ids = datas(8).split(",")
          ids.foreach(
            id => {
              acc.add(id, "order")
            }
          )
        } else if (datas(10) != "null") {
          //          支付的场景
          val ids = datas(10).split(",")
          ids.foreach(
            id => {
              acc.add(id, "pay")
            }
          )
        }
      }
    }
    val accVal = acc.value
    val categories = accVal.values
    val sort = categories.toList.sortWith(
      (left, right) => {
        if (left.clickCnt > right.clickCnt) {
          true
        } else if (left.clickCnt == right.clickCnt) {
          if (left.orderCnt > right.orderCnt) {
            true
          } else if (left.orderCnt == right.orderCnt) {
            left.payCnt > right.payCnt
          }
          else {
            false
          }
        }
        else {
          false
        }
      }
    )
    //    5.将结果采集到控制台打印出来
    sort.take(10).foreach(println)
    sc.stop()
  }
  case class HotCategory(cid: String, var clickCnt: Int, var orderCnt: Int, var payCnt: Int)
  /*
  自定义累加器
  1.继承AccumulatorV2，定义泛型
  2.重写方法
   */
  class HotCategoryAccumulator extends AccumulatorV2[(String, String), mutable.Map[String, HotCategory]] {
    private val hcMap = mutable.Map[String, HotCategory]()
    override def isZero: Boolean = {
      hcMap.isEmpty
    }
    override def copy(): AccumulatorV2[(String, String), mutable.Map[String, HotCategory]] = {
      new HotCategoryAccumulator()
    }
    override def reset(): Unit = {
      hcMap.clear()
    }
    override def add(v: (String, String)): Unit = {
      val cid = v._1
      val actionType = v._2
      val category = hcMap.getOrElse(cid, HotCategory(cid, 0, 0, 0))
      if (actionType == "click") {
        category.clickCnt += 1
      } else if (actionType == "order") {
        category.orderCnt += 1
      } else if (actionType == "pay") {
        category.payCnt += 1
      }
      hcMap.update(cid, category)
    }
    override def merge(other: AccumulatorV2[(String, String), mutable.Map[String, HotCategory]]): Unit = {
      val map1 = this.hcMap
      val map2 = other.value
      map2.foreach {
        case (cid, hc) => {
          val category = map1.getOrElse(cid, HotCategory(cid, 0, 0, 0))
          category.clickCnt += hc.clickCnt
          category.orderCnt += hc.orderCnt
          category.payCnt += hc.payCnt
          map1.update(cid, category)
        }
      }
    }
    override def value: mutable.Map[String, HotCategory] = hcMap
  }
}
```

## 6.2 需求2：Top10热门品类中每个品类的Top10活跃Session统计

 ### 6.2.1 需求说明

在需求1的基础上，增加每个品类用户session的点击统计

### 6.2.2 需求分析



### 6.2.3 功能实现

```scala
package net.codeshow.spark.core.req

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Spark05_Req2_HotCategoryTop10Analysis {
  def main(args: Array[String]): Unit = {
    //    todo top10热门品类
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis")
    val sc = new SparkContext(sparkConf)
    //    1.读取原始日志数据
    val actionRDD = sc.textFile("datas/user_visit_action.txt")
    actionRDD.cache()
    val top10Ids = top10Category(actionRDD)
    //1.过滤原始数据，保留点击和前10品类ID
    val filterActionRDD = actionRDD.filter(
      action => {
        val datas = action.split("_")
        if (datas(6) != "-1") {
          top10Ids.contains(datas(6))
        } else {
          false
        }
      }
    )
    //    2.根据品类ID和sessionid进行点击量的统计
    val reduceRDD = filterActionRDD.map(
      action => {
        val datas = action.split("_")
        ((datas(6), datas(2)), 1)

      }
    ).reduceByKey(_ + _)

    //    3.将统计结果转换结构
    //    ((品类ID,sessionId),sum) => (品类ID,(sessionId,sum))
    val mapRDD = reduceRDD.map {
      case ((cid, sid), sum) => {
        (cid, (sid, sum))
      }
    }
    //    4.相同的品类进行分组
    val groupRDD = mapRDD.groupByKey()

    //    5.将分组后的数据进行点击量的排序，取前10名
    val resultRDD = groupRDD.mapValues(
      iter => {
        iter.toList.sortBy(_._2)(Ordering.Int.reverse).take(10)
      }
    )

    resultRDD.collect().foreach(println)

    sc.stop()
  }

  def top10Category(actionRDD: RDD[String]) = {
    val flatRDD = actionRDD.flatMap {
      action => {
        val datas = action.split("_")
        if (datas(6) != "-1") {
          //          点击的场景
          List((datas(6), (1, 0, 0)))
        } else if (datas(8) != "null") {
          //          下单的场景
          val ids = datas(8).split(",")
          ids.map(id => (id, (0, 1, 0)))
        } else if (datas(10) != "null") {
          //          支付的场景
          val ids = datas(10).split(",")
          ids.map(id => (id, (0, 0, 1)))
        } else {
          Nil
        }
      }
    }
    val analysisRDD = flatRDD.reduceByKey(
      (t1, t2) => {
        (t1._1 + t2._1, t1._2 + t2._2, t1._3 + t2._3)
      }
    )
    analysisRDD.sortBy(_._2, ascending = false).take(10).map(_._1)
  }
}
```

## 6.3 需求3：页面单跳转换率统计

### 6.3.1 需求说明

1. 页面单跳转化率
   计算页面单跳转化率，什么是页面单跳转化率，比如一个用户在一次Session过程中访问的页面路径3,5,7,9,10,21,那么页面3跳到页面5叫一次单跳,7-9也叫一次单跳，那么单跳转化率就是要统计页面点击的概率。
   比如:计算3-5的单跳转化率，先获取符合条件的Session对于页面3的访问次数(PV)为A,然后获取符合条件的Session中访问了页面3又紧接着访问了页面5的次数为B,那么B/A就是3-5的页面单跳转化率
   ![image-20201227174556750](.\images\62.png)

   ​	

2. 统计页面单跳转化率意义
        产品经理和运营总监，可以根据这个指标去尝试分析整个网站、产品、各个页面的表现怎么样，是不是需要去优化产品的布局，吸引用户最终进入最后的支付页面。
      	数据分析师可以根据此数据做更深一步的计算和分析
      	企业管理层可以看到整个公司的网站，各个页面之间的跳转的表现如何，可以适当地调整公司的经营战略。



### 6.3.2  需求分析

### 6.3.3 功能实现

















































