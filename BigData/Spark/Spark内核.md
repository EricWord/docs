[TOC]

# 1. Spark内核概述

Spark内核泛指Spark的核心运行机制，包括Spark核心组件的运行机制、Spark任务调度机制、Spark内存管理机制、Spark核心功能的运行原理，熟练掌握Spark内核原理，能够帮助我们更好地完成Spark代码设计，并能够帮助我们准确锁定项目运行过程中出现的问题的症结所在。

## 1.1 Spark核心组件回顾

### 1.1.1 Driver

Spark驱动器节点，用于执行Spark任务中的main方法，负责实际代码的执行工作。Driver在Spark作业执行时主要负责:

1. 将用户程序转化为作业(Job)
2. 在Executor之间调度任务(Task)
3. 跟踪Executor的执行情况
4. 通过UI展示查询运行情况

### 1.1.2 Executor

Spark Executor对象是负责在Spark作业中运行具体任务，任务彼此之间相互独立。Spark应用启动时，ExecutorBackend节点被同时启动，并且始终伴随着整个Spark应用的声明周期而存在。如果有ExecutorBackend节点发生了故障或崩溃，Spark应用也可以继续执行，会将出错节点上的任务调度到其他Executor节点上继续运行。

Executor有两个核心功能:

1. 负责运行组成Spark应用的任务，并将结果返回给驱动器(Driver)
2. 它们通过自身的块管理器(Block Manager)为用户程序中要求缓存的RDD提供内存式存储。RDD是直接缓存在Executor进程内的，因此任务可以在运行时充分利用缓存数据加速运算



## 1.2 Spark通用运行流程概述

![image-20201228123117716](./images/63.png)

上图为Spark通用运行流程图，体现了基本的Spark应用程序在部署中的基本提交流程。

这个流程是按照如下的核心步骤进行工作的:

1. 任务提交后，都会先启动Driver程序
2. 随后Driver向集群管理器注册应用程序
3. 之后集群管理器根据此任务的配置文件分配Executor并启动
4. Driver开始执行main函数，Spark查询为懒执行，当执行到Actioin算子时开始反向推算，根据宽依赖进行stage的划分，随后每一个Stage对应一个TaskSet，TaskSet中有多个Task,查找可用资源Executor进行调度
5. 根据本地化原则，Task会被分发到指定的Executor去执行，在任务执行的过程中，Executor也会不断与Driver进行通信，报告任务完成情况。



# 2.Spark部署模式

Spark支持多种集群管理器(Cluster Manager),分别为:

1. Standalone:独立模式，Spark原生的简单集群管理器自带完整的服务，可单独部署到一个集群中，无需依赖任何其他资源管理系统，使用Standalone可以很方便地搭建一个集群

2. Hadoop Yarn:统一的资源管理机制，在上面可以运行多套计算框架，如MR，Storm等。根据Driver在集群中的位置不同，分为yarn client(集群外)和yarn cluster(集群内部)

3. Apache Mesos:一个强大的分布式资源管理框架，它允许多种不同的框架部署在其上，包括yarn

4. K8S:容器式部署环境

   实际上，除了上述这些通用的集群管理器外，Spark内部也提供了方便用户测试和学习的本地集群部署模式和Windows环境。由于在实际工作环境下使用的绝大多数的集群管理器是Hadoop Yarn,因此我们关注的重点是Hadoop Yarn模式下的Spark集群部署

## 2.1 Yarn模式运行机制

### 2.1.1 Yarn Cluster模式

1.  执行脚本提交任务，实际上是启动一个SparkSubmit的JVM进程
2. SparkSubmit类中的main方法反射调用YarnClusterApplication的main方法
3. YarnClusterApplication创建Yarn客户端，然后向Yarn服务器发送执行指令:bin/java ApplicationMaster
4. Yarn框架收到指令后会在指定的NM(NodeManager)中启动ApplicationMaster
5. ApplicationMaster启动Driver线程，执行用户的作业
6. AM(ApplicationMaster)向RM(ResourceManager)注册，申请资源
7. 获取资源后AM向NM发送指令:bin/java YarnCoarseGrainedExecutorBackend
8. CoarseGrainedExecutorBackend进程会接收消息，跟Driver通信，注册已经启动的Executor,然后启动计算对象Executor等待接收任务
9. Driver线程继续执行完成作业的调度和任务的执行
10. Driver分配任务并监控任务的执行

***注意:*** SparkSubmit、ApplicationMaster和CoarseGrainedExecutorBackend是独立的进程，Driver是独立的线程，Executor和YarnClusterApplication是对象

![image-20201228151044645](./images/64.png)

### 2.1.2 Yarn Client模式

1. 执行脚本提交任务，实际是启动一个SparkSubmit的JVM进程

2. SparkSubmit类中的main方法反射调用用户代码的main方法

3. 启动Driver线程，执行用户的作业，并创建ScheduleBackend

4. YarnClientSchedulerBackend向RM发送指令:bin/java ExecutorLauncher

5. Yarn框架收到指令后会在指定的NM中启动ExecutorLauncher(实际上还是调用ApplicationMaster的main方法)

   ```scala
   object ExecutorLauncher {
   def main(args: Array[String]): Unit = {
   ApplicationMaster.main(args)
   } }
   ```

   

6. AM向RM注册，申请资源

7. 获取资源后AM向NM发送指令:bin/java CoarseGrainedExecutorBackend

8. CoarseGrainedExecutorBackend进程会接收消息，跟Driver通信，注册已经启动的Executor，然后启动计算对象Executor等待接收任务

9. Driver分配任务并监控任务的执行

***注意:*** SparkSubmit、ApplicationMaster和CoarseGrainedExecutorBackend是独立的进程，Executor和Driver是对象
![image-20201228151720948](./images/65.png)

## 2.2 Standalone模式运行机制

Standalone集群有2个重要组成部分，分别是:

1. Master(RM):是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责
2. Worker(NM):是一个进程，一个Worker运行在集群的一台服务器上，主要负责两个职责，一个是自己的内存存储RDD的某个或某些partiton,另一个是启动其他进程和线程(Executor)，对RDD上的partition进行并行的处理和计算

### 2.2.1 Standalone Cluster模式

![image-20201228152428266](./images/66.png)

在Standalone Cluster模式下，任务提交后，Master会找到一个Worker启动Driver。Driver启动后向Master注册应用程序，Master根据submit脚本的资源需求找到内部资源至少可以启动一个Executor的所有Worker,然后在这些Worker之间分配Executor，Worker上的Executor启动后会向Driver反向注册，所有的



















