---
layout:     post
title:      "Hadoop学习笔记(1)-HDFS"
subtitle:   "HDFS QuickStart"
date:       2016-07-12 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - 大数据
    - Hadoop
tags:
    - Hadoop
    - 大数据
---



![](http://ww4.sinaimg.cn/mw690/63503acbjw1f5w446w8pcj20bp08rweu.jpg)

### Hadoop概述

Hadoop是一个由Apache基金会所开发的分布式系统基础架构。

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f5w4478njsj20as05wwep.jpg)

&nbsp;&nbsp;Hadoop 由许多元素构成。其最底部是 Hadoop Distributed File System（HDFS），它存储 Hadoop 集群中所有存储节点上的文件。HDFS的上一层是MapReduce 引擎，该引擎由 JobTrackers 和 TaskTrackers 组成。

**HDFS**

&nbsp;&nbsp;对外部客户机而言，HDFS就像一个传统的分级文件系统。可以创建、删除、移动或重命名文件，等等。但是 HDFS 的架构是基于一组特定的节点构建的，这是由它自身的特点决定的。这些节点包括 NameNode（仅一个），它在 HDFS 内部提供元数据服务；DataNode，它为 HDFS 提供存储块。由于仅存在一个 NameNode，因此这是 HDFS 的一个缺点（单点失败）。

&nbsp;&nbsp;存储在 HDFS 中的文件被分成块，然后将这些块复制到多个计算机中（DataNode）。这与传统的 RAID 架构大不相同。块的大小（通常为 64MB）和复制的块数量在创建文件时由客户机决定。NameNode 可以控制所有文件操作。HDFS 内部的所有通信都基于标准的 TCP/IP 协议。

&nbsp;&nbsp;HDFS和现有的分布式文件系统有很多共同点。但同时，它和其他的分布式文件系统的区别也是很明显的。HDFS是一个高度容错性的系统，适合部署在廉价的机器上。HDFS能提供高吞吐量的数据访问，非常适合大规模数据集上的应用。HDFS放宽了一部分POSIX约束，来实现流式读取文件系统数据的目的。HDFS在最开始是作为Apache Nutch搜索引擎项目的基础架构而开发的。HDFS是Apache Hadoop Core项目的一部分。

**NameNode**

&nbsp;&nbsp;NameNode 是一个通常在HDFS实例中的单独机器上运行的软件。它负责管理文件系统名称空间和控制外部客户机的访问。NameNode 决定是否将文件映射到 DataNode 上的复制块上。对于最常见的 3 个复制块，第一个复制块存储在同一机架的不同节点上，最后一个复制块存储在不同机架的某个节点上。

&nbsp;&nbsp;实际的 I/O事务并没有经过 NameNode，只有表示 DataNode 和块的文件映射的元数据经过 NameNode。当外部客户机发送请求要求创建文件时，NameNode 会以块标识和该块的第一个副本的 DataNode IP 地址作为响应。这个 NameNode 还会通知其他将要接收该块的副本的 DataNode。

&nbsp;&nbsp;NameNode 在一个称为FsImage的文件中存储所有关于文件系统名称空间的信息。这个文件和一个包含所有事务的记录文件（这里是 EditLog）将存储在 NameNode 的本地文件系统上。FsImage 和 EditLog 文件也需要复制副本，以防文件损坏或 NameNode 系统丢失。

&nbsp;&nbsp;NameNode本身不可避免地具有SPOF（Single Point Of Failure）单点失效的风险，主备模式并不能解决这个问题，通过Hadoop Non-stop namenode才能实现100% uptime可用时间。

**DataNode**

&nbsp;&nbsp;DataNode 也是一个通常在 HDFS实例中的单独机器上运行的软件。Hadoop 集群包含一个 NameNode 和大量 DataNode。DataNode通常以机架的形式组织，机架通过一个交换机将所有系统连接起来。Hadoop 的一个假设是：机架内部节点之间的传输速度快于机架间节点的传输速度。

