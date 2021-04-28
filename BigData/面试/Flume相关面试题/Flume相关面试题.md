[TOC]

# 1.sink到HDFS时小文件如何处理

![image-20210416203646204](images/image-20210416203646204.png)

将下面三行注释掉：

```bash
a1.sinks.k1.hdfs.round=true
a1.sinks.k1.hdfs.roundValue=10
a1.sinks.k1.hdfs.roundUnit=minute
```

参数含义：

round
默认值：false，是否启用时间上的”舍弃”，类似于”四舍五入”，如果启用，则会影响除了%t的其他所有时间表达式；

roundValue
默认值：1，时间上进行“舍弃”的值；

roundUnit

默认值：seconds，时间上进行”舍弃”的单位，包含：second,minute,hour


新增下面的内容：

```bash
a1.sinks.k1.hdfs.rollSize=0
a1.sinks.k1.hdfs.rollCount=0
```

参数含义：

rollSize
默认值：1024，当临时文件达到该大小（单位：bytes）时，滚动成目标文件。如果设置成0，则表示不根据临时文件大小来滚动文件。

rollCount
默认值：10，当events数据达到该数量时候，将临时文件滚动成目标文件，如果设置成0，则表示不根据events数据来滚动文件。

还有一个可能的原因是文件因为所在块的复制而滚动，可以设置如下参数：

```bash
a1.sinks.k1.hdfs.minBlockReplicas=1
```

参数含义：

 Specify minimum number of replicas per HDFS block. If not specified, it comes from the default Hadoop config in the classpath.

没有默认值







# 2. sink到HDFS的时候如何按照日期生成对应的目录

配置如下:

```bash
a1.sinks.k1.hdfs.path = hdfs://nameservice1/tmp/flume/jbw/%y-%m-%d
```

