[TOC]
# 图解Spark 核心技术与案例实战

***

## Spark简介：

Spark使用Scala语言进行编写，它是一种面向对象、函数式编程语言，能够像操作本地集合对象一样轻松的地操作分布式数据集。Spark具有运行速度快、易用性好、通用性强和随处运行等特点。

Spark相对于Hadoop有如此快的计算速度有数据本地性、调度优化和传输优化等原因，其中最主要的是***基于内存计算和引入DAG执行引擎***。

#### Spark与MR比较

Spark是通过借鉴Hadoop MapReduce发展而来的，继承了其分布式并行计算的优点，并改进了MR的缺点。

（1）Spark把中间数据放在内存中，迭代运行效率高。MR中的计算结果保存在磁盘上。Spark支持DAG图的分布式并行计算的编程框架，减少了迭代过程中的数据落地，提高了处理效率。

（2）Spark的容错性高。Spark引进了弹性分布式数据集（Resilient Distributed DataSet，RDD），可重建丢失数据。

#### 资源调度
YARN
Mesos

### Spark Streaming
Spark Streaming是一个对实时数据流进行高吞吐、高容错的流式处理系统，可以对多重数据源进行类似Map、Reduce和Join等操作。
***Spark Streaming最大的优势是提供的处理引擎和RDD编程模型可以同时进行批处理和流处理***。






## Spark组件

Spark core：提供内存计算框架

SparkStreaming：实时处理应用

Spark SQL：即时查询



Spark的编程模型

作业执行

存储原理

运行架构

Spark SQL即时查询

Spark Streaming实时流处理



spark操作

1. 使用spark-submit将打包好的jar提交到Spark上运行（模板）

```shell
./bin/spark-submit \
--class <main-class> \           # 执行程序的主类
--master <master-url> \          # 指向集群的url
--deploy-mode <deploy-mode> \    # 设置以什么模式运行，cluster or client，默认：client模式
--conf <key>=<value>             
... #其他参数，请参考spark-submit --help
<jar的路径> \

# 注意："\"反斜杠用于换行。

demo：
./bin/spark-submit \
--class "helloworld" \
--master spark://master:7070 \
--deploy-mode cluster \
--executor-memory 20G \
--total-executor-cores 100 \
/path/to/examples.jar \
```



### Spark体系结构

主节点：Master

从节点：Worker

![985154-20170825142247152-132882326](C:\Users\suntiansheng2\Desktop\985154-20170825142247152-132882326.png)

![1552492201974](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552492201974.png)

## Spark on YARN

yarn-client、yarn-cluster

