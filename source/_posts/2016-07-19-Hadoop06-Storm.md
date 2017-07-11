---
layout:     post
title:      "Hadoop学习笔记(6)-Storm"
subtitle:   "Storm QuickStart"
date:       2016-07-19 18:00
author:     "Sylvanas Sun"
header-img: "img/StormPost.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 后端开发
    - Hadoop
    - 大数据
---


![](http://ww3.sinaimg.cn/mw690/63503acbjw1f5z3iwds07j207g03nmxf.jpg)

### 概述

&nbsp;&nbsp;Storm是一个开源的分布式实时计算系统，可以简单、可靠的处理大量的数据流。被称作“实时的hadoop”。Storm有很多使用场景：如实时分析，在线机器学习，持续计算， 分布式RPC，ETL等等。Storm支持水平扩展，具有高容错性，保证每个消息都会得到处理，而且处理速度很快。Storm的部署和运维都很便捷，而且更为重要的是可以使用任意编程语言来开发应用。

### Storm的特点

**简单的编程模型**

&nbsp;&nbsp;在大数据处理方面相信大家对hadoop已经耳熟能详，基于Google Map/Reduce来实现的Hadoop为开发者提供了map、reduce原语，使并行批处理程序变得非常地简单和优美。

&nbsp;&nbsp;同样，Storm也为大数据 的实时计算提供了一些简单优美的原语，这大大降低了开发并行实时处理的任务的复杂性，帮助你快速、高效的开发应用。

**水平扩展**

&nbsp;&nbsp;在Storm集群中真正运行topology的主要有三个实体：工作进程、线程和任务。Storm集群中的每台机器上都可以运行多个工作进程，每个 工作进程又可创建多个线程，每个线程可以执行多个任务，任务是真正进行数据处理的实体，我们开发的spout、bolt就是作为一个或者多个任务的方式执行的。

&nbsp;&nbsp;计算任务在多个线程、进程和服务器之间并行进行，支持灵活的水平扩展。

**支持多种编程语言**

&nbsp;&nbsp;你可以在Storm之上使用各种编程语言。默认支持Clojure、Java、Ruby和Python。要增加对其他语言的支持，只需实现一个简单的Storm通信协议即可。

**高可靠性**

&nbsp;&nbsp;Storm保证每个消息至少能得到一次完整处理。任务失败时，它会负责从消息源重试消息。

&nbsp;&nbsp;spout发出的消息后续可能会触发产生成千上万条消息，可以形象的理解为一棵消息树，其中spout发出的消息为树根，Storm会跟踪这棵消息树的处理情况，只有当这棵消息树中的所有消息都被处理了，Storm才会认为spout发出的这个消息已经被“完全处理”。如果这棵消息树中的任何一个消息处理失败了，或者整棵消息树在限定的时间内没有“完全处理”，那么spout发出的消息就会重发。

**高容错性**

&nbsp;&nbsp;Storm会管理工作进程和节点的故障。

&nbsp;&nbsp;如果在消息处理过程中出了一些异常，Storm会重新安排这个出问题的处理单元。Storm保证一个处理单元永远运行（除非你显式杀掉这个处理单元）。

&nbsp;&nbsp;当然，如果处理单元中存储了中间状态，那么当处理单元重新被Storm启动的时候，需要应用自己处理中间状态的恢复。

**本地模式**

&nbsp;&nbsp;Storm有一个“本地模式”，可以在处理过程中完全模拟Storm集群。这让你可以快速进行开发和单元测试。

### Storm架构

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f5za3m6lqaj20go0f5gn1.jpg)

&nbsp;&nbsp;Storm集群由一个主节点和多个工作节点组成。主节点运行了一个名为“Nimbus”的守护进程，用于分配代码、布置任务及故障检测。每个工作节 点都运行了一个名为“Supervisor”的守护进程，用于监听工作，开始并终止工作进程。Nimbus和Supervisor都能快速失败，而且是无状态的，这样一来它们就变得十分健壮，两者的协调工作是由ApacheZooKeeper来完成的。

#### Stream

&nbsp;&nbsp;Stream是一个数据流的抽象。这是一个没有边界的Tuple序列,而这些Tuple序列会以一种分布式的方式并行地创建和处理。

&nbsp;&nbsp;对消息流的定义主要就是对消息流里面的tuple 进行定义，为了更好地使用tuple，需要给tuple 里的每个字段取一个名字，并且不同的tuple 字段对应的类型要相同，即两个tuple 的第一个字段类型相同，第二个字段类型相同，但是第一个字段和第二个字段的类型可以不同。默认情况下，tuple 的字段类型可以为integer、long、short、byte、string、double、float、boolean 和byte array 等基本类型，也可以自定义类型，只需要实现相应的序列化接口。

