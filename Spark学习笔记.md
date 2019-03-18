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

### Spark体系结构

主节点：Master

从节点：Worker

![985154-20170825142247152-132882326](C:\Users\suntiansheng2\Desktop\985154-20170825142247152-132882326.png)

![1552492201974](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552492201974.png)

yarn-client

yarn-cluster

![1552493571980](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552493571980.png)

![1552493616390](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552493616390.png)

container，executor的关系

![1552494524926](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552494524926.png)

![1552523772201](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1552523772201.png)

## Spark SQL

> [参考](http://www.cnblogs.com/qingyunzong/p/8987579.html#_label0)



全都弄会

https://www.nowcoder.com/discuss/163053