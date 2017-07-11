---
layout:     post
title:      "Hadoop学习笔记(2)-Mapreduce"
subtitle:   "Mapreduce QuickStart"
date:       2016-07-14 18:00
author:     "Sylvanas Sun"
header-img: "img/HadoopPost.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 后端开发
    - Hadoop
    - 大数据
---



![](http://ww4.sinaimg.cn/mw690/63503acbjw1f5w446w8pcj20bp08rweu.jpg)

### 什么是MapReduce 

&nbsp;&nbsp;MapReduce是一种分布式计算模型，由Google提出，主要用于搜索领域，解决海量数据的计算问题。

&nbsp;&nbsp;MapReduce是处理大量半结构化数据集合的编程模型。编程模型是一种处理并结构化特定问题的方式。例如，在一个关系数据库中，使用一种集合语言执行查询，如SQL。告诉语言想要的结果，并将它提交给系统来计算出如何产生计算。还可以用更传统的语言(C++，Java)，一步步地来解决问题。这是两种不同的编程模型，MapReduce就是另外一种。

&nbsp;&nbsp;MapReduce和Hadoop是相互独立的，实际上又能相互配合工作得很好。

### Yarn概述

&nbsp;&nbsp;Yarn是一个分布式的资源管理系统，用以提高分布式的集群环境下的资源利用率，这些资源包括内存、IO、网络、磁盘等。其产生的原因是为了解决原MapReduce框架的不足。最初MapReduce的committer们还可以周期性的在已有的代码上进行修改，可是随着代码的增加以及原MapReduce框架设计的不足，在原MapReduce框架上进行修改变得越来越困难，所以MapReduce的committer们决定从架构上重新设计MapReduce,使下一代的MapReduce(MRv2/Yarn)框架具有更好的扩展性、可用性、可靠性、向后兼容性和更高的资源利用率以及能支持除了MapReduce计算框架外的更多的计算框架。

### 原MapReduce架构的不足

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f5w4495s9rj20sa0foac1.jpg)

 - JobTracker是集群事务的集中处理点，存在单点故障。
 - JobTracker需要完成的任务太多，既要维护job的状态又要维护job的task的状态，造成过多的资源消耗。
 - 在taskTracker端，用map/reduce task作为资源的表示过于简单，没有考虑到CPU、内存等资源情况，当把两个需要消耗大内存的task调度到一起，很容易出现OOM(Out Of Memory内存不足)。
 - 把资源强制划分为map/reduce slot,当只有map task时，reduce slot不能用；当只有reduce task时，map slot不能用，容易造成资源利用不足。

### MRv2/Yarn工作流程

#### Yarn架构

&nbsp;&nbsp;Yarn/MRv2最基本的想法是将原JobTracker主要的资源管理和job调度/监视功能分开作为两个单独的守护进程。

&nbsp;&nbsp;有一个全局的ResourceManager(RM)和每个Application有一个ApplicationMaster(AM)，Application相当于map-reduce job或者DAG jobs。

&nbsp;&nbsp;ResourceManager和NodeManager(NM)组成了基本的数据计算框架。ResourceManager协调集群的资源利用，任何client或者运行着的applicatitonMaster想要运行job或者task都得向RM申请一定的资源。ApplicatonMaster是一个框架特殊的库，对于MapReduce框架而言有它自己的AM实现，用户也可以实现自己的AM，在运行的时候，AM会与NM一起来启动和监视tasks。 

**ResourceManager**

ResourceManager作为资源的协调者有两个主要的组件：Scheduler和ApplicationsManager(AsM)。

Scheduler负责分配最少但满足application运行所需的资源量给Application。Scheduler只是基于资源的使用情况进行调度，并不负责监视/跟踪application的状态，当然也不会处理失败的task。RM使用resource container概念来管理集群的资源，resource container是资源的抽象，每个container包括一定的内存、IO、网络等资源，不过目前的实现只包括内存一种资源。