&nbsp;&nbsp;每一个消息流在定义的时候需要被分配一个id，最常见的消息流是单向的消息流，在Storm 中OutputFieldsDeclarer 定义了一些方法，让你可以定义一个Stream 而不用指定这个id。在这种情况下，这个Stream 会有个默认的id: 1。

#### Topologies

&nbsp;&nbsp;**Topology是由Stream Grouping连接起来的Spout和Bolt节点网络。**

&nbsp;&nbsp;在 Storm 中，一个实时计算应用程序的逻辑被封装在一个称为Topology 的对象中，也称为计算拓扑。Topology 有点类似于Hadoop 中的MapReduce Job，但是它们之间的关键区别在于，一个MapReduce Job 最终总是会结束的，然而一个Storm 的Topology 会一直运行。在逻辑上，一个Topology 是由一些Spout（消息的发送者）和Bolt（消息的处理者）组成图状结构，而链接Spouts 和Bolts 的则是Stream Groupings。

#### Spouts&Bolts

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f5za9p00fyj20mo0a975d.jpg)

**Spouts**

&nbsp;&nbsp;Spouts 是Storm集群中一个计算任务（Topology）中消息流的生产者，Spouts一般是从别的数据源（例如，数据库或者文件系统）加载数据，然后向Topology中发射消息。

&nbsp;&nbsp;Spouts即可以是可靠的,也可以是不可靠的。

&nbsp;&nbsp;在一个Topology中存在两种Spouts，一种是可靠的Spouts，一种是非可靠的Spouts，可靠的Spouts 在一个tuple 没有成功处理的时候会重新发射该tuple，以保证消息被正确地处理。不可靠的Spouts 在发射一个tuple 之后，不会再重新发射该tuple，即使该tuple 处理失败。每个Spouts 都可以发射多个消息流，要实现这样的效果，可以使用OutFieldsDeclarer.declareStream 来定义多个Stream，然后使用SpoutOutputCollector 来发射指定的Stream。

&nbsp;&nbsp;在Storm 的编程接口中，Spout 类最重要的方法是nextTuple()方法，使用该方法可以发射一个消息tuple 到Topology 中，或者简单地直接返回，如果没有消息要发射。需要注意的是，nextTuple 方法的实现不能阻塞Spout，因为Storm在同一线程上调用Spout 的所有方法。Spout 类的另外两个重要的方法是ack()和fail()，一个tuple 被成功处理完成后，ack()方法被调用，否则就调用fail()方法。注意，只有对于可靠的Spout，才会调用ack()和fail()方法。

**Bolts**

&nbsp;&nbsp;所有消息处理的逻辑都在Bolt 中完成，在Bolt 中可以完成如过滤、分类、聚集、计算、查询数据库等操作。Bolt 可以做简单的消息处理操作，例如，Bolt 可以不做任何操作，只是将接收到的消息转发给其他的Bolt。Bolt 也可以做复杂的消息流的处理，从而需要很多个Bolt。在实际使用中，一条消息往往需要经过多个处理步骤，例如，计算一个班级中成绩在前十名的同学，首先需要对所有同学的成绩进行排序，然后在排序过的成绩中选出前十名的
成绩的同学。所以在一个Topology 中，往往有很多个Bolt，从而形成了复杂的流处理网络。

&nbsp;&nbsp;Bolts可以发射多条消息流。

 1. 使用OutputFieldsDeclarer.declareStream定义Stream。
 2. 使用OutputCollector.emit来选择要发射的Stream。

&nbsp;&nbsp;Bolts的主要方法是execute。

&nbsp;&nbsp;Bolts以Tuple作为输入,使用OutputCollector来发射Tuple,通过调用OutputCollector.ack()通知这个Tuple的发射者Spout。

&nbsp;&nbsp;Bolts一般流程。

&nbsp;&nbsp;处理一个输入Tuple,发射0个或多个Tuple,然后调用ack()通知Storm自己已经处理过这个Tuple了。Storm提供了一个IBasicBolt会自动调用ack()。

#### Stream Groupings

&nbsp;&nbsp;定义一个 Topology 的其中一步是定义每个Bolt 接收什么样的流作为输入。Stream Grouping 就是用来定义一个Stream 应该如何分配给Bolts 上面的多个Tasks。

