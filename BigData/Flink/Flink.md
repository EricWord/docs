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

