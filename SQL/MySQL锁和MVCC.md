# MySQL锁和MVCC

## 锁

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

## MVCC

> 多版本并发控制(Multiversion Concurrency Control)指定是一种提高并发的技术，引入多版本并发控制之后，能够做到不同事物间只有写操作是阻塞的，并且解决脏读等问题。

### 快照读和当前读

​		快照读：普通的select语句，可以读当前事物之前创建的快照。

​		当前读：select..lock in share mode,select ... for update, update, delete等语句是从数据库中读取最新的数据。

​		InnoDB在实现RR隔离级别的时候，使用MVCC+记录锁+间隙锁的方式，禁止在当前事务未提交之前在间隙间插入新行。

### RR和RC的read view区别

​		在RR级别下，只有在事务开始时候的第一条select语句才会创建快照，不会重新创建，直到事务结束。在RC级别下，每条select语句都会创建快照，快照会被重置。

### RR模式下发生幻读的例子

![截屏2020-07-24 上午10.36.57](/Users/denakira/Desktop/myworkspace/note/SQL/picture/截屏2020-07-24 上午10.36.57.png)

### 参考资料

[ 参考资料 ]: https://blog.csdn.net/Waves___/article/details/105295060
[参考资料]:https://zhuanlan.zhihu.com/p/29150809

