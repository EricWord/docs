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

Spark Executor是集群中工作节点(Worker)中的一个JVM 进程，负责在Spark作业中运行具体任务(Task),彼此之间相互独立。Spark应用启动时，Executor节点被同时启动，并且始终伴随着整个Spark应用的声明周期而存在。如果有Executor节点发生了故障或崩溃，Spark应用也可以继续执行，会将出错节点上的任务调度到其他Executor节点上继续运行

Executor有两个核心功能:

+ 负责运行组成Spark应用的任务，并将结果返回给驱动器进程
+ 它们通过自身的块管理器(Block Manager)为用户程序中要求缓存的RDD提供内存式存储。RDD是直接缓存在Executor进程内的，因此任务可以在运行时充分利用缓存数据加速运算

### 4.2.3 Master & Worker

Spark集群的独立部署环境中，不需要依赖其他的资源调度框架，自身就实现了资源调度的功能，所以环境中还有其他两个核心组件：Master和Worker,这里的Master是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于Yarn环境中的RM，而Worker呢，也是进程，一个Worker运行在集群中的一台服务器上，由Master分配资源对数据进行并行的处理和计算，类似于Yarn环境中NM

### 4.2.4 ApplicationMaster

Hadoop用户向YARN集群提交应用程序时，提交程序中应该包含ApplicationMaster,用于向资源调度器申请执行任务的资源容器Container,运行用户自己的程序任务job,监控整个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。

说的简单点就是ResourceManager(资源)和Driver(计算)之间的解耦合靠的就是ApplicationMaster.

## 4.3核心概念



















































