ApplicationsManager负责处理client提交的job以及协商第一个container以供applicationMaster运行，并且在applicationMaster失败的时候会重新启动applicationMaster。下面阐述RM具体完成的一些功能。

 1. 资源调度：Scheduler从所有运行着的application收到资源请求后构建一个全局的资源分配计划，然后根据application特殊的限制以及全局的一些限制条件分配资源。
 2. 资源监视：Scheduler会周期性的接收来自NM的资源使用率的监控信息，另外applicationMaster可以从Scheduler得到属于它的已完成的container的状态信息。
 3. Application提交：

    - client向AsM获得一个applicationIDclient将application定义以及需要的jar包.
    - client将application定义以及需要的jar包文件等上传到hdfs的指定目录，由yarn-site.xml的yarn.app.mapreduce.am.staging-dir指定.
    - client构造资源请求的对象以及application的提交context发送给AsM.
    - AsM接收application的提交context.
    - AsM根据application的信息向Scheduler协商一个Container供applicationMaster运行，然后启动applicationMaster.
    - 向该container所属的NM发送launchContainer信息启动该container,也即启动applicationMaster、AsM向client提供运行着的AM的状态信息.

 4.  AM的生命周期：AsM负责系统中所有AM的生命周期的管理。AsM负责AM的启动，当AM启动后，AM会周期性的向AsM发送heartbeat，默认是1s，AsM据此了解AM的存活情况，并且在AM fail时负责重启AM，若是一定时间过后(默认10分钟)没有收到AM的heartbeat，AsM就认为该AM已经fail。

**NodeManager**

&nbsp;&nbsp;NM主要负责启动RM分配给AM的container以及代表AM的container，并且会监视container的运行情况。在启动container的时候，NM会设置一些必要的环境变量以及将container运行所需的jar包、文件等从hdfs下载到本地，也就是所谓的资源本地化；当所有准备工作做好后，才会启动代表该container的脚本将程序启动起来。启动起来后，NM会周期性的监视该container运行占用的资源情况，若是超过了该container所声明的资源量，则会kill掉该container所代表的进程。

&nbsp;&nbsp;NM还提供了一个简单的服务以管理它所在机器的本地目录。Applications可以继续访问本地目录即使那台机器上已经没有了属于它的container在运行。例如，Map-Reduce应用程序使用这个服务存储map output并且shuffle它们给相应的reduce task。

&nbsp;&nbsp;NM上还可以扩展自己的服务，yarn提供了一个yarn.nodemanager.aux-services的配置项，通过该配置，用户可以自定义一些服务，例如Map-Reduce的shuffle功能就是采用这种方式实现的。

NM在本地为每个运行着的application生成如下的目录结构：

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f5w44b7lxfj208u05dt9v.jpg)

Container目录下的目录结构如下： 

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f5w44bjrc6j206m05jq3d.jpg)

&nbsp;&nbsp;在启动一个container的时候，NM就执行该container的default_container_executor.sh，该脚本内部会执行launch_container.sh。launch_container.sh会先设置一些环境变量，最后启动执行程序的命令。对于MapReduce而言，启动AM就执行org.apache.hadoop.mapreduce.v2.app.MRAppMaster；启动map/reduce task就执行org.apache.hadoop.mapred.YarnChild。 

**ApplicationMaster**

&nbsp;&nbsp;ApplicationMaster是一个框架特殊的库，对于Map-Reduce计算模型而言有它自己的ApplicationMaster实现，对于其他的想要运行在yarn上的计算模型而言，必须得实现针对该计算模型的ApplicationMaster用以向RM申请资源运行task，比如运行在yarn上的spark框架也有对应的ApplicationMaster实现，归根结底，yarn是一个资源管理的框架，并不是一个计算框架，要想在yarn上运行应用程序，还得有特定的计算框架的实现。

