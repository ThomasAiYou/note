# Spark 宽窄依赖和Shuffle

## 1. 简介

​		在DAG阶段以shuffle为界，划分stage，上游stage做map task，每个map task将计算结果数据分成多份，每一份对应到下游stage的每个partition中，并将其临时写到磁盘，该过程叫做shuffle write；下游stage做reduce task，每个reduce task通过网络拉取上游stage中所有map task的指定分区结果数据，该过程叫做shuffle read，最后完成reduce的业务逻辑。



## 2. 发展过程

### Hash Shuffle V1

​		在spark-1.1版本以前，spark内部实现的是Hash Shuffle。

​		在map阶段(shuffle write)，每个map都会为下游stage的每个partition写一个临时文件，假如下游stage有1000个partition，那么每个map都会生成1000个临时文件，一般来说一个executor上会运行多个map task，这样下来，一个executor上会有非常多的临时文件，假如一个executor上运行M个map task，下游stage有N个partition，那么一个executor上会生成M*N个文件。*

![img](http://sharkdtu.com/images/spark-shuffle-v1.png)



### Hash Shuffle v2

​		在V1中，每个map task都要生成N个partition文件，为了减少文件数，后面引进了，目的是减少单个executor上的文件数。如下图所示，一个executor上所有的map task生成的分区文件只有一份，即将所有的map task相同的分区文件合并，这样每个executor上最多只生成N个分区文件。

![img](http://sharkdtu.com/images/spark-shuffle-v2.png)

### bypass运行机制

​		当shuffle read task的数量小于等于spark.shuffle.sort.bypassMergeThreshold参数的值的时候，就会启用bypass机制。该过程实际跟未经优化的hashshuffle流程相同，都会创建数量惊人的磁盘文件，只是在最后会做磁盘文件的合并，最终只有一个分区文件和一个索引文件。节省了sort shuflle中排序的开销。

### Sort Shuffle

​		在map阶段(shuffle write)，会按照partition id以及key对记录进行排序，将所有partition的数据写在同一个文件中，该文件中的记录首先是按照partition id排序一个一个分区的顺序排列，每个partition内部是按照key进行排序存放，map task运行期间会顺序写每个partition的数据，并通过一个索引文件记录每个partition的大小和偏移量。

![img](http://sharkdtu.com/images/spark-shuffle-v3.png)



### Unsafe Shuffle 

​		它的做法是将数据记录用二进制的方式存储，直接在序列化的二进制数据上sort而不是在java 对象上，这样一方面可以减少memory的使用和GC的开销，另一方面避免shuffle过程中频繁的序列化以及反序列化。但是使用Unsafe Shuffle有几个限制，shuffle阶段不能有aggregate操作。



### Sort Shuffle v2 

​		从spark-1.6.0开始，把Sort Shuffle和Unsafe Shuffle全部统一到Sort Shuffle中，如果检测到满足Unsafe Shuffle条件会自动采用Unsafe Shuffle，否则采用Sort Shuffle。从spark-2.0.0开始，spark把Hash Shuffle移除，可以说目前spark-2.0中只有一种Shuffle，即为带bypass机制的Sort Shuffle。

e[]:http://sharkdtu.com/posts/spark-shuffle.html



## 3. 触发shuffle操作的算子

![img](https://pic4.zhimg.com/80/v2-6c5382709dc907e1c469d73b12bfbde7_1440w.jpg)



## 4. shuffle数据倾斜的问题

### 1. 提升reduce的并行度

​		将reduce task的数量变多，就可以让每个reduce task分配到更少的数据量。这样的话也许就可以缓解甚至是基本解决掉数据倾斜的问题。

### 2. 将reduce join转换为map join

​		如果两个RDD要进行join，其中一个RDD是比较小的，可以将小的RDD广播出去，这样在每个executor值得block manager中都保存一份，这样不会发生shuffle操作，从根本上杜绝了数据倾斜的问题。

### 3. 将数据量大的key进行单独的处理

​		对于极个别的数据倾斜，并且无法在map端进行合并的时候，可以单独进行处理，然后将结果进行合并，减少单个task压力。



## 5. 宽依赖和窄依赖

### 1.概念		

​		宽依赖是指一个父RDD分区对应多个子RDD分区，窄依赖是指一个父RDD分区对应一个子RDD分区。

### 2. 为什么分为宽依赖和窄依赖

​		窄依赖可以支持在同一个集群的Executor上，已pipeline管道的形式顺序执行多条命令，分区内的计算收敛，不需要依赖所有分区的数据，可以并行的在不同节点间进行计算。宽依赖需要所有父分区都是可用的，需要shuffle进行节点之间的数据传输，因此宽窄依赖是spark job划分为stage的关键。

