[TOC]

# 1.ClickHouse概述

## 1.1 什么是ClickHouse

![image-20210501222322040](images/image-20210501222322040.png)

## 1.2 什么是列式存储

![image-20210501222755308](images/image-20210501222755308.png)

![image-20210501222831191](images/image-20210501222831191.png)

## 1.3 安装前的准备

### 1.3.1 Centos取消打开文件数限制

![image-20210501223032531](images/image-20210501223032531.png)

![image-20210501224521534](images/image-20210501224521534.png)

# 2. 安装

## 2.1 网址

![image-20210501224625107](images/image-20210501224625107.png)

![image-20210501224942985](images/image-20210501224942985.png)

## 2.2 单机模式

注意：使用root用户操作

![image-20210501225442864](images/image-20210501225442864.png)

![image-20210501225449929](images/image-20210501225449929.png)

![image-20210501225755253](images/image-20210501225755253.png)

![image-20210501225900392](images/image-20210501225900392.png)

![image-20210501225939037](images/image-20210501225939037.png)

![image-20210501230443564](images/image-20210501230443564.png)



## 2.3 分布式集群安装

### 2.3.1 分发配置文件

![image-20210502163319230](images/image-20210502163319230.png)

### 2.3.2 三台机器修改配置文件config.xml

![image-20210502163410154](images/image-20210502163410154.png)

### 2.3.3 在三台机器的etc目录下新建metrika.xml文件

添加如下内容

```xml
<yandex>
<remote_servers>
    <clickhouse_cluster>
        <shard>
            <replica>
                <host>Hadoop02</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <replica>
                <host>Hadoop03</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <replica>
                <host>Hadoop04</host>
                <port>9000</port>
            </replica>
        </shard>
    </clickhouse_cluster>
</remote_servers>
<zookeeper>
    <node>
        <host>Hadoop02</host>
        <port>2181</port>
    </node>
    <node>
        <host>Hadoop03</host>
        <port>2181</port>
    </node>
    <node>
        <host>Hadoop04</host>
        <port>2181</port>
    </node>
</zookeeper>
<macros>
    <replica>Hadoop02</replica>
</macros>
  <networks> 
    <ip>::/0</ip>
  </networks>
  <clickhouse_compression> 
    <case> 
      <min_part_size>10000000000</min_part_size>
      <min_part_size_ratio>0.01</min_part_size_ratio>
      <method>lz4</method>
    </case>
  </clickhouse_compression>
</yandex>
```

### 2.3.4 三台机器启动ClickServer

![image-20210502164136241](images/image-20210502164136241.png)





# 3. 数据类型

## 3.1 与其他框架比较

![image-20210502170343834](images/image-20210502170343834.png)

## 3.2 整型

![image-20210502170728288](images/image-20210502170728288.png)

![image-20210502170804750](images/image-20210502170804750.png)

## 3.3 浮点型

Float32-float

![image-20210502171304985](images/image-20210502171304985.png)

![image-20210502171401953](images/image-20210502171401953.png)

![image-20210502171442847](images/image-20210502171442847.png)

## 3.4 布尔型

![image-20210502171456965](images/image-20210502171456965.png)

## 3.5 字符串

![image-20210502171508559](images/image-20210502171508559.png)

![image-20210502171534650](images/image-20210502171534650.png)

## 3.6 枚举类型

![image-20210502171611959](images/image-20210502171611959.png)

![image-20210502171653099](images/image-20210502171653099.png)

![image-20210502172116247](images/image-20210502172116247.png)

![image-20210502172124823](images/image-20210502172124823.png)

![image-20210502172139851](images/image-20210502172139851.png)



## 3.7 数组

![image-20210502172244083](images/image-20210502172244083.png)

![image-20210502172316987](images/image-20210502172316987.png)

## 3.8 元组

![image-20210502172452130](images/image-20210502172452130.png)

![image-20210502172555425](images/image-20210502172555425.png)

## 3.9 Date

![image-20210502172612268](images/image-20210502172612268.png)



# 4. 表引擎

