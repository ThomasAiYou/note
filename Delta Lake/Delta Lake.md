# Delta Lake

## Key Feature

**ACID Transaction** ：数据湖一般有多个写和读的pipeline，delta lake提供了多次写入之间的ACID事务，并且使用的是乐观并发控制(使用的也是快照的方式)。

**Schema Enforcement & Schema Evolution** ：delta lake提供自动验证schema的能力，表中存在但不在 DataFrame 中的列设置为 null。如果 DataFrame 中有 Schema 中没有定义的列，或者datatype不同，则此操作会引发异常。引发一场的操作不会写入任何数据。同时delta lake通过mergeSchema和overwriteSchema提供schema更新的能力。

**Time Travel**：delta lake通过快照的方式允许用户读取表的历史记录，在写入新版本的时候会写入新版本的文件同时保留旧版本。如果不想保留可以使用vacuum来删除指定的旧版本数据。

**Insert & Update**：Delta Lake 以文件级粒度跟踪和修改数据，因此它比读取和覆盖整个分区或表更有效。

[]:https://www.jianshu.com/p/857ad6e2ed69