#### 工作流程

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f5w48pm9byj20ui0kl0uy.jpg)

 1. JobClient向ResourceManager(AsM)申请提交一个job。
 2. RM返回jobId和job提交路径。
 3. JobClient提交job相关的文件。
 4. 向RM汇报提交完成。
 5. RM将job写入Job Queue。
 6. NodeManager(NM)向Job Queue领取任务。
 7. ApplicationMaster(AM)启动,向RM进行注册。
 8. RM向AM返回资源信息。
 9. AM启动map。
 10. 当所有map任务完成后,AM启动reduce。 
 11. AM监视运行着的task直到完成,当task失败时,申请新的container运行失败的task。
 12. 当每个map/reduce task完成后,AM运行MR OutputCommitter的cleanup 代码，进行一些收尾工作。
 13. 当所有的map/reduce完成后,AM运行OutputCommitter的必要的job commit或者abort APIs。
 14. AM注销自己。

### Shuffle过程

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f5w449wf5hj20mv0aodh4.jpg)

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f5w44an439j20k10g5mzg.jpg)

#### Map

 1. 每个输入分片会让一个map任务来处理，默认情况下，以HDFS的一个块的大小（默认为64M）为一个分片，当然我们也可以设置块的大小。map输出的结果会暂且放在一个环形内存缓冲区中（该缓冲区的大小默认为100M，由io.sort.mb属性控制），当该缓冲区快要溢出时（默认为缓冲区大小的80%，由io.sort.spill.percent属性控制），会在本地文件系统中创建一个溢出文件，将该缓冲区中的数据写入这个文件。

 2. 在写入磁盘之前，线程首先根据reduce任务的数目将数据划分为相同数目的分区，也就是一个reduce任务对应一个分区的数据。这样做是为了避免有些reduce任务分配到大量数据，而有些reduce任务却分到很少数据，甚至没有分到数据的尴尬局面。其实分区就是对数据进行hash的过程。然后对每个分区中的数据进行排序，如果此时设置了Combiner，将排序后的结果进行Combia操作，这样做的目的是让尽可能少的数据写入到磁盘。

 3. 当map任务输出最后一个记录时，可能会有很多的溢出文件，这时需要将这些文件合并。合并的过程中会不断地进行排序和combia操作，目的有两个：
 
    - 尽量减少每次写入磁盘的数据量；
    - 尽量减少下一复制阶段网络传输的数据量。最后合并成了一个已分区且已排序的文件。

    为了减少网络传输的数据量，这里可以将数据压缩，只要将mapred.compress.map.out设置为true就可以了。

 4. 将分区中的数据拷贝给相对应的reduce Task。有人可能会问：分区中的数据怎么知道它对应的reduce是哪个呢？其实map任务一直和其父TaskTracker保持联系，而TaskTracker又一直和JobTracker保持心跳。所以JobTracker中保存了整个集群中的宏观信息。只要reduce任务向JobTracker获取对应的map输出位置就ok了哦。
 
#### Reduce

 1. Reduce会接收到不同map任务传来的数据，并且每个map传来的数据都是有序的。如果reduce端接受的数据量相当小，则直接存储在内存中（缓冲区大小由mapred.job.shuffle.input.buffer.percent属性控制，表示用作此用途的堆空间的百分比），如果数据量超过了该缓冲区大小的一定比例（由mapred.job.shuffle.merge.percent决定），则对数据合并后溢写到磁盘中。

 2. 随着溢写文件的增多，后台线程会将它们合并成一个更大的有序的文件，这样做是为了给后面的合并节省时间。其实不管在map端还是reduce端，MapReduce都是反复地执行排序，合并操作。

 3. 合并的过程中会产生许多的中间文件（写入磁盘了），但MapReduce会让写入磁盘的数据尽可能地少，并且最后一次合并的结果并没有写入磁盘，而是直接输入到reduce函数。

### WordCount案例

#### Mapper

```java
public class MyWordCountMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		// 读取一行的value
		String line = value.toString();
		// 按照规则切分
		String[] words = line.split(" ");
		// 按照<单词,1>的格式输出
		for (String word : words) {
			context.write(new Text(word), new LongWritable(1));
		}
	}
}
```

#### Reducer

```java
public class MyWordCountReducer extends Reducer<Text, LongWritable, Text, LongWritable> {

	@Override
	protected void reduce(Text key, Iterable<LongWritable> values, Context context)
			throws IOException, InterruptedException {
		// 初始化计数器
		long count = 0;
		// 迭代values,累加计数器计算出总次数
		for (LongWritable value : values) {
			count += value.get();
		}
		// 输出<单词,总次数>
		context.write(key, new LongWritable(count));
	}
}
```

