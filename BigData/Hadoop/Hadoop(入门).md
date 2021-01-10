[TOC]

# 1. 大数据概论

## 1.1 大数据概念

![image-20210103121848064](.\images\1.png)

## 1.2 大数据特点（4V）

![image-20210103122026347](.\images\2.png)

![image-20210103122052597](.\images\3.png)

![image-20210103122123337](.\images\4.png)

![image-20210103122151909](.\images\5.png)

## 1.3 大数据应用场景

![image-20210103122258347](.\images\6.png)

![image-20210103122322610](.\images\7.png)

![image-20210103122348233](.\images\8.png)

![image-20210103122413354](E:\Projects\docs\BigData\Hadoop\images\9.png)

![image-20210103122433554](.\images\10.png)

![image-20210103122502208](.\images\11.png)

## 1.4 大数据发展前景

![image-20210103122536672](.\images\12.png)

![image-20210103122606104](.\images\13.png)

![image-20210103122634487](.\images\14.png)

![image-20210103122657982](.\images\15.png)

![image-20210103122727274](.\images\16.png)

## 1.5 大数据部门业务流程分析

![image-20210103122827360](E:\Projects\docs\BigData\Hadoop\images\17.png)

## 1.6 大数据部门组织结构（重点）

![image-20210103125211341](.\images\18.png)

# 2. 从Hadoop框架讨论大数据生态

## 2.1 Hadoop是什么

![image-20210103125311067](.\images\19.png)

## 2.2 Hadoop发展历史

![image-20210103133030761](.\images\20.png)

![image-20210103133114052](.\images\21.png)

## 2.3 Hadoop三大发行版本

Hadoop三大发行版本：Apache、Cloudera、Hortonworks。

+ Apache版本最原始（最基础）的版本，对于入门学习最好。

+ Cloudera在大型互联网企业中用的较多。

+ Hortonworks文档较好

1. Apache Hadoop
   官网地址：http://hadoop.apache.org/releases.html

   下载地址：https://archive.apache.org/dist/hadoop/common/

2. Cloudera Hadoop
   官网地址：https://www.cloudera.com/downloads/cdh/5-10-0.html

   下载地址：http://archive-primary.cloudera.com/cdh5/cdh/5/

   + 2008年成立的Cloudera是最早将Hadoop商用的公司，为合作伙伴提供Hadoop的商用解决方案，主要是包括支持、咨询服务、培训。
   + 2009年Hadoop的创始人Doug Cutting也加盟Cloudera公司。Cloudera产品主要为CDH，Cloudera Manager，Cloudera Support
   + CDH是Cloudera的Hadoop发行版，完全开源，比Apache Hadoop在兼容性，安全性，稳定性上有所增强
   + Cloudera Manager是集群的软件分发及管理监控平台，可以在几个小时内部署好一个Hadoop集群，并对集群的节点及服务进行实时监控。Cloudera Support即是对Hadoop的技术支持。
   + Cloudera的标价为每年每个节点4000美元。Cloudera开发并贡献了可实时处理大数据的Impala项目。

3. Hortonworks Hadoop
   官网地址：https://hortonworks.com/products/data-center/hdp/

   下载地址：https://hortonworks.com/downloads/#data-platform

   + 2011年成立的Hortonworks是雅虎与硅谷风投公司Benchmark Capital合资组建
   + 公司成立之初就吸纳了大约25名至30名专门研究Hadoop的雅虎工程师，上述工程师均在2005年开始协助雅虎开发Hadoop，贡献了Hadoop80%的代码
   + 雅虎工程副总裁、雅虎Hadoop开发团队负责人Eric Baldeschwieler出任Hortonworks的首席执行官
   + Hortonworks的主打产品是Hortonworks Data Platform（HDP），也同样是100%开源的产品，HDP除常见的项目外还包括了Ambari，一款开源的安装和管理系统。
   + HCatalog，一个元数据管理系统，HCatalog现已集成到Facebook开源的Hive中。Hortonworks的Stinger开创性的极大的优化了Hive项目。Hortonworks为入门提供了一个非常好的，易于使用的沙盒。
   + Hortonworks开发了很多增强特性并提交至核心主干，这使得Apache Hadoop能够在包括Window Server和Windows Azure在内的Microsoft Windows平台上本地运行。定价以集群为基础，每10个节点每年为12500美元。

## 2.4 Hadoop的优势（4高）

![image-20210103155850425](.\images\22.png)

## 2.5  Hadoop组成（面试重点）

![image-20210103155934490](.\images\23.png)

### 2.5.1 HDFS架构概述

HDFS（Hadoop Distributed File System）的架构概述

![image-20210103160204990](.\images\24.png)

### 2.5.2 YARN架构概述

![image-20210103160301593](.\images\25.png)

### 2.5.3 MapReduce架构概述

MapReduce将计算过程分为两个阶段：Map和Reduce

+ Map阶段并行处理输入数据
+ Reduce阶段对Map结果进行汇总

![image-20210103160452874](.\images\26.png)

## 2.6 大数据技术生态体系

![image-20210103160554664](.\images\27.png)

图中涉及的技术名词解释如下:

