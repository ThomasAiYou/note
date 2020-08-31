# Delta Lake

> delta是一个在HDFS存储层之上的中间件。

## 传统数据湖遇到的挑战：

1.数据和写入数据湖是不可靠的：缺乏像传统数据库相似的事务支持，会看到脏数据。

2.数据湖中的数据质量较低：因为传统数据湖中的数据是schema on read的，因此并没有schema验证，数据质量较差。

3.数据湖中的数据更新困难：传统数据湖中的文件是按照分区粒度划分的，因此如果发生数据的更新需要读取整个分区数据，进行清洗转换，最终写回分区中，效率较低。



## Delta Lake特性

**ACID Transaction** ：数据湖一般有多个写和读的pipeline，delta lake提供了多次写入之间的ACID事务，并且使用的是乐观并发控制(使用的也是快照的方式)。

**Schema Enforcement & Schema Evolution** ：delta lake提供自动验证schema的能力，表中存在但不在 DataFrame 中的列设置为 null。如果 DataFrame 中有 Schema 中没有定义的列，或者datatype不同，则此操作会引发异常。引发异常的操作不会写入任何数据。同时delta lake通过mergeSchema和overwriteSchema提供schema更新的能力。

**Time Travel**：delta lake通过快照的方式允许用户读取表的历史记录，在写入新版本的时候会写入新版本的文件同时保留旧版本。如果不想保留可以使用vacuum来删除指定的旧版本数据。

**Insert & Update**：Delta Lake 以文件级粒度跟踪和修改数据，因此它比读取和覆盖整个分区或表更有效。

**数据期望**（即将推出）：Delta Lake 还将支持新的 API 来设置表或目录的数据期望。工程师将能够指定布尔条件并调整严重程度以处理数据期望。当 Apache Spark 作业写入表或目录时，Delta Lake 将自动验证记录，当存在违规时，它将根据提供的严重程度配置对记录进行处理。

[]:https://www.jianshu.com/p/857ad6e2ed69

