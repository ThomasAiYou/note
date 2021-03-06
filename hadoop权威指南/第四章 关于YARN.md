# 第四章 关于YARN



![截屏2020-04-01下午12.03.21](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/截屏2020-04-01下午12.03.21.png)

## 4.1 剖析YARN应用运行机制

YARN通过两类长期运行的守护进程提供自己的核心服务：管理集群上资源使用的资源管理器(Resource Manager)，运行在集群中所有节点上且能够启动和监控容器的结点管理器(Node Manager)。

![截屏2020-03-31下午6.03.21](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/截屏2020-03-31下午6.03.21.png)

1.客户端连接资源管理器，运行一个application master

2.资源管理器找到一个能够在容器中启动application master的结点管理器

3.application master一旦运行起来能够做什么依赖于应用本身，可能是简单的运行一个计算，或者向资源管理器请求更多的容器。

### 4.1.1 资源请求

YARN有一个灵活的资源请求模型，可以指定每个容器需要的计算资源数量(内存和CPU),还可以指定容器对本地限制要求。

YARN可以在运行中的任意时刻提出资源申请，可以在最开始提出所有请求，或者为了满足不断变化的应用需要，采取更为动态的方式提出请求

## 4.3 YARN中的调度

YARN中有三种调度器可用：FIFO调度器，容器调度器，公平调度器

**FIFO调度器**

FIFO调度器将应用放置在一个队列中，然后按照提交的顺序运行应用。

FIFO不适合共享集群，大的应用会占用集群中的所有资源，所以每个应用必须等待直到轮到自己运行。

**容量调度器**

容量调度器有两个队列，一个独立的队列专门保证小作业一提交就可以启动，由于队列容量是为那个队列中的作业所保留的，因此这种策略是以整个集群的利用率为代价的。

与FIFO相比大作业执行的时间要长。

**公平调度器**

使用公平调度器时候不需要预留一定量的资源，因为调度器会在所有运行的作业之间动态平衡资源。

第二个作业启动到获得公平资源之间会有时间滞后，因为它必须等待第一个作业使用的容器用完并释放出资源。

### 4.3.2 容量调度器配置

容量调度器允许多个组织共享一个Hadoop集群，每个组织可以分配到全部集群资源的一部分，每个组织被配置一个专门的队列，每个队列配置一定的集群资源，队列可以进一步按层次划分

如果队列中有多个作业，队列资源不够用，如果这时仍有可用的空闲资源，那么容量调度器可能会将空闲的资源分配个队列中的作业，哪怕这会超出队列容量，这被称为“弹性队列“

正常操作时，容量调度器不会通过强行终止来抢占容器，可以通过设置最大容量限制，来保证当前队列不会过多侵占其他队列的容量。但这样做会牺牲队列弹性。

### 4.3.3 公平调度器配置

通过分配文件对公平调度器进行配置，可以为每个队列进行配置，可以对容量调度器支持的层次队列进行配置。

**抢占**

在一个繁忙的集群中，当作业被提交给一个空队列的时候，不会立刻启动，直到集群上已经运行的作业释放了资源。但公平调度器同时也支持抢占。

所谓抢占，就是允许调度器中指那些占用资源超过了其公平共享份额的队列的容器，注意抢占资源会降低整个集群的效率，因为被终止的容器需要重新执行。

**延迟调度**

所有的YARN调度器都试图以本地请求为重，但当一个请求发送到本地节点上时，大概有容器正在运行，因此如果等待一小段时间，能够增加请求在本地节点上运行的机会，从而可以提高集群的效率。容量调度器和公平调度器都支持延迟调度。

YARN中每个节点管理器会周期性地向资源管理器发送心跳请求。心跳中携带了节点管理器中所运行的容器，新容器可用的资源等信息，这样对于一个计划运行一个容器的应用而言，每个心跳就是一个潜在的操作机会。

当使用延迟调度时，他不会简单的使用它收到的第一个调度机会，而是等待最大数目的调度机会发生，然后才放松本地性限制并接受下一个调度机会。

**主导资源公平性**

对于单一类型的资源，公平性的概念很好确定。但多资源类型比如CPU和内存，事情就会变的复杂。

YARN中调度器解决这个问题的思路是，观察每个用户的主导资源，并将其作为对集群资源使用的一个度量。



*参考资料*

[1]: https://blog.csdn.net/weixin_42129080/article/details/80754745