> [参考](https://www.cnblogs.com/BYRans/p/5889374.html)
>
> [参考](https://www.imooc.com/article/267635)

![1552493571980](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552493571980.png)

![1552493616390](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552493616390.png)

Container，executor的关系

Container：YARN中的抽象资源

executor：执行任务的进程

资源管理角色：

1. ResourceManager：负责整个集群的资源管理和分配
2. ApplicationMaster：YARN中的每个Application对应一个AM进程，负责与RM协商获取资源后告诉NodeManager为其分配并启动Container。
3. NodeManager：每个节点的资源和任务管理器，负责启动/停止Container，并监测资源情况。
4. Container：YARN中的抽象资源。

任务调度角色：

 	1. Driver：和ClusterManager通信，进行资源申请、任务分配并监控其运行状况等。
 	2. DAGScheduler:把spark作业装换成stage的DAG图。
 	3. TaskScheduler:把Task分配给具体的Executor。



![1552494524926](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552494524926.png)

![1552523772201](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552523772201.png)



## RDD

### 1. 什么是RDD？

RDD即弹性分布式数据集（Resilient Distributed Datasets），是Spark对数据的核心抽象，也就意味着在Spark上进行数据挖掘首先需要将待处理数据源转化成RDD，在此RDD上进行操作。



### 2. RDD特点：

- 容错性
- 易恢复性
- 高效性

### 3. RDD相关概念

- 转化操作

  由一个RDD生成一个新的RDD

- 行动操作

  对RDD计算出一个结果，应用程序的返回值或向存储系统导出操作

- 惰性求值

  可以在任何时候通过转化操作定义新的RDD，Spark只记录RDD的转换过程，不会直接进行计算，只有第一次在一个行动操作中用到时，才会真正触发计算。

- RDD缓存

  默认情况下，Spark的RDD会在你每次对它们进行行动操作时重新计算。如果想在多个行动操作中重用同一个RDD，可以使用RDD.persist()/RDD.cache()让Spark把这个RDD缓存下来。

- 持久化

- checkpoint机制

基本RDD转化操作

   （1）map、filter

   经常用到的两个转化操作是map()和filter()，两者的共同点在于会触发RDD中所有元素进行遍历。转化操作map()接收一个函数，把这个函数用于RDD的每个元素，将函数的返回结果作为结果RDD中对应元素的值。而转化操作filter()则接受一个函数，并将RDD中满足该函数的元素放入新的RDD中返回。
### 4. 宽依赖，窄依赖
RDD之间存在依赖

- 窄依赖(Narrow Dependencies)

  每一个父RDD的分区最多被子RDD的一个分区使用

  父 : 子=N : 1（只有一个儿子）

  map,filter,union

- 宽依赖(Wide Dependencies)

  发生shuffle

  多个子RDD分区会依赖同一个父RDD

  父 : 子=N : N（有多个儿子）

  groupByKey

血统（Lineage）：Linage会记录RDD的元数据的转换行为，以便恢复丢失的分区



### 5. stage的划分

是否发生了shuffle过程

划分stage的目的是为了生成任务

从后往前推stage

## Spark SQL

> [参考](http://www.cnblogs.com/qingyunzong/p/8987579.html#_label0)

1. Spark SQL是什么

   ​        Spark SQL是用于结构化数据、半结构化数据处理的Spark高级模块、可用于从各种结构化数据源的文件存储格式，具有降低查询成本、高效压缩的特点，广泛用于大数据存储、分析领域。

### Spark SQL数据核心抽象—DataFrame

1. DataFrame的概念

   ​        DataFrame的定义与RDD类似，即都是Spark平台用以分布式并行计算的不可变分布式数据集合。与RDD最大的不同在于，RDD仅仅是一条条数据的集合，并不了解每一条数据的内容是怎样的，而DataFrame明确的了解每一条数据有几个命名字段组成，***即可以形象地理解为RDD是一条条数据组成的一维表，而DataFrame是每一行数据都有共同清晰的列划分的二维表***。概念上来说，它和关系型数据库的表或者R和Python中data frame等价，只不过DataFrame在底层实现了更多优化。

2. RDD与DataFrame的区别

   ​        RDD和DataFrame均为Spark平台对数据的一种抽象，一种组织方式，但是两者的地位或者说设计目的却截然不同。RDD是整个Spark平台的存储、计算以及任务调度的逻辑基础，更具有通用性，适用于各类数据源，而DataFrame是只针对结构化数据源的高级数据抽象，其中在DataFrame对象的创建过程中必须指定数据集的结构信息（Schema），所以DataFrame生来便是具有专用性的数据抽象、只能读取具有鲜明结构的数据集。

### Spark SQL编程入门

1. 程序主入口：SparkSession

   ​        Spark SQL模块的编程主入口点是SparkSession，SparkSession对象不仅为用户提供了创建DataFrame对象、读取外部数据源并转化为DataFrame对象以及执行sql查询的API，还负责记录着用户希望Spark应用如何在Spark集群运行的控制、调优参数，是SparkSQL的上下文环境，是运行的基础。

2. 执行SQL查询

   ​	SparkSession为用户提供了直接执行sql语句的SparkSession(sqlText:String)方法，sql语句可直接作为字符串传入sql()方法中，sql查询所得到的结果依然为DataFrame对象，在SparkSQL模块上直接执行sql语句的查询需要首先将标志着结构化数据源的DataFrame对象注册成临时表，进而在sql语句中对该临时表进行查询操作，示例:

   ```
   //调用DataFrame提供的createOrReplaceTempView，将df（沿用记录着姓名、年龄、身高、体重等学生信息的DataFrame对象—df）注册成student临时表
   df.createOrReplaceTempView("student")
   //调用SparkSession提供的sql接口，对studnet临时表进行sql查询，需要注意的是sql()方法的返回值仍为DataFrame对象
   val sqlDF = sparkSession.sql("SELECT name,age FROM student")
   sqlDF.show(3)
   ```

   ​	Spark SQL的SQL接口全面支持SQL的select标准语法，包括SELECT、DINSTINCT、from子句、where子句、order by子句、group by子句、having子句、join子句；还有典型的SQL函数，例如：avg()、count()、max()、min()等

   > [官方文档](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$)

3. 全局临时表

4. Dataset

   ​	Spark SQL的核心数据抽象DataFrame正式特殊的Dataset，Dataset将会逐渐的取代RDD称为Spark是开发编程的主要使用API。

   ​	DataFrame等价于Dataset[Row],SparkSession接口读取数据源将结构化数据文件的每一行作为一个Row对象，最后由Row对象和对应的结构信息组成了Dataset对象。

全都弄会

https://www.nowcoder.com/discuss/163053

DataFrame

Spark对数据的核心抽象—RDD

Spark SQL的核心数据抽象—DataFrame

***RDD vs DataFrame***





## Spark Streming

1. Spark Streaming  vs Storm

![Spark Streaming vs Storm](https://github.com/Shengliannan/BigData_Learning/blob/master/img/spark%20Streaming%20%20vs%20Storm.jpg?raw=true)

Spark Streaming是把流转化成一个个小的批来处理，这种方案的一个问题是我们需要的延迟越低，额外开销占的比例就会越大，这导致了Spark Streaming很难做到秒级甚至亚秒级的延迟。