&nbsp;&nbsp;Storm里有7种类型的Stream Grouping。

 1. Shuffle Grouping 随机分组,随机派发Stream里面的Tuple,保证每个Bolt接收到的Tuple数量大致相同。
 2. Fields Grouping 按字段分组,以id举例。具有相同id的Tuple会被分到相同的Bolt中的一个Task,而不同id的Tuple会被分到不同的Bolt中的Task。
 3. All Grouping 广播,对于每一个Tuple,所有的Bolts都会收到。
 4. Global Grouping 全局分组,这个Tuple被分配到Storm中的一个Bolt的其中一个Task。具体一点就是分配给id值最低的那个Task。
 5. Non Grouping 不分组,Stream不关心到底谁会收到它的Tuple。目前这种分组和Shuffle Grouping是一样的效果,有一点不同的是Storm会把这个Bolt放到这个Bolt的订阅者同一个线程中去执行。
 6. Direct Grouping 直接分组,这是一种比较特别的分组方法,用这种分组意味着消息的发送者指定由消息接收者的哪个Task处理这个消息。只有被声明为Direct Stream的消息流可以声明这种分组方法。而且这种消息Tuple必须使用emitDirect方法来发射。消息处理者可以通过TopologyContext来获取处理它的消息的Task的id(OutputCollector.emit方法也会返回Task的id)。
 7. Local or Shuffle Grouping 如果目标Bolt有一个或者多个Task在同一个工作进程中,Tuple将会被随机发射给这些Tasks。否则,和普通的Shuffle Grouping行为一致。

#### Workers

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f5zdzh1eo9j20bq08h0t6.jpg)

 1. 每个Supervisor中运行着多个Workers进程。
 2. 每个Worker进程中运行着多个Executor线程。
 3. 每个Executor线程中运行着若干个相同的Task(Spout/Bolt)。

&nbsp;&nbsp;一个 Topology 可能会在一个或者多个工作进程里面执行，每个工作进程执行整个Topology 的一部分。比如，对于并行度是300 的Topology 来说，如果我们使用50 个工作进程来执行，那么每个工作进程会处理其中的6 个Tasks（其实就是每个工作进程里面分配6 个线程）。Storm 会尽量均匀地把工作分配给所有的工作进程。

#### Task

&nbsp;&nbsp;在 Storm 集群上，每个Spout 和Bolt 都是由很多个Task 组成的，每个Task对应一个线程，流分组策略就是定义如何从一堆Task 发送tuple 到另一堆Task。在实现自己的Topology 时可以调用TopologyBuilder.setSpout() 和TopBuilder.setBolt()方法来设置并行度，也就是有多少个Task。

### Storm安装部署

 1. 安装jdk。
 2. 搭建Zookeeper集群。
 3. 下载并解压Storm。
 4. 修改storm.yaml配置文件。
    - storm.zookeeper.servers: Storm集群使用的Zookeeper集群地址。例如:
      storm.zookeeper.servers:
      -"192.168.145.141"
      -"192.168.145.142"
    - 如果Zookeeper没有使用默认端口,那么还需要修改storm.zookeeper.port。
    - storm.local.dir Nimbus和Supervisor进程用于存储少量状态,如jars、confs等的本地磁盘目录,需要提前创建该目录并给予足够的访问权限。然后在storm.yaml中配置该目录,例如:
      storm.local.dir:"/home/application/storm/workdir"

#### 注意事项

&nbsp;&nbsp;启动Storm后台进程时,需要对conf/storm.yaml配置文件中设置的storm.local.dir目录具有写权限。

&nbsp;&nbsp;Storm后台进程被启动时,将在Storm安装目录下的logs/子目录下生成各个进程的日志文件。

&nbsp;&nbsp;Storm UI必须和Storm Nimbus部署在同一台机器上,否则UI无法正常工作,因为UI进程会检查本机是否存在Nimbus链接。

### 常用命令

| 命令描述       | 格式             | 例子             |
| -------------- | ---------------- | ---------------- |
| 启动Nimbus     | storm nimbus     | storm nimbus     |
| 启动Supervisor | storm supervisor | storm supervisor |
| 启动UI         | storm ui         | storm ui         |

#### 提交Topologies

**格式** 

storm jar 【jar路径】 【拓扑包名.拓扑类名】【stormIP地址】【storm端口】【拓扑名称】【参数】

**Example**

```
storm jar /home/storm/hello.jar
storm.hello.WordCountTopology wordcountTop
提交hello.jar到远程集群,并启动wordcountTop拓扑
```

#### 停止Topologies

**格式**

storm kill [拓扑名称]

**Example**

```
storm kill wordcountTop
```

### API

#### Spouts

&nbsp;&nbsp; Spout是Stream的消息产生源， Spout组件的实现可以通过继承BaseRichSpout类或者其他*Spout类来完成，也可以通过实现IRichSpout接口来实现。


**open**

&nbsp;&nbsp;当一个Task被初始化的时候会调用open()。一般都会在此方法中对发送Tuple的对象SpoutOutputCollector和配置对象TopologyContext初始化。

**getComponentConfiguration**

&nbsp;&nbsp;此方法用于声明针对当前组件的特殊的Configuration配置。

**nextTuple**

&nbsp;&nbsp;这是Spout类中最重要的一个方法。发射一个Tuple到Topology都是通过这个方法来实现的。

**declareOutputFields**