&nbsp;&nbsp;DataNode 响应来自 HDFS 客户机的读写请求。它们还响应来自 NameNode 的创建、删除和复制块的命令。NameNode 依赖来自每个 DataNode 的定期心跳（heartbeat）消息。每条消息都包含一个块报告，NameNode 可以根据这个报告验证块映射和其他文件系统元数据。如果 DataNode 不能发送心跳消息，NameNode 将采取修复措施，重新复制在该节点上丢失的块。

### HDFS体系结构

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f5w447vawzj20hq0aptar.jpg)

 - **NameNode**:唯一的master节点,管理HDFS的名称空间和数据块映射信息、配置副本策略和处理客户端请求。
 - **Secondary NameNode**:辅助NameNode，分担NameNode工作，定期合并fsimage和edits并推送给NameNode，紧急情况下可辅助恢复NameNode。
 - **DataNode**:Slave节点，实际存储数据、执行数据块的读写并汇报存储信息给NameNode。
 - **FSImage**:元数据镜像文件。
 - **Edits**:元数据的操作日志。

### HDFSWriteOperation

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f5w48p90chj20k20bzdgr.jpg)

&nbsp;&nbsp;在分布式文件系统中，需要确保数据的一致性。对于HDFS来说，直到所有要保存数据的DataNodes确认它们都有文件的副本时，数据才被认为写入完成。因此，数据一致性是在写的阶段完成的。一个客户端无论选择从哪个DataNode读取，都将得到相同的数据。

 1. 客户端请求NameNode,表示写入文件。
 2. NameNode响应客户端,并告诉客户端将文件保存到DataNodeA、B、D。
 3. 客户端连接DataNodeA写入文件,DataNode集群内完成复制。
 4. DataNodeA将文件副本发送给DataNodeB。
 5. DataNodeB将文件副本发送给DataNodeD。
 6. DataNodeD返回确认消息给DataNodeB。
 7. DataNodeB返回确认消息给DataNodeA。
 8. DataNodeA返回确认消息给客户端,写入完成。

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f5w448fsbnj20gn0a6acc.jpg)

 1. Client调用DistributedFileSystem的create()函数创建新文件。
 2. DistributedFileSystem使用RPC调用NameNode创建一个没有block关联的新文件,NameNode在创建之前将进行校验,如果校验通过,NameNode则创建一个新文件并记录一条记录,否则抛出IO异常。
 3. 前两步成功后,将会返回一个DFSOutputStream对象,DFSOutputStream可以协调NameNode与DataNode,当客户端写入数据到DFSOutputStream,DFSOutputStream会将数据分割为一个一个Packet(数据包),并写入数据队列。
 4. DataStreamer处理数据队列,它会先去询问NameNode存储到哪几个DataNode,例如Replication为3,则会去找到3个最适合的DataNode。DataStreamer会将DataNode排成一个Pipeline,它会将Packet按队列输出到管道中的第一个DataNode,第一个DataNode又会把Packet输出到第二个DataNode,直到最后一个DataNode。
 5. DataStreamer中还有一个Ack Queue,Ack Queue之中也含有Packet。Ack Queue负责接收DataNode的确认响应,当Pipeline中的所有DataNode都确认完毕后,Ack Queue将移除对应的Packet。
 6. Client完成数据写入,关闭流。
 7. DataStreamer等待Ack Queue信息,当收到最后一个信息时,通知NameNode把文件标记为完成。

### HDFSReadOperation

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f5w48ovukqj20jm0ermy3.jpg)

 1. 客户端请求NameNode,表示读取文件。
 2. NameNode响应客户端,将block(数据块)的信息发送给客户端。
 3. 客户端检查数据块信息,连接相关的DataNode。
 4. DataNodeA将block1发送给客户端。
 5. DataNodeB将block2发送给客户端。
 6. 拼接数据,读取完成。

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f5w4486azhj20g209adi1.jpg)

 1. Client调用FileSystem的open()函数打开希望读取的文件。
 2.  DistributedFileSystem使用RPC调用NameNode确定文件起始块的位置，同一Block按照重复数会返回多个位置，这些位置按照Hadoop集群拓扑结构排序，距离客户端近的排在前面。
 3. 前两步成功后,将会返回一个DFSInputStream对象,DFSInputStream可以协调NameNode与DataNode。客户端对DFSInputStream输入流调用read()函数。
 4. DFSInputStream连接距离最近的DataNode，通过对数据流反复调用read()函数，可以将数据从DataNode传输到客户端。
 5. 当到达Block的末端时，DFSInputStream会关闭与该DataNode的连接，然后寻找下一个Block的最佳DataNode，这些操作对客户端来说是透明的。
 6. 客户端完成读取，对FSDataInputStream调用close()关闭文件读取。

