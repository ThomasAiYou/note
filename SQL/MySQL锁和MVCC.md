# MySQL锁和MVCC

## 1. 锁

### MyISAM锁

> MyISAM有两种锁，分别是共享读锁和独占写锁，都是表锁。

​		MyISAM读操作与写操作之间，写操作与写操作之间是串行的。并且默认情况下，写锁比读锁有着更高的优先级。



### InnoDB锁

> InnoDB的行锁必须是现在InnoDB的索引上，否则还是会使用表锁。



#### 共享锁和独占锁

​		InnoDB实现了行级的共享锁(S Lock)和独占锁(X lock)。



#### 意向锁

​		InnoDB为了让同一张表表锁和行锁共存，防止表锁和行锁冲突情况，对于行锁使用之前需要先添加意向锁。意向锁之间不会产生阻塞，意向锁只与表锁会发生阻塞。

![截屏2020-07-24 上午10.15.19](/Users/denakira/Desktop/myworkspace/note/SQL/picture/截屏2020-07-24 上午10.15.19.png)



#### 记录锁

​		记录锁是锁记录的一行，防止在更新的时候其他事务进行更改。

```sql
select c1 from t where c1 = 10 for update
```



#### 间隙锁	

​		间隙锁添加在范围数据的第一行之前，最后一行之后。例如	

```sql
select c1 from t where c1 between 10 and 20 for update;
```

​		间隙锁会防止其他事务添加c1为15的数据。



#### Next - Key Lock

​		next-key 锁是锁印记录上的记录锁和索引记录前间隙上的间隙锁的组合。行锁实际上是索引记录锁，next-key锁是索引记录锁加上索引记录之前的间隙锁。



## 2. MVCC

> 多版本并发控制(Multiversion Concurrency Control)指定是一种提高并发的技术，引入多版本并发控制之后，能够做到不同事物间只有写操作是阻塞的，并且解决脏读等问题。MVCC实现的依赖是**隐藏字段、Read View和Undo log**。



### 2.1 隐藏字段

​		InnoDB存储引擎在没行后面添加了三个隐藏字段：

1. DB_TRX_ID：最近一次对本记录作修改的事务ID。
2. DB_ROLL_PTR：回滚指针，指向当前记录行的undo log信息。
3. DB_ROLL_ID：随着新行插入而单调递增的ID，当没有主键和唯一非空索引的时候使用这行作为聚簇索引。



### 2.2 Read View

​		Read View在MVCC中主要是用来做可见性判断的。在InnoDB中创建一个新事务，执行第一个select语句的时候，InnoDB会创建一个快照，快照中会根据TRX_ID列表，当事务要读取某个记录行的时候，InnoDB会根据会根据该记录行于快照中的进行比较，判断是否满足可见性条件，以此来解决脏读和不可重复读的问题。



#### 快照读和当前读

* 快照读：普通的select语句，可以读当前事务之前创建的快照。

* 当前读：select..lock in share mode,select ... for update, update, delete等语句是从数据库中读取最新的数据。

​		InnoDB在实现RR隔离级别的时候，使用MVCC+记录锁+间隙锁的方式，禁止在当前事务未提交之前在间隙间插入新行。



#### RR和RC Read View的区别

​		在RR级别下，在事务begin之后，只有第一条select的时候，才会创建快照，其他select不会创建新的快照，直到事务结束。在RC级别下，在事务begin之后，每条select语句快照会被重置，即重新创建新的快照。



### 2.3 undo log

​		undo log中保存的是之前的数据版本，当一个事务要读取记录行时，如果当前记录行的TRX_ID不可见，可以顺着undo log的DB_ROLL_PTR找到满足可见性条件的记录版本。

![截屏2020-09-11 上午11.19.04](/Users/denakira/Desktop/myworkspace/note/SQL/picture/截屏2020-09-11 上午11.19.04.png)



### 2.4 RR模式下发生幻读的例子

![截屏2020-07-24 上午10.36.57](/Users/denakira/Desktop/myworkspace/note/SQL/picture/截屏2020-07-24 上午10.36.57.png)



## 3. 参考资料

[ 参考资料 ]: https://blog.csdn.net/Waves___/article/details/105295060
[参考资料]:https://zhuanlan.zhihu.com/p/29150809