&nbsp;&nbsp;此方法用于声明当前Spout的Tuple发送流。Stream的定义是通过OutputFieldsDeclare.declare方法完成的,其中的参数包括了发送的Fields。

&nbsp;&nbsp;另外，除了上述几个方法之外，还有ack、fail和close方法等。

&nbsp;&nbsp;Storm在监测到一个Tuple被成功处理之后会调用ack方法，处理失败会调用fail方法。这两个方法在BaseRichSpout等类中已经被隐式的实现了。

#### Bolts

&nbsp;&nbsp; Bolt类接收由Spout或者其他上游Bolt类发来的Tuple，对其进行处理。Bolt组件的实现可以通过继承BasicRichBolt类或者IRichBolt接口来完成。

**prepare**

&nbsp;&nbsp;此方法与Spouts的open方法类似,为Bolt提供了OutputCollector,用来从Bolt中发射Tuple。Bolt中Tuple的发射可以在prepare中、execute中、cleanup等方法中进行,一般都是在execute中。

**getComponentConfiguration**

&nbsp;&nbsp;与Spouts类似。

**execute**

&nbsp;&nbsp;  这是Bolt中最关键的一个方法，对于Tuple的处理都可以放到此方法中进行。具体的发送也是在execute中通过调用emit方法来完成的。

&nbsp;&nbsp;emit有两种情况，一种是emit方法中有两个参数，另一个种是有一个参数。

 1. emit有一个参数：此唯一的参数是发送到下游Bolt的Tuple，此时，由上游发来的旧的Tuple在此隔断，新的Tuple和旧的Tuple不再属于同一棵Tuple树。新的Tuple另起一个新的Tuple树。
 2. emit有两个参数：第一个参数是旧的Tuple的输入流，第二个参数是发往下游Bolt的新的Tuple流。此时，新的Tuple和旧的Tuple是仍然属于同一棵Tuple树，即，如果下游的Bolt处理Tuple失败，则会向上传递到当前Bolt，当前Bolt根据旧的Tuple流继续往上游传递，申请重发失败的Tuple。保证Tuple处理的可靠性。

**declareOutputFields**

&nbsp;&nbsp;用于声明当前Bolt发送的Tuple中包含的字段。

#### Topology Example

```java
public class RandomWordSpout extends BaseRichSpout {

	// 初始化数据字典
	private final static String[] words = { "java", "c", "c++", "c#", "python", "go", "javascript",
			"swift" };

	private SpoutOutputCollector collector;

	@Override
	public void nextTuple() {
		Random random = new Random();
		// 获取随机的单词
		String word = words[random.nextInt(words.length)];
		// 发射消息
		this.collector.emit(new Values(word));
		// 休息2秒
		Utils.sleep(2000);
	}

	@Override
	public void open(Map arg0, TopologyContext arg1, SpoutOutputCollector collector) {
		this.collector = collector;
	}

	@Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {
		// 声明字段名
		declarer.declare(new Fields("initName"));
	}

}
```

```java
public class UpperBolt extends BaseBasicBolt {

	@Override
	public void execute(Tuple tuple, BasicOutputCollector collector) {
		// 获得上个bolt传入的initName
		String initName = tuple.getString(0);
		// 将initName转为大写
		String upperCase = initName.toUpperCase();
		// 发射消息
		collector.emit(new Values(upperCase));
	}

	@Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {
		declarer.declare(new Fields("upperName"));
	}

}
```

```java
public class PrefixBolt extends BaseBasicBolt {

	private FileWriter fileWriter;

	@Override
	public void prepare(Map stormConf, TopologyContext context) {
		// 初始化fileWriter
		try {
			this.fileWriter = new FileWriter("/home/storm/output/" + UUID.randomUUID());
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	@Override
	public void execute(Tuple tuple, BasicOutputCollector collector) {
		String upperName = tuple.getString(0);
		// 添加前缀
		String finalName = "hello-" + upperName;
		// write
		try {
			this.fileWriter.write(finalName);
			this.fileWriter.write("\n");
			this.fileWriter.flush();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	@Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {

	}

}
```

```java
public class TopologyMain {

	public static void main(String[] args) throws Exception {
		TopologyBuilder topologyBuilder = new TopologyBuilder();
		// 设置Spout
		topologyBuilder.setSpout("randomWordSpout", new RandomWordSpout());
		// 设置Bolt
		topologyBuilder.setBolt("upperBolt", new UpperBolt()).shuffleGrouping("randomWordSpout");
		topologyBuilder.setBolt("prefixBolt", new PrefixBolt()).shuffleGrouping("upperBolt");

		Config config = new Config();
		// 设置Workers数量
		config.setNumWorkers(4);
		config.setDebug(true);
		// 提交Topology
		StormSubmitter.submitTopology("randomTopo", config, topologyBuilder.createTopology());
	}

}
```