# 第七章 MapReduce的工作机制

## 7.1 剖析MapReduce作业运行机制

整个过程描述如图所示，在最高层有5个独立实体：

1.客户端：提交MapReduce作业。

2.YARN资源管理器(Resource Manager,RM)，负责协调集群上计算资源的分配。

3.YARN节点管理器(Node Manager,NM)，负责启动和监视集群机器上的计算容器。

4.MapReduce的application master，负责协调运行MapReduce作业的任务，它和MapReduce任务在容器中运行，这些容器由资源管理器分配并由节点管理器进行管理。

5.分布式文件系统，用于与其他实体分享作业文件。

![](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/u=2505986884,3554737584&fm=26&gp=0.jpg)

### 7.1.1 作业的提交

> Job的submit()方法创建一个内部的JobSummiter实例，并且调用其submitJobInteral()方法，提交作业后，waitForCompletion()每秒轮询作业的进度。

**JobSummiter作业提交过程**

1.submitter检查输出目录是否有异常，接着得到一个存储jar包的路径JobStagingArea，再向资源管理器请求得到一个JobID。

2.submitter提交器将jar提交到由jobStagingArea和JobID拼接而成的地址，默认写入10份(mapreduce.client.submit.file.replication)通过copyAndConfigureFiles提交到hdfs

3.通过资源管理器的submitApplication方法提交作业。

### 7.1.2 作业的初始化

1.资源管理器收到submitApplication消息后，传递个YARN调度器。调度器分配一个容器，然后资源管理器在节点管理器的管理下在容器中启动application master的进程。

2.application master是一个Java应用程序，主类为MRAppMaster。接下来它接受来自HDFS的，在客户端计算的输入分片，然后对每一个分片创建一个map任务以及由mapreduce.job.reduces属性确定的多个reduce任务，分配任务ID。

3.如果application master判断新分配容器将会浪费开销，就会在当前JVM上运行任务，这种作业称为uberized或者称为uber。默认情况少于10个mapper且只有1个reducer且输入大小小于一个HDFS块的作业为Uber任务。

### 7.1.3 任务的分配

如果任务不适合作为uber任务来运行，那么application master就会该改作业中的所有map任务和reduce任务向资源管理器请求容器。首先为Map任务发出所有请求，该请求优先级高于reduce任务的请求。这是因为所有的map任务必须在reduce的排序阶段能够启动前完成。直到有5%的map任务已经完成时，为reduce任务的请求才会发出。

### 7.1.4 任务的执行

一旦资源管理器为一任务分配了一个特定节点上的容器，application master就通过与节点管理器通信来启动容器。在它运行任务之前，首先将任务需要的资源本地化，包括作业的配置，JAR文件和所有来自分布式缓存的文件。

### 7.1.5 进度和状态的更新

当map任务或者reduce任务运行时，自己成和自己的父application master通过umbilical接口通信。每隔3秒钟任务通过umbilical接口向自己的application master报告进度和状态。application会形成一个自己的作业汇聚视图。

### 7.1.6 作业的完成

最后作业完成时，application master的任务和任务容器清理其工作状态，作业信息由作业历史服务器存档。



## 7.2 失败

### 7.2.1 任务运行失败

任务JVM会在退出之前向其父application master发送错误报告。错误报告被计入用户日志。application master将此次任务尝试标记为failed，并释放容器资源。

若application master发现已经有一段时间没有收到任务进度更新，将会标记任务失败，并且JVM进程将会被杀死(mapreduce.task.timeout)指定。

application master会自动尝试失败的任务，并且会尽量避免在以前失败过的结点管理器上重新调度该任务，尝试次数由mapreduce.map.maxattempts和mapreduce.reduce.maxattempts属性控制。

对于一些应用不希望少数几个任务失败终止所有作业，可设置mapreduce.map.failures.maxpercent来设定接受允许最大失败百分比。      

### 7.2.2 application master运行失败

Application master通过设置mapreduce.am.max-attempts属性设置最大尝试次数。

application master向资源管理器周期性发送心跳，当application master失败时，资源管理器将检测到该失败并在一个新的容器开始新的master实例。对于mapreeduce application master他将使用作业历史来恢复失败的应用程序所运行任务的状态，使其不必重新运行。

作业初始化期间，作业端向资源管理器请求并缓存application master的地址，使其每次需要想application master查询时不必重载资源管理器。但是如果application master运行失败，就会重新请求地址。

