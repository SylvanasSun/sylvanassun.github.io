---
layout:     post
title:      "如何搭建与维护一个Redis集群"
subtitle:   "redis-cluster"
date:       2016-06-27 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Database
    - Redis
tags:
    - Redis
---



![](http://ww3.sinaimg.cn/mw690/63503acbjw1f67o71v3m6j20av0640st.jpg)

### Redis集群架构

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f67o70ppf9j20dr0fudhb.jpg)

 1. 每一个`Redis`节点使用PING-PONG的形式互相通信。

 2. 当半数以上的节点fail时,则整个集群失效。

 3. 客户端不需要连接集群所有节点,只要连接集群中任意一个节点即可。

 4. `Redis`集群中内置了16384个哈希槽,当操作数据时,`Redis`会对key使用crc16算法算出一个结果,
    并把结果对16384取余,每个key都会对应0-16383之间的哈希槽,`Redis`会根据节点数量平均的将
    哈希槽映射到不同的节点上。

### Redis投票机制

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f67o717oqhj20cn0acmy4.jpg)

`Redis`集群中每一个节点都会参与投票,如果当半数以上的节点认为一个节点通信超时,则该节点fail。

当集群中任意节点的master(主机)挂掉,且这个节点没有slave(从机),则整个集群进入fail状态。

### 搭建Redis集群

#### 1.安装Ruby环境

因为redis-trib.rb脚本依赖ruby环境,所以需要先安装ruby。

```
yum install ruby 
yum install rubygems
```

安装ruby和redis的接口程序

```
gem install /usr/local/redis-3.0.0.gem
```

#### 2.搭建Redis集群

一个`Redis`集群最少需要3组主从机,即6个`Redis`。

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f67o71g231j20cz03lta6.jpg)

修改redis.conf配置文件,将集群开关开启。

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f67obgv67nj20an07gdim.jpg)

启动`Redis`实例

```
cd redis01
./redis-server redis.conf
cd ..
cd redis02
./redis-server redis.conf
cd ..
cd redis03
./redis-server redis.conf
cd ..
cd redis04
./redis-server redis.conf
cd ..
cd redis05
./redis-server redis.conf
cd ..
cd redis06
./redis-server redis.conf
cd ..
```

使用redis-trib.rb创建集群

```
./redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005  127.0.0.1:7006
```

#### 3.添加主节点

如果想添加一个port为7007的`Redis`实例,可以使用以下命令。

```
./redis-trib.rb add-node 127.0.0.1:7007 127.0.0.1:7001
```

当添加了一个主节点后,需要重新分配哈希槽。

```
./redis-trib.rb reshard 127.0.0.1:7001
```

#### 4.添加从节点

添加一个port为7008的`Redis`实例做为7007的从节点。

```
./redis-trib.rb add-node --slave --master-id cad9f7413ec6842c971dbcc2c48b4ca959eb5db4  127.0.0.1:7008 127.0.0.1:7001
```
命令格式: ./redis-trib.rb add-node --slave --master-id 主节点id 新加从节点的ip和端口 集群中节点的ip和端口(任意一个节点)。

主节点id可以在client中使用 **cluster nodes** 命令查询。

注意: 如果原来该结点在集群中的配置信息已经生成到cluster-config-file指定的配置文件中（如果cluster-config-file没有指定则默认为nodes.conf),则会报错:

```
[ERR] Node XXXXXX is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0
```

解决方法: 删除生成的配置文件 nodes.conf,再执行./redis-trib.rb add-node命令。

#### 5.删除节点

./redis-trib.rb del-node 要删除的节点的ip和端口 节点id
注意: 如果这个节点已经占有哈希槽,则无法删除,需要先将哈希槽分配出去。

### Jedis连接集群

使用Jedis连接集群需要先创建JedisCluster对象。代码如下:

```java
	public void test01(){
		Set<HostAndPort> nodes = new HashSet<HostAndPort>();
		nodes.add(new HostAndPort("192.168.145.134", 7001));
		nodes.add(new HostAndPort("192.168.145.134", 7002));
		nodes.add(new HostAndPort("192.168.145.134", 7003));
		nodes.add(new HostAndPort("192.168.145.134", 7004));
		nodes.add(new HostAndPort("192.168.145.134", 7005));
		nodes.add(new HostAndPort("192.168.145.134", 7006));
		// 创建JedisCluster
		JedisCluster jedisCluster = new JedisCluster(nodes);
		// 操作redis
		String set = jedisCluster.set("hello", "helloWorld");
		String hello = jedisCluster.get("hello");
		System.out.println(set);
		System.out.println(hello);
		// 关闭JedisCluster
		jedisCluster.close();
	}
```

在Spring容器中维护:

```
    <bean class="redis.clients.jedis.JedisCluster">
		<constructor-arg name="nodes">
			<set>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.145.134"/>
					<constructor-arg name="port" value="7001"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.145.134"/>
					<constructor-arg name="port" value="7002"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.145.134"/>
					<constructor-arg name="port" value="7003"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.145.134"/>
					<constructor-arg name="port" value="7004"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.145.134"/>
					<constructor-arg name="port" value="7005"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.145.134"/>
					<constructor-arg name="port" value="7006"/>
				</bean>
			</set>
		</constructor-arg>
	</bean>
```
