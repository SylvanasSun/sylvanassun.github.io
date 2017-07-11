---
layout:     post
title:      "SolrCloud初体验"
subtitle:   "SolrCloud快速入门"
date:       2016-06-30 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - 全文检索
    - Solr
tags:
    - Solr
    - 全文检索引擎
---



![](http://ww4.sinaimg.cn/mw690/63503acbjw1f67ocs5srhj207g05m3yl.jpg)

### 什么是SolrCloud

SolrCloud是基于Solr和Zookeeper的分布式搜索方案,它的主要思想是使用Zookeeper作为集群的配置信息中心。

### SolrCloud的特点

#### 1.近实时搜索

立即推送式的replication（也支持慢推送）。可以在秒内检索到新加入索引。

#### 2.自动容错

SolrCloud对索引分片,并对每个分片创建多个Replication。每个Replication都可以对外提供服务。一个Replication挂掉不会影响索引服务。更强大的是，它还能自动的在其它机器上帮你把失败机器上的索引Replication重建并投入使用。

#### 3.查询时自动负载均衡

SolrCloud索引的多个Replication可以分布在多台机器上,均衡查询压力。如果查询压力大，可以通过扩展机器，增加Replication来减缓。

#### 4.集中式的配置信息

SolrCloud可以将配置文件上传到Zookeeper,由Zookeeper对配置文件进行管理。

### 结构分析

为了减少处理压力,SolrCloud需要由多台服务器共同完成索引和搜索。

#### 1.实现思路

SolrCloud将索引数据进行分片(Shard),每个分片由多台服务器共同完成。

#### 2.结构

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f67oct3xegj20rx0qfdht.jpg)

**物理结构**
SolrCloud由三个Solr服务器组成,每个Solr服务器包含2个Core。

**逻辑结构**
一个Collection包含2个Shard,每个Shard由3个core组成(一个Leader,两个Replication)。

**Collection**
Collection是一个在逻辑意义上完整的索引结构,它常常被划分为一个或多个Shard分片,它们使用相同的配置信息。如果Shard数超过一个，它就是分布式索引，SolrCloud让你通过Collection名称引用它，而不需要关心分布式检索时需要使用的和Shard相关参数。

**Core**
一个Solr中包含一个或者多个Solr Core，每个Solr Core可以独立提供索引和查询功能，每个Solr Core对应一个索引或者Collection的Shard，Solr Core的提出是为了增加管理灵活性和共用资源。在SolrCloud中有个不同点是它使用的配置是在Zookeeper中的，传统的Solr core的配置文件是在磁盘上的配置目录中。

**Shard**
Collection的逻辑分片。每个Shard被化成一个或者多个replicas，通过选举确定哪个是Leader。

**Replication**
在master-slave结构中,Replication是一个从节点,同一个Shard下主从节点存储的数据是一致的。

**Leader**
在master-slave结构中,Leader是一个主节点,Leader是赢得选举的Replication。选举可以发生在任何时间，但是通常它们仅在某个Solr实例发生故障时才会触发。当索引documents时，SolrCloud会传递它们到此Shard对应的Leader，Leader再分发它们到全部Shard的Replication。

### SolrCloud的搭建

#### 1.Zookeeper
SolrCloud需要Zookeeper进行管理,所以需要先安装Zookeeper。

- .解压缩zookeeper.tar.gz,并复制出3个Zookeeper实例。

- .进入zookeeper01目录,创建一个data文件夹,并在data中创建一个myid文件,内容为1(其他Zookeeper实例为2和3)。

- 进入conf文件夹,将zoo_sample.cfg改名为zoo.cfg

- vim zoo.cfg 修改dataDir=data文件夹所在的目录,
 添加:
 server.myid的值=ip:每个Zookeeper服务器之间的通讯端口:Zookeeper与其他应用的通讯端口。(每个Zookeeper实例都需要添加这行内容)
 例如:
 
 ![](http://ww2.sinaimg.cn/mw690/63503acbjw1f67ocpgc1zj20d80cwdjc.jpg)

#### 2.Solr实例

 - 安装一个单机的Solr实例,并复制成4份,分别对应4个SolrHome。

 - 修改SolrHome的solr.xml文件
 
 ![](http://ww3.sinaimg.cn/mw690/63503acbjw1f67ocqpcfqj20he097acw.jpg)

 - 将配置文件上传到Zookeeper,当配置文件发生改变时,需要重新上传。
 

 
	 java -classpath .:/usr/local/solr-cloud/solr-lib/* org.apache.solr.cloud.ZkCLI -cmd upconfig -zkhost 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183 -confdir /usr/local/solr-cloud/solrhome01/collection1/conf  -confname solr-conf
	
	-cmd upconfig 上传配置文件命令
	-zkhost Zookeeper集群的ip与端口
	-confdir 配置文件的目录
	-confname 上传到Zookeeper后的文件夹名称
	
	其中参数/usr/local/solr-cloud/solr-lib/可以自己创建，内容如下：
	        复制tomcat/webapps/solr/WEB-INF/lib下所有jar包
	        复制example/lib/ext下所有jar包
	        复制example/resources/log4j.properties

 
 
 - 通知Solr实例Zookeeper的地址,需要修改tomcat/bin/catalina.sh
 添加一行:JAVA_OPTS="-DzkHost=Zookeeper集群的地址列表"
 
 ![](http://ww3.sinaimg.cn/mw690/63503acbjw1f67ocr82b3j20ig04ptas.jpg)

#### 3.设置Shard

```
http://web容器/solr/admin/collections?action=CREATE&name=Collection名称&numShards=Shard个数&replicationFactor=Replication个数
```

例:

```
http://192.168.145.150:8080/solr/admin/collections?action=CREATE&name=collection2&numShards=2&replicationFactor=2
上面的命令为 创建一个name为collection2的Collection,并分成了2个Shard,每个Shard有2个Replication
```

删除一个Collection

```
http://192.168.145.150:8080/solr/admin/collections?action=DELETE&name=collection1
上面的命令为 删除一个name为collection1的Collection
```

### Spring整合SolrCloud

```
<!-- SolrCloud -->
	<bean id="cloudSolrServer" class="org.apache.solr.client.solrj.impl.CloudSolrServer">
		<constructor-arg name="zkHost"
			value="192.168.145.136:2181,192.168.145.136:2182,192.168.145.136:2183" />
		<!-- 设置默认搜索的Collection -->
		<property name="defaultCollection" value="collection2" />
	</bean>
```
