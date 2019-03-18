# Hadoop 权威指南

***

数据格式

Avro（用于数据序列化）

Parquet（用于嵌套数据）



数据摄取

Flume（用于序列化数）

Sqoop（用于块数据传输）



数据处理

高层数据处理工具（Pig、Hive、Cruch、Spark、Hadoop）



存储

HBase分布式数据库



协作

Zookeeper分布式配置服务



> [书的官网](http://hadoopbook.com/)
>
> [书中代码](https://github.com/tomwhite/hadoop-book)

传统数据库的存储机构

### [数据库索引及其数据结构](https://www.cnblogs.com/fanguangdexiaoyuer/p/5385109.html)



map和reduce

MapReduce任务分为两个处理阶段：map阶段和reduce阶段。每阶段都以键值对作为输入和输出，其类型由程序员来选择。程序员还需要写两个函数：map函数和reduce函数。

##### map函数

Mapper类型是一个泛型类型，它有四个形参类型，分别指定map函数的输入键、输入值、输出键和输出值的类型。

Hadoop本身提供了一套可优化网络序列化传输的基本类型，而不直接使用Java内嵌的类型。这些类型都在org.apache.hadoop.io包中。

LongWritable类型（相当于Java的Long类型）

Text类型（相当于Java中的String类型）

IntWritable类型（相当于Java的Integer类型）

##### reduce函数

reduce函数有四个形式参数类型用于指定输入和输出类型。***reduce函数的输入类型必须匹配map函数的输出类型***。

job.setJarByClass(maxTemperature.class);
job.setJobName("Max temperature");
FileInputFormat.addInputPath(job,new Path(args[0]));
FileOutPutFormat.setOutPath(job,new Path(args[1]));
job.setMapperClass(maxTemperatureMapper.class);
job.setReducerClass(MaxTemperatureReducer.class);
job.setOutputKeyClass(Text.class);
job.setOutputValueClass(InWritable.class);
System.exit(job.waitForCompletion(true)?0:1);

Job对象指定作业执行规范。我们可以用它来控制整个作业的运行。我们在Hadoop集群上运行这个作业时，要把代码打包成一个JAR文件（Haodop在集群上发布这个文件）。不必明确指定JAR文件的名称，在Job对象的setJarByClass()方法中传递一个类即可。Hadoop利用这个类来查找包含它的JAR文件，进而找到相关的JAR文件。

构造Job对象之后，需要指定输入和输出数据的路径。调用FileInputFormat类的静态方法addInputPath()来定义输入数据的路径，这个路径可以是单个的文件、一个目录（此时，将目录下司所有文件当作输入）或符合特定文件模式的一些列文件。由函数名可值，可以多次调用addInputPath()来实现多路径输入。

调用FileOutputFormat类中静态方法setOutputPath()来指定输出路径(只能有一个输出路径)。这个方法指定的是reduce函数输出文件的写入目录。***在运行作业前该目录是不应该存在的，否则Hadoop会报错并拒绝运行作业***。这种预防措施的目的是防止数据丢失（长时间运行的作业如果结果被意外覆盖，肯定是非常恼人的）。

Hadoop将MapReduce的输入数据划分为等长的小数据块，称为输入分片(input split)或简称“分片”。Hadoop为每个分片构建一个map任务，并由改任务来运行用户自定义的map函数从而处理分片中的每条记录。

一个合理的分片大小趋向于HDFS的一个块的大小，默认是128MB。

如果有好多个reduce任务，每个map任务就会针对输出进行分区（partition），即为每个reduce任务建一个分区。每个分区有很多键值（及其对应的值），但每个键对应的键-值对记录都在同一分区中。分区可由用户定义的分区函数控制，但通常用默认的partitioner通过哈希函数来分区，很高效 。

combiner函数

Hadoop允许用户针对map任务的输出指定一个combiner（就像mapper和reducer一样），combiner函数的输出作为reduce函数的输入。由于combiner属于优化方案，所以Hadoop无法确定要对一个指定的map任务输出记录调用多少次combiner。换而言之，不管调用combiner多少次，0次、1次或多次，reducer的输出结果都是一样的。（没看懂）

combiner函数不能取代reduce函数。我们仍然需要reduce函数来处理不同map输出中具有相同键的记录。但combiner函数能帮助减少mapper和reducer之间的数据传输量。

Hadoop Streaming

Hadoop提供了MapReduce的API，允许使用非Java的其他语言来写自己的map和reduce函数。Hadoop Streaming使用Unix标准流作为Hadoop和应用程序之间的接口，我们可以使用任何其他的语言通过标准输入/输出来写MapReduce程序。

### Hadoop分布式文件系统

HDFS Hadoop Distributed Filesystem

大量的小文件 由于namenode将文件系统的元数据存储在内存中，因此该文件系统所能存储的文件总数受限于namenode的内存容量。

每个文件、目录和数据块的存储信息大约占150字节。

eg：一百个文件，且每个文件占一个数据块，那至少需要300MB内存。

#### HDFS的概念
##### 数据块
每个磁盘都有默认的数据块大小，这是磁盘进行数据读/写的最小单位。构建于单个磁盘之上的文件系统通过磁盘块来管理改文件系统中的块，该文件系统块的大小可以是磁盘块的整数倍。文件系统块一般为几千字节，而磁盘块一般为512字节。这些信息（文件系统块大小）对于需要读/写文件的文件系统用户来说是透明的。尽管如此，系统仍然提供了一些工具（如df和fsck）来维护文件系统，由它们对文件系统中的块进行操作。

HDFS同样也有块（block）的概念，但是大得多，默认为128MB。与单一磁盘上文件系统相似，HDFS上文件也被划分为块大小的多个分块（chunk），作为独立的存储单元。但与单一磁盘的文件系统不同的是，HDFS中小于一个块大小的文件不会占据整个块的空间（例如，当一个1MB的文件存储在一个128MB的块中时，文件只使用1MB的磁盘空间，而不是128MB）。

***HDFS的块为什么这么大（目的是为了最小化寻址开销）***

如果快足够大，从磁盘传输数据的时间会明显大于定位整个块开始位置所需的时间，因而，传输一个由多个块组成的大文件的时间取决于磁盘传输速率。

分布式文件系统中的一个文件可以大于网络中的任意一个磁盘的容量。文件的所有块并不需要存储在同一个磁盘上，因此它们可以利用集群上的任意一个磁盘进行存储。

 hdfs fsck / -files -blocks

##### namenode

namenode管理文件系统的命名空间。它维护者文件系统树及整棵树内所有的文件和目录。这些信息以两个文件形式永久保存在本地磁盘上：命名空间镜像文件和编辑日志文件。namenode也记录着每个文件中各个块所在的数据节点信息，但它并不永久保存块的位置信息，因为这些信息会在系统启动时根据数据节点信息重建。

容错。

##### datanode



#### 数据流

剖析文件读取

客户端与之交互的HDFS、namenode和datanode之间的数据流

客户端通过调用FileSystem对象的open()方法来打开希望读取的文件，对于HDFS来说，这个对象是DistributedFileSystem的一个实例。DistributedFileSysem通过使用远程过程调用（RPC）来调用namenode，以确定文件起始块的位置。对于每一个块，namenode返回存有该块副本的datanode地址。

DistributedFileSystem类返回一个FSDataInputStream对象（一个支持文件定位的输入流）给客户端以便读取数据。FSDataInputStream类转而封装成DFSInputStream对象，该对象管理着datanode和namenode的I/O。接着客户端对这个输入流调用read()方法。存储着文件起始几个块的datanode地址的DFSInputStream随即连接距离最近的文件中的第一个块所在的datanode。通过对数据流反复调用read()方法，可以将数据从datanode传输到客户端。到达块的末端时，DFSInputStream关闭与该datanode的连接，然后寻找下一块最佳的datanode.

剖析文件写入

一致模型

通过distcp并行复制

```
# 并行处理文件复制文件
hadoop distcp dir1 dir2
hadoop disctcp -overwrite dir1 dir2 //强制覆盖原有文件
hadoop distcp -update dir1 dir2  //仅更新发生变化的文件
```

即使对于单个文件复制，由于hadoop fs  -cp通过运行命令的客户端进行文件复制，因此更倾向于使用distcp变种复制大文件。

distcp是作为一个MapReduce作业来实现的，该复制作业是通过集群中并行运行map来完成，这里没有reducer。



关于YARN（Yet Another Resource Negotiator）

集群资源管理系统

RM NM

AM？

为了在YARN上运行一个应用，首先，客户端联系资源管理器，要求它运行一个***application maser进程***。然后，资源管理器找到一个能够容器中启动application mater的节点管理器。



### Hadoop IO操作

压缩：LZO，SNAPPY



udf的编写

