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