+ Sqoop：Sqoop是一款开源的工具，主要用于在Hadoop、Hive与传统的数据库(MySql)间进行数据的传递，可以将一个关系型数据库（例如 ：MySQL，Oracle
+ Flume：Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。
+ Kafka：Kafka是一种高吞吐量的分布式发布订阅消息系统，有如下特性：
  + 通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能
  + 高吞吐量：即使是非常普通的硬件Kafka也可以支持每秒数百万的消息
  + 支持通过Kafka服务器和消费机集群来分区消息
  + 支持Hadoop并行数据加载
+ Storm：Storm用于“连续计算”，对数据流做连续查询，在计算时就将结果以流的形式输出给用户
+ Spark：Spark是当前最流行的开源大数据内存计算框架。可以基于Hadoop上存储的大数据进行计算
+ Oozie：Oozie是一个管理Hdoop作业（job）的工作流程调度管理系统
+ Hbase：HBase是一个分布式的、面向列的开源数据库。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库
+ Hive：Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的SQL查询功能，可以将SQL语句转换为MapReduce任务进行运行。 其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析
+ R语言：R是用于统计分析、绘图的语言和操作环境。R是属于GNU系统的一个自由、免费、源代码开放的软件，它是一个用于统计计算和统计制图的优秀工具
+ Mahout：Apache Mahout是个可扩展的机器学习和数据挖掘库
+ ZooKeeper：Zookeeper是Google的Chubby一个开源的实现。它是一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、 分布式同步、组服务等。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

## 2.7 推荐系统框架图

![image-20210103161012085](.\images\28.png)



# 3.Hadoop运行环境搭建

## 3.1 虚拟机环境准备

1.	克隆虚拟机		

2.	修改克隆虚拟机的静态IP

3.	修改主机名

4.	关闭防火墙

5.	创建atguigu用户

6. 配置atguigu用户具有root权限

​            7．在/opt目录下创建文件夹



+ 在/opt目录下创建module、software文件夹
+ 在/opt目录下创建module、software文件夹

## 3.2 安装JDK

1. 卸载现有JDK

   + 查询是否安装Java软件

     ```shell
     rpm -qa | grep java
     ```

   + 如果安装的版本低于1.7，卸载该JDK：

     ```shell
     sudo rpm -e 软件包
     ```

   + 查看JDK安装路径

     ```shell
     which java
     ```

2. 上传jdk到linux

3. 解压JDK到/opt/module目录下

   ```shell
   tar -zxvf jdk-8u144-linux-x64.tar.gz -C /opt/module/
   ```

4. 配置JDK环境变量

   ```shell
   sudo vi /etc/profile
   #在文件末尾追加如下内容
   #JAVA_HOME
   export JAVA_HOME=/opt/module/jdk1.8.0_144
   export PATH=$PATH:$JAVA_HOME/bin
   ```

5. 让修改后的文件生效

   ```shell
   source /etc/profile
   ```

6. 测试JDK是否安装成功

   ```shell
   java -version
   ```

   如果上述命令不可用，重启下机器

## 3.3 安装Hadoop

1. Hadoop下载地址
   https://archive.apache.org/dist/hadoop/common/stable/

2. 上传安装文件到linux

3. 解压安装文件到/opt/module下面

4. 将Hadoop添加到环境变量

   ```shell
   sudo vi /etc/profile
   #在文件末尾追加以下内容
   ##HADOOP_HOME
   export HADOOP_HOME=/opt/module/hadoop-2.7.2
   export PATH=$PATH:$HADOOP_HOME/bin
   export PATH=$PATH:$HADOOP_HOME/sbin
   ```

   执行下面命令让配置文件生效

   ```shell
   source /etc/profile
   ```

5. 测试是否安装成功

   ```shell
   hadoop version
   ```

   如果上述命令不好使，重启下机器

## 3.4 Hadoop目录结构

1. 查看Hadoop目录结构

   ```shell
   总用量 52
   drwxr-xr-x. 2 atguigu atguigu  4096 5月  22 2017 bin
   drwxr-xr-x. 3 atguigu atguigu  4096 5月  22 2017 etc
   drwxr-xr-x. 2 atguigu atguigu  4096 5月  22 2017 include
   drwxr-xr-x. 3 atguigu atguigu  4096 5月  22 2017 lib
   drwxr-xr-x. 2 atguigu atguigu  4096 5月  22 2017 libexec
   -rw-r--r--. 1 atguigu atguigu 15429 5月  22 2017 LICENSE.txt
   -rw-r--r--. 1 atguigu atguigu   101 5月  22 2017 NOTICE.txt
   -rw-r--r--. 1 atguigu atguigu  1366 5月  22 2017 README.txt
   drwxr-xr-x. 2 atguigu atguigu  4096 5月  22 2017 sbin
   drwxr-xr-x. 4 atguigu atguigu  4096 5月  22 2017 share
   ```

2. 重要目录

   + bin目录：存放对Hadoop相关服务（HDFS,YARN）进行操作的脚本
   + etc目录：Hadoop的配置文件目录，存放Hadoop的配置文件
   + lib目录：存放Hadoop的本地库（对数据进行压缩解压缩功能）
   + sbin目录：存放启动或停止Hadoop相关服务的脚本
   + share目录：存放Hadoop的依赖jar包、文档、和官方案例

# 4.Hadoop运行模式

