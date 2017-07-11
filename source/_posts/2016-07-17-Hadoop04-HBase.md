---
layout:     post
title:      "Hadoop学习笔记(4)-HBase"
subtitle:   "HBase QuickStart"
date:       2016-07-17 18:00
author:     "Sylvanas Sun"
header-img: "img/HadoopPost.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 后端开发
    - Hadoop
    - 大数据
---



![](http://ww1.sinaimg.cn/mw690/63503acbjw1f5wxvstywwj20dx03odg2.jpg)

### 概述

&nbsp;&nbsp;HBase是一个分布式的、面向列的开源数据库，该技术来源于 Fay Chang 所撰写的Google论文“Bigtable：一个结构化数据的分布式存储系统”。就像Bigtable利用了Google文件系统（File System）所提供的分布式数据存储一样，HBase在Hadoop之上提供了类似于Bigtable的能力。HBase是Apache的Hadoop项目的子项目。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。另一个不同的是HBase基于列的而不是基于行的模式。

&nbsp;&nbsp;与FUJITSU Cliq等商用大数据产品不同，HBase是Google Bigtable的开源实现，类似Google Bigtable利用GFS作为其文件存储系统，HBase利用Hadoop HDFS作为其文件存储系统；Google运行MapReduce来处理Bigtable中的海量数据，HBase同样利用Hadoop MapReduce来处理HBase中的海量数据；Google Bigtable利用 Chubby作为协同服务，HBase利用Zookeeper作为对应。

### HBase的特点

 1. Hbase可以往数据里面insert，也可以update一些数据，但update的实际上也是insert，只是插入一个新的时间戳的一行。delete数据，也是insert，只是insert一行带有delete标记的一行。Hbase的所有操作都是追加插入操作。Hbase是一种日志集数据库。它的存储方式，像是日志文件一样。它是批量大量的往硬盘中写，通常都是以文件形式的读写。这个读写速度，取决于硬盘与机器之间的传输有多快。
 2. Hbase中数据可以保存许多不同时间戳的版本（即同一数据可以复制许多不同的版本，准许数据冗余，也是优势）。数据按时间排序，因此Hbase特别适合寻找按照时间排序寻找Top n的场景。找出某个人最近浏览的消息，最近写的N篇博客，N种行为等等，因此Hbase在互联网应用非常多。
 3. Hbase只有主键索引，因此在建模的时候会遇到了问题。例如，在一张表中，很多的列我都想做某种条件的查询。但却只能在主键上建快速查询。
 4. Hbase是列式数据库,列式数据库的优势在于数据分析。
 5. Hbase中的数据都是字符串，没有其他类型。

### 行式数据库与列式数据库的区别

 **行式数据库**
 
 &nbsp;&nbsp;以Oracle为例，数据文件的基本组成单位：块/页。块中数据是按照一行行写入的。这就存在一个问题，当我们要读一个块中的某些列的时候，不能只读这些列，必须把这个块整个的读入内存中，再把这些列的内容读出来。换句话就是：为了读表中的某些列，必须要把整个表的行全部读完，才能读到这些列。这就是行数据库最糟糕的地方。
 
 **列式数据库** 
 
 &nbsp;&nbsp;列式数据库是以列作为元素存储的。同一个列的元素会挤在一个块。当要读某些列，只需要把相关的列块读到内存中，这样读的IO量就会少很多。通常，同一个列的数据元素通常格式都是相近的。这就意味着，当数据格式相近的时候，数据就可以做大幅度的压缩。所以，列式数据库在数据压缩方面有很大的优势，压缩不仅节省了存储空间，同时也节省了IO。（这一点，可利用在当数据达到百万、千万级别以后，数据查询之间的优化，提高性能，示场景而定）

### HBase架构

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f5x1207zfmj20gr0atq4h.jpg)

&nbsp;&nbsp;HBase采用Master/Slave架构搭建集群，它隶属于Hadoop生态系统，由以下类型节点组成：HMaster节点、HRegionServer节点、ZooKeeper集群，而在底层，它将数据存储于HDFS中，因而涉及到HDFS的NameNode、DataNode等节点。

**Zookeeper**

