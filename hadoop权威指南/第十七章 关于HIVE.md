# 第十七章 关于HIVE

> HIVE是构建在Hadoop上的数据仓库框架，设计的目的是使用SQL来运行MapReduce程序。

## 17.4 Hive与传统数据库相比

### 17.4.1 读时模式(schema on read) vs 写时模式(schema on write)

​		schema on write，写时模型，作用于数据源到数据存储之间，传统数据库使用此方式，数据在入库的时候需要预先设置schema。

​		schema on read, 读时模型，做用户数据存储到数据分析之间，数据先进行存储，在需要分析的时候再为数据设置schema。（ES）

​		schema on write 提升查询性能，schema on read 提升存储性能。



## 17.6 表

### 17.6.1 托管表和外部表

​		在HIVE中创建表，分为托管表和外部表两种情况。默认情况下创建的是托管表。

**托管表**

​		HIVE负责管理数据，这意味着HIVE将把数据移入它的仓库目录。当删除表的时候元数据和数据将会被一起删除。

**外部表**

​		另一种选择是创建外部表，这会让HIVE到仓库以外的位置访问数据。并不会把数据移动到自己的仓库目录，在删除的时候不会删除外部数据，只会删除元数据。

### 17.6.2 分区和桶

​		HIVE把表组织成分区，这是一种根据分区列的值，对表进行粗略划分机制，使用分区可以加快数据分片的查询速度。表或分区可以进一步划分为桶，他会为数据提供额外的结构以获得更高效的查询处理。

#### 17.6.2.1 分区

​		如果根据日期来进行分区，那么同一天的记录就会被存储到同一个分区中。因此对于特定日期的数据的查询处理就会变的非常迅速。一个表可以以多个维度来进行分区。

```sql
create table table_name(ts, line) partitioneed by (dt,string)
```

#### 17.6.2.2 桶

​		把表或者分区组织成桶有两个理由。第一个理由是能获得更高的查询效率。桶为表加上了额外的结构，Hive在处理某些查询的时候能够用上这个额外的数据结构，本质还是避免全表扫描。第二个理由是能使取样或者采样更加高效。

`````sql
create table table_name(col_name col_type) 
clustered by (id) into 4 buckets
`````

​		分桶是将列取hash值然后除以桶的个数取模，来确定最终放在哪个桶之中。桶的个数对应底层Mapreduce中reduceTask的个数。

​		对于高效的数据查询一个例子是Map端join操作，如果两张表都在join的字段上面进行了分桶，则只需要将相同列的桶进行join操作，大大减少了需要扫描的数据量。



## 17.7 查询数据

### 17.7.1 排序和聚集

​		在某些情况下，需要控制特定行该到哪个reducer，这就是Hive的distribute by子句所做的事情。这样所有相同年份的行都将进入同一个reducer分区中。

```sql
select year, temperature from records 2
distribute by year	
```

### 17.7.3 Join

​		Join字句中表的顺序很重要，一般最好将最大的表放在最后。Hive只支持等值连接。

### 17.7.5 视图

​		视图是一种用select语句定义的虚表，Hive中的视图是只读的，无法通过视图为基表加载或插入数据。

```sql
create view view_name as select * from table_name where condition
```