#### Main

```java
public class MyWordCountDriver {

	public static void main(String[] args)
	    throws IOException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();
		// 构造一个job对象
		Job wordCountJob = Job.getInstance(conf);

		// 指定job用到的jar包位置,这里使用当前类
		wordCountJob.setJarByClass(MyWordCountDriver.class);

		// 指定mapper
		wordCountJob.setMapperClass(MyWordCountMapper.class);
		// 指定reducer
		wordCountJob.setReducerClass(MyWordCountReducer.class);

		// 指定mapper输出key/value的类型
		wordCountJob.setMapOutputKeyClass(Text.class);
		wordCountJob.setMapOutputValueClass(LongWritable.class);

		// 指定reducer输出key/value的类型
		wordCountJob.setOutputKeyClass(Text.class);
		wordCountJob.setOutputValueClass(LongWritable.class);

		// 指定输入数据的路径
		FileInputFormat.setInputPaths(wordCountJob, new Path(args[0]));
		// 指定输出结果的路径
		FileOutputFormat.setOutputPath(wordCountJob, new Path(args[1]));
		// 通过yarn客户端进行提交,参数2为是否打印到控制台
		wordCountJob.waitForCompletion(true);
	}
}
```

#### 启动MapReduce

&nbsp;&nbsp;**方式1:** 将程序打成jar包,上传到hadoop中执行。hadoop jar <jar> [mainClass] args…

&nbsp;&nbsp;**方式2:** 将程序打成jar包,在本地IDE上直接运行(需要代码指定jar)。

### 自定义Sort

#### bean

```java
public class FlowBean implements WritableComparable<FlowBean> {

	private Long upFlow;
	private Long downFlow;
	private Long sumFlow;

	public void setAll(Long upFlow, Long downFlow) {
		this.upFlow = upFlow;
		this.downFlow = downFlow;
		this.sumFlow = upFlow + downFlow;
	}

	public Long getUpFlow() {
		return upFlow;
	}

	public void setUpFlow(Long upFlow) {
		this.upFlow = upFlow;
	}

	public Long getDownFlow() {
		return downFlow;
	}

	public void setDownFlow(Long downFlow) {
		this.downFlow = downFlow;
	}

	public Long getSumFlow() {
		return sumFlow;
	}

	public void setSumFlow(Long sumFlow) {
		this.sumFlow = sumFlow;
	}

	@Override
	public String toString() {
		return "FlowBean [upFlow=" + upFlow + ", downFlow=" + downFlow
				+ ", sumFlow=" + sumFlow + "]";
	}

	/**
	 * 序列化
	 */
	@Override
	public void write(DataOutput out) throws IOException {
		out.writeLong(upFlow);
		out.writeLong(downFlow);
		out.writeLong(sumFlow);
	}

	/**
	 * 反序列化
	 */
	@Override
	public void readFields(DataInput in) throws IOException {
		upFlow = in.readLong();
		downFlow = in.readLong();
		sumFlow = in.readLong();
	}

	/**
	 * 降序排序 -1 大于 0 等于 1小于
	 */
	@Override
	public int compareTo(FlowBean o) {
		// 如果当前类的总和大于其他类的总和 则返回-1(大于) false 1(小于)
		return this.sumFlow > o.getSumFlow() ? -1 : 1;
	}

}
```

#### main

