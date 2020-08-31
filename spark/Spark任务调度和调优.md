# Spark任务调度和调优

## 1.名词解释

**Master**：Master节点常驻master守护进程，负责管理worker节点，并且从master上提交应用。

**Worker**：Worker节点常驻worker守护进程，与Master节点进行通信，并且管理executor进程。

**Executor进程**：executor进程宿主在worker节点上，一个worker可以有多个executor，每个executor有一个线程池，每个线程可以执行一个task。executor将执行完task后的结果返回至driver。并且RDD是直接缓存在executor的。

**Driver进程**：当我们提交一个spark应用便会启动一个driver进程，构建一个sparkcontext对象，driver会向资源管理器(YARN)申请所需要的资源，也就是executor所需要的资源，然后便会在worker上生成一定数量的executor。每个executor占用一定数量的cpu和memory。当申请完资源后driver就开始调度和执行所写代码。driver将spark代码拆分成多个stage，每个stage执行一部分代码片段，每个stage再拆分成多个task，然后将task分配到executor中执行。

![截屏2020-06-25 下午3.34.15](/Users/denakira/Desktop/myworkspace/note/spark/picture/截屏2020-06-25 下午3.34.15.png)

**Application**:用户提交的应用程序

**Job**:一个action类算子出发的操作

**Stage**:一组任务

**Task**:最小的任务单元。



## 2.任务调度流程

1.启动集群后，Worker会向Master节点报告资源情况，Master掌握集群资源状况。

2.当spark提交一个Application后，spark会在driver端创建两个对象：DAGScheduler和TaskScheduler。根据RDD之间的依赖关系将Application形成一个DAG有向无环图。

3.DAGScheduler是任务调度的高层调度器，它的主要作用是根据RDD宽窄依赖将DAG划分为一个个stage，然后将这些stage以TaskSet的形式提交给TaskScheduler(TaskScheduler是任务调度的底层调度器，TaskSet里面就是一个个stage中并行的task任务)。

4.TaskScheduler会遍历TaskSet集合，拿到每个task后会将task发送到executor中去执行，当task执行失败有TaskScheduler负责重试，将task重新发送给executor，默认3次。如果重试3次依然失败，则整个stage失败。

5.若stage失败，则DAGScheduler负责重试，重新发送TaskSet到TaskScheduler，默认4次。如果4次后依然失败，则整个job失败进而导致整个job失败。



## 3. 调优

spark设置并行task数量一般为 num-executor * executor-cores的2～3倍 比如executor总cpu core数量为300个，那么可以设置900个task数量。