### HDFSShell命令

&nbsp;&nbsp;既然 HDFS 是存取数据的分布式文件系统，那么对 HDFS 的操作，就是文件系统的基本 操作，比如文件的创建、修改、删除、修改权限等，文件夹的创建、删除、重命名等。对 HDFS 的操作命令类似于 Linux 的 shell 对文件的操作，如 ls、mkdir、rm 等。 我们执行以下操作的时候，一定要确定 hadoop 是正常运行的，使用 jps 命令确保看到 各个 hadoop 进程。 

| 命令名         | 格式                                                         | 含义                       |
| -------------- | ------------------------------------------------------------ | -------------------------- |
| -ls            | -ls<路径>                                                    | 查看指定路径的当前目录结构 |
| -lsr           | -lsr<路径>                                                   | 递归查看指定路径的目录结构 |
| -du            | -du<路径>                                                    | 统计目录下个文件大小       |
| -dus           | -dus<路径>                                                   | 汇总统计目录下文件(夹)大小 |
| -count         | -count[-q]<路径>                                             | 统计文件(夹)数量           |
| -mv            | -mv<源路径><目的路径>                                        | 移动                       |
| -cp            | -cp<源路径><目的路径>                                        | 复制                       |
| -rm            | -rm[-skipTrash]<路径>                                        | 删除文件/空白文件夹        |
| -rmr           | -rmr[-skipTrash]<路径>                                       | 递归删除                   |
| -put           | -put<多个 linux 上的文件><hdfs 路径>                         | 上传文件                   |
| -copyFromLocal | -copyFromLocal<多个 linux 上的文件> <hdfs 路径>              | 从本地复制                 |
| -moveFromLocal | -moveFromLocal<多个 linux 上的文件> <hdfs 路径>              | 从本地移动                 |
| -getmerge      | -getmerge<源路径><linux 路径>                                | 合并到本地                 |
| -cat           | -cat<hdfs 路径>                                              | 查看文件内容               |
| -text          | -text<hdfs 路径>                                             | 查看文件内容               |
| -copyToLocal   | -copyToLocal[-ignoreCrc][-crc][hdfs 源路 径][linux 目的路径] | 复制到本地                 |
| -moveToLocal   | -moveToLocal[-crc]<hdfs 源路径><linux 目的路径>              | 移动到本地                 |
| -mkdir         | -mkdir<hdfs 路径>                                            | 创建空白文件夹             |
| -setrep        | -setrep[-R][-w]<副本数><路径>                                | 修改副本数量               |
| -touchz        | -touchz<文件路径>                                            | 创建空白文件               |
| -stat          | -stat[format]<路径>                                          | 显示文件统计信息           |
| -tail          | -tail[-f]<文件>                                              | 查看文件尾部信息           |
| -chmod         | -chmod[-R]<权限模式>[路径]                                   | 修改权限                   |
| -chown         | -chown[-R][属主][:[属组]] 路径                               | 修改属主                   |
| -chgrp         | -chgrp[-R] 属组名称 路径                                     | 修改属组                   |
| -help          | -help[命令选项]                                              | 帮助                       |

### 使用JAVA操作HDFS

