# 第五章 Hadoop的I/O操作

## 5.1 数据完整性

检测数据是否损坏的常见措施是，在数据第一次引入系统时计算校验和(checkSum)并在数据通过一个不可靠的通道进行传输时再次计算校验和，这样就能够发现数据的损坏。

### 5.1.1 HDFS的数据完整性

HDFS会对所有的写入数据计算校验和，它针对每个由dfs.bytes-per-checksum指定字节的数据计算校验和，默认情况下为512字节，由于CRC-32校验和是四个字节，因此存储校验和的额外开销低于1%。

DataNode负责在收到数据后存储该数据及其校验和之前对数据进行验证。正在写数据的客户端将数据及其校验和发送到一系列由datanode组成的管线，管线中最后一个datanode负责验证校验和。

不止是客户端在读取数据块时会验证校验和，每个datanode也会在一个后台线程中运行一个DataBlockScanner，从而定期验证存储在这个datanode上所有数据块。

如果检验错误，则先向namenode报告。这样namenode不会再将请求直接发送到这个节点，之后利用副本复制的方式，删除损坏的数据块。



## 5.2 压缩

## 5.3 序列化

序列化是指将结构化对象转化为字节流以便在网络上传输或写到磁盘进行永久存储的过程。反序列化是指将字节流转回结构化对象的逆过程。

在Hadoop中，系统中多个节点上进程间的通信是通过RPC的方式实现的。RPC协议将消息序列转化成二进制流后发送到远程节点，远程节点接着将二进制流反序列化为原始消息。

Hadoop使用的是自己的序列化格式Writable，Avro(克服了writable)不足。

### 5.3.1 Writable接口

Writable接口定义了两个方法，write方法用于写入，readFields用于读取。

![截屏2020-04-02下午3.46.31](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/截屏2020-04-02下午3.46.31.png)

**WritableComparable接口和comparator**

IntWritable实现了原始的WritableComparable接口，该接口继承了Comparable接口，重写了compareTo方法。

对于MapReduce来说类型比较非常重要，Hadoop提供的一个优化接口是继承自Java Comparator的RawComparator接口。![截屏2020-04-02下午4.09.35](/Users/denakira/Desktop/myworkspace/note/hadoop权威指南/picture/截屏2020-04-02下午4.09.35.png)

该接口允许其实现直接比较数据流中的记录，无需先把数据流反序列化为对象，这样可以避免新建对象的额外开销。

WritableComparator是对继承自Comparator接口的RawComparator接口的一个通用实现，它提供两个主要功能：

1.它提供对原始compare()方法的默认实现，该方法能能够反序列化将在流中进行比较的对象，然后进行比较。

2.它充当RawComparator实例的工厂。



## 5.4 基于文件的数据结构