&nbsp;&nbsp;Zookeeper Quorum存储-ROOT-表地址、HMaster地址。HRegionServer把自己以Ephedral方式注册到Zookeeper中，HMaster随时感知各个HRegionServer的健康状况。

**HMaster**

&nbsp;&nbsp;HMaster没有单点问题,HBase中可以启动多个HMaster，通过Zookeeper的Master Election机制保证总有一个Master在运行。

&nbsp;&nbsp;HMaster主要负责Table和Region的管理工作

 1. 实现DDL操作（Data Definition Language，namespace和table的增删改，column familiy的增删改等）。
 2. 管理HRegionServer的负载均衡，调整Region分布。
 3. 管理和分配HRegion，比如在HRegion split时分配新的HRegion；在HRegionServer退出时迁移其内的HRegion到其他HRegionServer上。
 4. 权限控制（ACL）。

**HRegionServer**

&nbsp;&nbsp;HBase中最核心的模块，主要负责响应用户I/O请求，向HDFS文件系统中读写数据。

 1. 存放和管理本地HRegion。
 2. 读写HDFS，管理Table中的数据。
 3. Client直接通过HRegionServer读写数据（从HMaster中获取元数据，找到RowKey所在的HRegion/HRegionServer后）。

**HRegion**

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f5x2cuji5gj20k409ddho.jpg)

&nbsp;&nbsp;HBase使用RowKey将表水平切割成多个HRegion，从HMaster的角度，每个HRegion都纪录了它的StartKey和EndKey（第一个HRegion的StartKey为空，最后一个HRegion的EndKey为空），由于RowKey是排序的，因而Client可以通过HMaster快速的定位每个RowKey在哪个HRegion中。HRegion由HMaster分配到相应的HRegionServer中，然后由HRegionServer负责HRegion的启动和管理，和Client的通信，负责数据的读(使用HDFS)。

### HBase集群搭建

&nbsp;&nbsp;如果HDFS是HA集群,需要把HDFS的core-site.xml和hdfs-site.xml copy到conf下。

#### hbase-env.sh

```
//告诉hbase使用外部的zookeeper
export HBASE_MANAGES_ZK=false
```

#### hbase-site.xml

```
<configuration>
	<!-- 指定hbase在HDFS上存储的路径 -->
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://ns1/hbase</value>
    </property>
	<!-- 指定hbase是分布式的 -->
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
	<!-- 指定zk的地址，多个用“,”分割 -->
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>datanode01:2181,datanode02:2181,datanode03:2181</value>
    </property>
    <property>
        <name>hbase.master.maxclockskew</name>
        <value>180000</value>
        <description>Time difference of regionserver from master</description>
 	</property>
</configuration>
```

#### regionservers

&nbsp;&nbsp;配置regionserver的节点,为了尽量实现数据本地化,可以与DataNode在同一个节点上。

### HBase常用Shell命令

