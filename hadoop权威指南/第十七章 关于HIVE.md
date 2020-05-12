# 第十七章 关于HIVE

> HIVE是构建在Hadoop上的数据仓库框架，设计的目的是使用SQL来运行MapReduce程序。

## 17.4 Hive与传统数据库相比

### 17.4.1 读时模式(schema on read) vs 写时模式(schema on write)

​		schema on write，写时模型，作用于数据源到数据存储之间，传统数据库使用此方式，数据在入库的时候需要预先设置schema。

​		schema on read, 读时模型，做用户数据存储到数据分析之间，数据先进行存储，在需要分析的时候再为数据设置schema。（ES）

​		schema on write 提升查询性能，schema on read 提升存储性能。