![image-20210502185928625](images/image-20210502185928625.png)

## 4.1 TinyLog

![image-20210502190154984](images/image-20210502190154984.png)

![image-20210502190640809](images/image-20210502190640809.png)

![image-20210502191105882](images/image-20210502191105882.png)

## 4.2 Memory

![image-20210502191124443](images/image-20210502191124443.png)

## 4.3 Merge

![image-20210502191302785](images/image-20210502191302785.png)

![image-20210502191453654](images/image-20210502191453654.png)

## 4.4 MergeTree(重点)

![image-20210502192158121](images/image-20210502192158121.png)

![image-20210502192853993](images/image-20210502192853993.png)

![image-20210502192928665](images/image-20210502192928665.png)

![image-20210502193459402](images/image-20210502193459402.png)



## 4.5 ReplacingMergeTree

![image-20210502194149415](images/image-20210502194149415.png)

![image-20210502194220940](images/image-20210502194220940.png)

```sql
create table aliyun_app_info(package_name String,app_name String,code_tag String,create_time Date,status UInt8,other_info String,md5_value String)ENGINE=ReplacingMregeTree() ;
```



## 4.6 SummingMergeTree

![image-20210502200705301](images/image-20210502200705301.png)

![image-20210502200905225](images/image-20210502200905225.png)

## 4.7 Distributed(重点)

![image-20210502201427430](images/image-20210502201427430.png)

参数解析：

![image-20210502201600301](images/image-20210502201600301.png)

![image-20210502201608292](images/image-20210502201608292.png)

![image-20210502201634672](images/image-20210502201634672.png)

![image-20210502202133396](images/image-20210502202133396.png)



# 5. SQL语法

## 5.1 CREATE

### 5.1.1 CREATE DATABASE

![image-20210502202700371](images/image-20210502202700371.png)

![image-20210502202714296](images/image-20210502202714296.png)

## 5.1.2 CREATE TABLE

![image-20210502202740098](images/image-20210502202740098.png)

![image-20210502203703326](images/image-20210502203703326.png)

![image-20210502203712155](images/image-20210502203712155.png)

![image-20210502203725979](images/image-20210502203725979.png)

## 5.2 INSERT INTO

![image-20210502203744458](images/image-20210502203744458.png)

![image-20210502203754728](images/image-20210502203754728.png)

## 5.3 ALTER

![image-20210502203809699](images/image-20210502203809699.png)

![image-20210502203851278](images/image-20210502203851278.png)

![image-20210502203936391](images/image-20210502203936391.png)

![image-20210502203957132](images/image-20210502203957132.png)

## 5.4 DESCRIBE TABLE

![image-20210502204021277](images/image-20210502204021277.png)

## 5.5 CHECK TABLE

![image-20210502204042640](images/image-20210502204042640.png)



# 6. 从HDFS导入数据

![image-20210502204151782](images/image-20210502204151782.png)

## 6.1 案例一：查询HDFS上的CSV文件

![image-20210502204301060](images/image-20210502204301060.png)

![image-20210502204427198](images/image-20210502204427198.png)



![image-20210502205727481](images/image-20210502205727481.png)

![image-20210502205740288](images/image-20210502205740288.png)

## 6.2 案例二：导入HDFS上的数据

![image-20210502220419620](images/image-20210502220419620.png)

![image-20210502220430493](images/image-20210502220430493.png)

# 7. 优化

## 7.1 max_table_size_to_drop

![image-20210502220753195](images/image-20210502220753195.png)

## 7.2 max_memory_usage

![image-20210502220839052](images/image-20210502220839052.png)

## 7.3 删除多个节点上的同一张表

![image-20210502220906563](images/image-20210502220906563.png)

## 7.4 自动数据备份

![image-20210502221424014](images/image-20210502221424014.png)

![image-20210502221545352](images/image-20210502221545352.png)

![image-20210502221646838](images/image-20210502221646838.png)

![image-20210502221815400](images/image-20210502221815400.png)

![image-20210502223459144](images/image-20210502223459144.png)