### 7.2.3 节点管理器运行失败

如果节点管理器由于崩溃或者运行非常缓慢而失败，就会停止向资源管理器发送心跳信息。如果资源管理器没有收到心跳信息，就会通知停止发送心跳信息的节点管理器，并且将其从自己的节点池中移除。



## 7.3 shuffle和排序

> MapReduce确保每个reducer的输入都是按键排序的。系统执行排序，将map输出作为输入传递给reducer的过程称为shuffle。

![](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/截屏2020-04-17下午4.40.30.png)

### 7.3.1 map端

​		 map函数开始产生输出时，并不是简单的将它写到磁盘，他利用缓冲的方式写到内存并出于效率的考虑进行预排序。

![截屏2020-04-16上午10.46.37](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/截屏2020-04-16上午10.46.37.png)

​	每个map任务都有一个环形内存缓冲区用于存储任务输出。默认情况下，缓冲区大小为100MB(mapreduce.task.io.sort.mb)，一旦缓冲区内容达到阈值(mapreduce.map.sort.spill.percent)默认0.8一个后台线程便开始把内容溢出到磁盘。在溢出写到磁盘过程中，map输出继续写到缓冲区，但如果此期间缓冲区被填满，map会被阻塞，直到写磁盘过程完成。溢出写过程按轮询的方式将缓冲区中的内容写到mapreduce.cluster.local.dir属性在作业特定子目录下指定的目录中。

在写磁盘之前，线程首先根据数据最终要传的reducer把数据划分成相应的partition，在每个分区中后台线程根据键在内存中进行排序。

### 7.3.2 reduce端

![](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/20190716185316790.png)

每个map任务的完成时间可能不同，因此在每个任务完成时，reduce任务就开始复制其输出，因此能够并行取得map输出(mapreduce.reduce.shuffle.parallelcopies)设置，默认5。

复制完所有map输出后，reduce人物进入排序阶段，这个阶段将合并map输出，维持其顺序排序，然后将数据输入至reduce函数。

### 7.3.3 配置调优

> 总原则是给shuffle过程尽量多提供内存空间。



## 7.4 appendix

|                     Name                      | Default  |           Description            |             Error             |
| :-------------------------------------------: | :------: | :------------------------------: | :---------------------------: |
|           mapreduce.task.io.sort.mb           |   100    |      map输出排序缓冲区大小       |                               |
|            mapred.child.java.opts             | -Xmx200m | 运行map任务和reduce任务的jvm大小 |        Java heap space        |
| mapreduce.input.fileinputformat.split.minsize |    0     |       map切分split的最小值       |                               |
|            mapreduce.task.timeout             |  600000  |           task超时时间           |                               |
|            mapreduce.map.memory.mb            |   1024   |        每个map任务的内存         | beyond physical memory limits |
|          mapreduce.reduce.memory.mb           |   1024   |       每个reduce任务的内存       | beyond physical memory limits |
|      yarn.nodemanager.resource.memory-mb      |   8192   |  NodeManager contianer内存大小   |                               |
|     yarn.scheduler.minimum-allocation-mb      |   1024   |  每个container请求的最低jvm配置  |                               |
|     yarn.scheduler.maximum-allocation-mb      |   8192   |  每个container请求的最高jvm配置  |                               |
|         mapreduce.task.io.sort.factor         |    10    |  排序文件时，一次最多合并的流数  |                               |
|          yarn.nodemanager.local-dirs          |          |                                  |                               |

![](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/截屏2020-04-17下午2.59.14-7106782.png)

### 7.4.1 map task数量

```java
//blockSize:取出block大小
//minSize:mapreduce.input.fileinputformat.split.minsize
//goalSize:totalSize/numSplit
//The last split of a file can overflow by 10%. This is called as SPLIT_SLOP and it is set at 1.1
long splitSize = this.computeSplitSize(goalSize, minSize, blockSize);

protected long computeSplitSize(long goalSize, long minSize, long blockSize) {
        return Math.max(minSize, Math.min(goalSize, blockSize));
}
```

### 7.4.2 reduce task数量

reduce task的launch数量通过mapreduce.job.reduces设置，默认值是1。但若设置的值为a，partitioner返回值个数为b，若a>b则有a-b个reduce浪费。

### 7.4.3 参考资料

[参考资料]：(https://blog.csdn.net/aijiudu/article/details/72353510)

[参考资料] : (https://zhuanlan.zhihu.com/p/134124471)

