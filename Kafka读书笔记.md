# Kafka权威指南&Kafka源码解析与实战

***

Kafka是一个流平台，在这个平台上可以发布和订阅数据流，并把它们保存起来、进行处理。这就是构建Kafka的初衷。（权威指南）

#### 消息模式

JSON和XML易用，可读性好。***但是缺乏强类型处理能力，不同版本之间的兼容性也不是很好***。

Apache Avro，最初是为Hadoop开发的一款序列化框架。Avro提供了一种紧凑的序列化格式，模式和消息体是分开的，当模式发生变化时，不需要重写生成代码；它还支持强类型和模式金华，其版本既向前兼容，也向后兼容。

#### 生产者和消费者

生产者创建消息，一般情况下，一个消息会被发布到一个特定的主题上。生产者在默认情况下把消息均衡地分不到主题的所有分区上，而并不关心特定消息会被写到哪个分区。不过，在某些情况下，生产者会把消息直接写到指定的分区。这通常是通过消息键和分区器来实现的，分区器为键生成一个散列值，并将其映射到指定的分区上。这样可以保证包含同一个键值的消息会被写到同一个分区上。

***消费者是消费者群组的一部分。群组保证每个分区只能被一个消费者使用***。



Kafka和Spark的底层都是用Scala编写的。

实现原理、运维工具、客户端编程、实际应用

LinkedIn使用的数据系统包括：

- 全文搜索
- Social Graph（社会图谱）
- Voldemort（键值存储）
- Espresso（文档存储）
- 推荐引擎
- OLAP（查询引擎）
- Hadoop
- Teradata（数据仓库）
- Ingraphs（监控图表和指标服务）

#### Kafka的主要设计目标

Kafka作为一种分布式、基于发布/订阅的消息系统，其主要设计目标如下：

- 以时间复杂度为O(1)的方式提供***消息持久化能力***，



Kafka的问题

无法保证全局消息有序性，只能同一分区中的消息有效。

kafka只能保证分区内的有序性。针对部分消息有序（message.key相同的message要保证消费顺序）场景，可以在producer往kafka插入数据时控制，同一key分发到同一partition上面。



Kafka管理offset
消费者需要自己保留一个offset，从kafka 获取消息时，只拉去当前offset 以后的消息。Kafka 的scala/java 版的client 已经实现了这部分的逻辑，将offset 保存到zookeeper 上



为何要本地保存位点，不是保存在zk上吗

如何保证不同分区的消息的有序性。



producer、consumer、consumer group、broker、partition、topic之间的关系

每个分区都有备份，一个topic对应不同分区，一个topic的不同分区可以存在不同的broker上



Kafka的内部通信协议

Kafka消息系统内部的通信协议主要区分为三种：

- 生产者和Broker之间
- 消费者和Broker之间
- Broker和Broker之间

nohup命令

#### Broker

Broker内部模块组成

KafkaServer类包含的模块：

- SocketServer（监听Socket请求）
- KafkaRequestHandlerPool（请求处理资源池）
- LogManager（日志管理）
- ReplicaManager（分区副本管理）
- OffsetManager（偏移量管理）
- KafkaScheduler（后台任务调度资源池）
- KafkaApis（业务逻辑实现层）
- KafkaHealthcheck（提供Broker健康状态）
- TopicConfigManger（Topic配置信息管理）
- KafkaController（Kafka集群控制管理）

Kafka的位点管理

direct模式和receiver模式

> [参考](https://blog.csdn.net/wyqwilliam/article/details/84430548)

rtf自己管理位点，也可以zk管理，但是怕zk挂掉，位点丢失，会丢数据，或者有重复数据。

消费spark

Spark-Streaming

Storm

#### Kafka的综合实例

安防大数据的主要应用

Kakfa可以解耦数据的生产者和数据的消费者之间的强耦合。

分布式数据库：

Hbase、Redis、LevelDB

分布式计算框架：

Spark、MapReduce

分布式存储

HDFS、GFS、TFS





MR 进程模型

Spark 多线程

#### ELK

ELK其实并不是一款软件，而是一整套解决方案，是三个软件产品的首字母缩写，

ElasticSearch、LogStash和Kibana。







徐坤浩:
基础架构、高性能的原因、消息写入的过程、一些调优办法。。。大概就会这些方面

影子:
问个问题，kafka能否保持不同分区的消息有序性

徐坤浩:
不能哦

徐坤浩:
只能保证单个 partation 的有序性，不保证 partation 之间或者说是 topic 级别的有序性

徐坤浩:
而且不保证消息完全落盘，因为kafak 是写到 os 的页缓存就完事儿了，消息落盘是等这个页缓存写满之后自动持久化到磁盘的

徐坤浩:
所以kafka高性能的原因：顺序读写、零拷贝、只写到页缓存、不保证强有序性

影子:
那自动持久化的意思就是这个持久化的过程是由操作系统完成的吗

徐坤浩:
对的 默认情况下是页缓存写满之后 os将页缓存中的消息写到磁盘；但是也可以在配置文件更改持久化的策略，比如改成一段时间持久化一次或者累计发送N条消息之后持久化一次

徐坤浩:
但是这都不如 把持久化完全交给os处理 速度快

徐坤浩:
哦 对了 高性能还有一个原因：Producer 端会合并小消息，等累积到一定大小之后才发送，而不是来一个消息就发送一个消息