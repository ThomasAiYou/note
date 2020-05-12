# 第三章 Hadoop分布式文件系统

## 3.6 数据流

### 3.6.1 剖析文件读取

![截屏2020-05-04下午3.25.39](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/截屏2020-05-04下午3.25.39.png)

1.客户端通过调用fileSystem对象的open()方法来打开希望读取的文件，对于HDFS来说，这个对象是DFS的一个实例。

2.DFS通过RPC机制来调用namenode，以确定文件起始块的位置。对于每一个块，namenode返回存有该块副本的namenode地址。

3.DFS返回一个FSDataInputStream对象，这个对象封装了DFSInputStream对象，该对象管理着datanode和namenode的I/O。客户端对这个流调用read()方法DFSInputStream便开始读取文件数据。

4.通过对数据流反复调用read()方法，可以将数据从datanode传输至客户端。

5.到达块末端时候，DFSInputStream关闭与datanode连接，然后寻找下一个块。

6.一旦客户端完成读取，就调用FSDataInputStream的close()方法关闭流。

### 3.6.2 剖析文件写入

![](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/截屏2020-05-04下午3.31.31.png)

1.客户端通过DFS对象调用create()方法来新建文件。

2.DFS使用RPC调用的方式在文件系统命名空间中创建文件。namenode检查创建文件权限等信息，如果通过，在namenode中添加一条记录。DFS返回FSDataOutputStream对象，这个对象封装了DFSOutputStream对象。

3.客户端调用FSDataOutputStream的write方法进行数据写入。

4.DFSOutputStream将数据分为一个个数据包，写入内部数据队列，符合条件的namenode构成管线，datanode串行存储数据包。

5.DFSOutputStream也维护着一个内部队列来等待datanode的确认ACK。

6.写入完成后调用close()方法关闭数据流。

#### 副本存放策略

​		默认策略是在客户端节点上存放第一个副本，在与第一个不同机架的节点上存储第二个副本，在第二个副本的相同机架不同节点上存储第三个副本。