```java
public class FlowSummarySort {

	/**
	 * 因为只有key才能进行排序,所以输出key为FlowBean
	 * 
	 * @author sylvanasp
	 * @version 1.0
	 */
	public static class FlowSummarySortMapper
			extends Mapper<LongWritable, Text, FlowBean, Text> {

		@Override
		protected void map(LongWritable key, Text value, Context context)
				throws IOException, InterruptedException {
			String line = value.toString();
			String[] fields = StringUtils.split(line, "\t");
			String phoneNum = fields[0];
			Long upFlow = Long.parseLong(fields[1]);
			Long downFlow = Long.parseLong(fields[2]);

			FlowBean flowBean = new FlowBean();
			flowBean.setAll(upFlow, downFlow);
			context.write(flowBean, new Text(phoneNum));
		}

	}

	/**
	 * 因为在mapper中已经完成了排序,所以reducer中需要将phoneNum重新设置为key
	 * 
	 * @author sylvanasp
	 * @version 1.0
	 */
	public static class FlowSummarySortReducer
			extends Reducer<FlowBean, Text, Text, FlowBean> {

		@Override
		protected void reduce(FlowBean bean, Iterable<Text> phoneNum,
				Context context) throws IOException, InterruptedException {
			// 因为每个bean都是完全独立的,所以Iterable中只有一个数据
			for (Text phoneNumKey : phoneNum) {
				context.write(phoneNumKey, bean);
			}
		}

	}

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(FlowSummarySort.class);
		job.setMapperClass(FlowSummarySortMapper.class);
		job.setReducerClass(FlowSummarySortReducer.class);

		job.setMapOutputKeyClass(FlowBean.class);
		job.setMapOutputValueClass(Text.class);

		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(FlowBean.class);

		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		int result = job.waitForCompletion(true) ? 0 : 1;
		System.exit(result);
	}

}
```

### 自定义Partition

#### partitioner

```java
public class MyPartitioner extends Partitioner<Text, FlowBean> {

	// 使用map模拟数据库
	private static HashMap<String, Integer> map = new HashMap<String, Integer>();

	// 初始化分区规则
	static {
		map.put("136", 0);
		map.put("137", 1);
		map.put("138", 2);
		map.put("139", 3);
	}

	@Override
	public int getPartition(Text key, FlowBean value, int numPartitions) {
		// 获取手机号前3位
		String phonePrefix = key.toString().substring(0, 3);
		// 根据手机号前缀获得对应的分区编号
		Integer partitionId = map.get(phonePrefix);
		// 如果手机号不在分区规则内,则分配到分区4。
		return partitionId == null ? 4 : partitionId;
	}

}
```

#### main

```java
public class FlowSummaryPartition {

	public static class FlowSummaryPartitionMapper
			extends Mapper<LongWritable, Text, Text, FlowBean> {

		@Override
		protected void map(LongWritable key, Text value, Context context)
				throws IOException, InterruptedException {
			String line = value.toString();
			String[] fields = StringUtils.split(line, "\t");

			String phoneNum = fields[1];
			Long upFlow = Long.parseLong(fields[fields.length - 3]);
			Long downFlow = Long.parseLong(fields[fields.length - 2]);

			FlowBean flowBean = new FlowBean();
			flowBean.setAll(upFlow, downFlow);
			context.write(new Text(phoneNum), flowBean);
		}

	}

	public static class FlowSummaryPartitionReducer
			extends Reducer<Text, FlowBean, Text, FlowBean> {

		@Override
		protected void reduce(Text key, Iterable<FlowBean> beans,
				Context context) throws IOException, InterruptedException {
			long upSum = 0;
			long downSum = 0;

			for (FlowBean bean : beans) {
				upSum += bean.getUpFlow();
				downSum += bean.getDownFlow();
			}
			FlowBean flowBean = new FlowBean();
			flowBean.setAll(upSum, downSum);
			context.write(key, flowBean);
		}

	}

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);

		job.setJarByClass(FlowSummaryPartition.class);
		job.setMapperClass(FlowSummaryPartitionMapper.class);
		job.setReducerClass(FlowSummaryPartitionReducer.class);

		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(FlowBean.class);

		// 设置分区器
		job.setPartitionerClass(MyPartitioner.class);
		// 设置Reducer Task 实例数量 (与分区数一致)
		job.setNumReduceTasks(5);

		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		int result = job.waitForCompletion(true) ? 0 : 1;
		System.exit(result);
	}

}
```

### END
> 部分资料来源于http://blog.sina.com.cn/s/blog_829a682d0101lc9d.html&http://weixiaolu.iteye.com/blog/1474172