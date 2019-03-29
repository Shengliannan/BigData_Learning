# Spark快速大数据分析读书笔记

## 一、Spark核心

### 1.1 Spark核心概念简介

每个Spark应用程序都由一个驱动程序（driver program）来发起集群上的各种并行操作。驱动器程序包含应用的main函数，并且定义了集群上的分布式数据集，还对这些分布式数据集应用了相关操作。

驱动器程序通过一个SparkContext对象来访问Spark。这个对象代表对计算集群的一个连接。shell启动时已经自动创建了一个SparkContext对象，是一个叫做sc的变量。

一旦有了SparkContext，就可以用它来创建RDD。

### 1.2 RDD基础

弹性分布式数据集—RDD(Resilient Distributed Dataset)

Spark中的RDD就是一个不可变的分布式对象集合。每个RDD都被分为多个分区，这些分区运行在集群中的不同节点。

RDD支持两种类型的操作：转化操作和行动操作

总的来说，每个Spark程序或shell回话都按照如下方式工作。

（1）从外部数据创建出输入RDD。

（2）使用诸如filter()这样的转化操作对RDD进行转化，以定义新的RDD。

（3）告诉Spark对需要被重用的中间结果RDD执行persisit()操作。

（4）使用行动操作（例如count()和filter()等）来触发一次并行计算，Spark会对计算进行优化后再执行。

```scala
# 从已有的集合创建RDD
val lines = sc.parallelize(List("pandas","i like pandas"))
# 从外部文件存储读取创建RDD
lines = sc.textFile("/path/to/README.md")
# filter()函数
val line1 = lines.filter(line => line.contain("python"))
# 

```


## 二、 在集群上运行Spark

角色：驱动器，执行器

### 2.1 驱动器节点

Spark驱动器是执行程序中的main()方法的进程。

它执行用户编写的用来创建SparkContext，创建RDD，以及进行RDD的转化操作和行动操作的代码。

驱动器程序在Spark应用中有两个职责：

- 把用户程序转为任务
- 为执行器节点调度任务



## 三、Spark SQL

Spark SQL提供了一种特殊的RDD：SchemaRDD，SchemaRDD是存放Row对象的RDD，每个Row对象代表一行记录。



## 四、Spark Streaming

和Spark基于RDD的概念很相似，Spark Streaming使用离散化流（discretized stream）作为抽象表示，叫做DStream。DStream是随时间推移而受到的数据的序列。

和批处理程序不同，Spark Streaming应用需要进行额外配置来保证24/7不间断工作。***检查点（checkpointing）机制***：也就是数据储存到可靠文件系统（比如HDFS）上的机制，这也是Spark Streaming用来实现不间断工作的主要方式）。

遇到失败后如何重启应用，以及如何把应用设置为自动重启模式