```
进入hbase命令行
./hbase shell

显示hbase中的表
list

创建user表，包含info、data两个列族
create 'user', 'info1', 'data1'
create 'user', {NAME => 'info', VERSIONS => '3'}

向user表中插入信息，row key为rk0001，列族info中添加name列标示符，值为zhangsan
put 'user', 'rk0001', 'info:name', 'zhangsan'

向user表中插入信息，row key为rk0001，列族info中添加gender列标示符，值为female
put 'user', 'rk0001', 'info:gender', 'female'

向user表中插入信息，row key为rk0001，列族info中添加age列标示符，值为20
put 'user', 'rk0001', 'info:age', 20

向user表中插入信息，row key为rk0001，列族data中添加pic列标示符，值为picture
put 'user', 'rk0001', 'data:pic', 'picture'

获取user表中row key为rk0001的所有信息
get 'user', 'rk0001'

获取user表中row key为rk0001，info列族的所有信息
get 'user', 'rk0001', 'info'

获取user表中row key为rk0001，info列族的name、age列标示符的信息
get 'user', 'rk0001', 'info:name', 'info:age'

获取user表中row key为rk0001，info、data列族的信息
get 'user', 'rk0001', 'info', 'data'
get 'user', 'rk0001', {COLUMN => ['info', 'data']}

get 'user', 'rk0001', {COLUMN => ['info:name', 'data:pic']}

获取user表中row key为rk0001，列族为info，版本号最新5个的信息
get 'user', 'rk0001', {COLUMN => 'info', VERSIONS => 2}
get 'user', 'rk0001', {COLUMN => 'info:name', VERSIONS => 5}
get 'user', 'rk0001', {COLUMN => 'info:name', VERSIONS => 5, TIMERANGE => [1392368783980, 1392380169184]}

获取user表中row key为rk0001，cell的值为zhangsan的信息
get 'people', 'rk0001', {FILTER => "ValueFilter(=, 'binary:图片')"}

获取user表中row key为rk0001，列标示符中含有a的信息
get 'people', 'rk0001', {FILTER => "(QualifierFilter(=,'substring:a'))"}

put 'user', 'rk0002', 'info:name', 'fanbingbing'
put 'user', 'rk0002', 'info:gender', 'female'
put 'user', 'rk0002', 'info:nationality', '中国'
get 'user', 'rk0002', {FILTER => "ValueFilter(=, 'binary:中国')"}


查询user表中的所有信息
scan 'user'

查询user表中列族为info的信息
scan 'user', {COLUMNS => 'info'}
scan 'user', {COLUMNS => 'info', RAW => true, VERSIONS => 5}
scan 'persion', {COLUMNS => 'info', RAW => true, VERSIONS => 3}
查询user表中列族为info和data的信息
scan 'user', {COLUMNS => ['info', 'data']}
scan 'user', {COLUMNS => ['info:name', 'data:pic']}


查询user表中列族为info、列标示符为name的信息
scan 'user', {COLUMNS => 'info:name'}

查询user表中列族为info、列标示符为name的信息,并且版本最新的5个
scan 'user', {COLUMNS => 'info:name', VERSIONS => 5}

查询user表中列族为info和data且列标示符中含有a字符的信息
scan 'user', {COLUMNS => ['info', 'data'], FILTER => "(QualifierFilter(=,'substring:a'))"}

查询user表中列族为info，rk范围是[rk0001, rk0003)的数据
scan 'people', {COLUMNS => 'info', STARTROW => 'rk0001', ENDROW => 'rk0003'}

查询user表中row key以rk字符开头的
scan 'user',{FILTER=>"PrefixFilter('rk')"}

查询user表中指定范围的数据
scan 'user', {TIMERANGE => [1392368783980, 1392380169184]}

删除数据
删除user表row key为rk0001，列标示符为info:name的数据
delete 'people', 'rk0001', 'info:name'
删除user表row key为rk0001，列标示符为info:name，timestamp为1392383705316的数据
delete 'user', 'rk0001', 'info:name', 1392383705316


清空user表中的数据
truncate 'people'


修改表结构
首先停用user表（新版本不用）
disable 'user'

添加两个列族f1和f2
alter 'people', NAME => 'f1'
alter 'user', NAME => 'f2'
启用表
enable 'user'


###disable 'user'(新版本不用)
删除一个列族：
alter 'user', NAME => 'f1', METHOD => 'delete' 或 alter 'user', 'delete' => 'f1'

添加列族f1同时删除列族f2
alter 'user', {NAME => 'f1'}, {NAME => 'f2', METHOD => 'delete'}

将user表的f1列族版本号改为5
alter 'people', NAME => 'info', VERSIONS => 5
启用表
enable 'user'


删除表
disable 'user'
drop 'user'


get 'person', 'rk0001', {FILTER => "ValueFilter(=, 'binary:中国')"}
get 'person', 'rk0001', {FILTER => "(QualifierFilter(=,'substring:a'))"}
scan 'person', {COLUMNS => 'info:name'}
scan 'person', {COLUMNS => ['info', 'data'], FILTER => "(QualifierFilter(=,'substring:a'))"}
scan 'person', {COLUMNS => 'info', STARTROW => 'rk0001', ENDROW => 'rk0003'}

scan 'person', {COLUMNS => 'info', STARTROW => '20140201', ENDROW => '20140301'}
scan 'person', {COLUMNS => 'info:name', TIMERANGE => [1395978233636, 1395987769587]}
delete 'person', 'rk0001', 'info:name'

alter 'person', NAME => 'ffff'
alter 'person', NAME => 'info', VERSIONS => 10


get 'user', 'rk0002', {COLUMN => ['info:name', 'data:pic']}
```