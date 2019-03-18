# HBase不睡觉

***

HBase是Apache旗下一个高可靠性、高性能、面向列、可伸缩的分布式存储系统。利用HBase技术可在廉价的PC服务器上搭建大规模的存储化集群，使用HBase可以对数十亿级别的大数据进行实时性的高性能读写，在满足高性能的同时还保证了数据存取的原子性。





HBase安装部署、主要功能、架构设计、性能优化

HBase的配置、部署、优化、二次开发、NoSQL

### NoSQL：Not Only SQL

### NoSQL数据库和传统数据库的区别

### CAP理论

Consistency：一致性，数据一致更新，所有数据变动都是同步的。

Availability：可用性，良好的响应性能。

Partition Tolerance：分区容错性，可靠性。

***任何分布式系统只可同时满足二点，没法三者兼顾。***

最终一致性。

### HBase起源

Bigtable

HBase的存储是基于Hadoop的。

Hadoop实现了一个分布式文件系统（HDFS）。HDFS有高容错性的特点，被设计用来部署在低廉的硬件上。

HBase采用的是Key/Value的存储方式。HBase是一个列式数据库（对比于传统的行式数据库而言）。

数据分析是HBase弱项，因为对于HBase乃至整个NoSQL生态圈来说，基本上都是***不支持表关联***的。当你想实现group by或者order by的时候，你会发现，你需要写很多的代码来实现MapReduce。

### 部署架构

HBase的架构：

从HBase的部署架构上来说，HBase有两种服务器：Master服务器和几个RegionServer服务器。Master服务器负责维护表结构信息，实际的数据都存储在RegionServer服务器上。

HBase有一点很特殊：***客户端获取数据由客户端直连RegionServer的***，所以你会发现Master挂点之后你依然可以查询数据，但就是不能新建表了。

RegionServer是直接负责存储数据的的服务器。RegionServer保存的表数据直接存储在Hadoop的HDFS上。

RegionServer非常依赖Zookeeper服务，可以说没有ZooKeeper就没有HBase。ZooKeeper在HBase中扮演的角色类似一个管家。ZooKeeper管理了HBase的所有RegionServer信息，包括具体的数据段存放在哪个RegionServer上。

客户端每次与HBase连接，其实都是先与ZooKeeper通信，查询出哪个RegionServer需要连接，然后再连接RegionServer。

Region是什么？

Region就是一段数据的集合。HBase中的表一般拥有一个到多个Region。