```java
public class HdfsTest {

	private Configuration conf = null;
	private FileSystem fs = null;
	private FSDataInputStream DFSInputStream = null;

	/**
	 * 初始化FlieSystem
	 * 
	 * @throws IOException
	 * @throws InterruptedException
	 */
	@Before
	public void init() throws IOException, InterruptedException {
		conf = new Configuration();
		conf.set("fs.defaultFS", "hdfs://192.168.145.145:9000");
		fs = fs.get(URI.create("hdfs://192.168.145.145:9000"), conf, "root");
	}

	/**
	 * 读取文件
	 * 
	 * @throws IOException
	 * @throws IllegalArgumentException
	 */
	@Test
	public void testReadAsOpen() throws IllegalArgumentException, IOException {
		Path path = null;
		try {
			path = new Path("/test");
			if (fs.exists(path)) {
				DFSInputStream = fs.open(path);
				IOUtils.copyBytes(DFSInputStream, System.out, conf);
			}
		} finally {
			IOUtils.closeStream(DFSInputStream);
			fs.close();
		}
	}

	/**
	 * 上传本地文件
	 * 
	 * @throws IOException
	 */
	@Test
	public void testUpload() throws IOException {
		Path src = null;
		Path dst = null;
		try {
			src = new Path("f:/saber_by_wlop-d8tjwa5.jpg");// 原路径
			dst = new Path("/saber.jpg");// 目标路径
			// 参数1为是否删除原文件,true为删除,默认为false
			fs.copyFromLocalFile(false, src, dst);
			// 打印文件路径
			System.out.println("Upload to " + conf.get("fs.default.name"));
			System.out.println("----------------------------------------");
			FileStatus[] fileStatus = fs.listStatus(dst);
			for (FileStatus file : fileStatus) {
				System.out.println(file.getPath());
			}
		} finally {
			fs.close();
		}
	}

	/**
	 * 下载文件
	 * 
	 * @throws IOException
	 */
	@Test
	public void testDownload() throws IOException {
		Path src = null;
		Path dst = null;
		try {
			if (fs.exists(src)) {
				src = new Path("/saber.jpg");
				dst = new Path("D:/temp/");
				fs.copyToLocalFile(false, src, dst);
				// 打印文件路径
				System.out.println("Download from " + conf.get("fs.default.name"));
				System.out.println("--------------------------------------------");
				// 迭代路径,参数2为是否递归迭代
				RemoteIterator<LocatedFileStatus> iterator = fs.listFiles(src, true);
				while (iterator.hasNext()) {
					LocatedFileStatus fileStatus = iterator.next();
					System.out.println(fileStatus.getPath());
				}
			}
		} finally {
			fs.close();
		}
	}

	/**
	 * 创建目录
	 * 
	 * @throws IOException
	 */
	@Test
	public void testMkdir() throws IOException {
		Path path = null;
		try {
			path = new Path("/create01");
			// 判断目录是否已存在
			boolean exists = fs.exists(path);
			if (!exists) {
				// 创建目录
				boolean mkdirs = fs.mkdirs(path);
				if (mkdirs) {
					System.out.println("create dir success!");
				} else {
					System.out.println("create dir failure!");
				}
			}
		} finally {
			fs.close();
		}
	}

	/**
	 * 重命名文件
	 * 
	 * @throws IOException
	 */
	@Test
	public void testRename() throws IOException {
		Path oldPath = null;
		Path newPath = null;
		try {
			oldPath = new Path("/saber.jpg");
			newPath = new Path("/saber01.jpg");
			if (fs.exists(oldPath)) {
				boolean rename = fs.rename(oldPath, newPath);
				if (rename) {
					System.out.println("rename success!");
				} else {
					System.out.println("rename failure!");
				}
			}
		} finally {
			fs.close();
		}
	}

	/**
	 * 删除文件
	 * 
	 * @throws IOException
	 */
	@Test
	public void testDelete() throws IOException {
		Path path = null;
		try {
			path = new Path("/saber01.jpg");
			if (fs.exists(path)) {
				boolean delete = fs.deleteOnExit(path);
				if (delete) {
					System.out.println("delete success!");
				} else {
					System.out.println("delete failure!");
				}
			}
		} finally {
			fs.close();
		}
	}
}
```
