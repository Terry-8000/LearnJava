# mysql事务以及隔离级别

### 1. 简介

MySQL 事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！

- 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
- 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
- 事务用来管理 insert,update,delete 语句

### 2. 事务的基本要素ACID

一般来说，事务是必须满足4个条件（ACID）：：原子性（**A**tomicity，或称不可分割性）、一致性（**C**onsistency）、隔离性（**I**solation，又称独立性）、持久性（**D**urability）。

- **原子性：**一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- **一致性：**在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。比如A向B转账，不可能A扣了钱，B却没收到。
- **隔离性：**数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- **持久性：**事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。

### 3. 事务的并发问题

1. **脏读：**允许读取未提交的脏数据，比如：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据；
2. **不可重复读：**如果你在时间点t1读取了一些记录，在t2时间点也想重新读取一样的数据时，这些记录可能已经被改变，或者消失，比如：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果不一致。
3. **幻读：**系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。

不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表。

### 4. 事务的4种隔离级别

为了解决上面事务的并发问题，sql标准提出了4种隔离级别，下面是每种隔离级别能够解决的问题对应关系：

| 事务隔离级别                   | 脏读   | 不可重复读 | 幻读   |
| ------------------------ | ---- | ----- | ---- |
| read-uncommitted         | N    | N     | N    |
| read-committed           | Y    | N     | N    |
| repeatable-read(default) | Y    | Y     | N    |
| serializable             | Y    | Y     | Y    |

mysql的默认隔离级别是Repeatable。

查看系统级和会话级的隔离级别：

```sql
mysql> select @@global.tx_isolation,@@tx_isolation; 
+-----------------------+-----------------+
| @@global.tx_isolation | @@tx_isolation  |
+-----------------------+-----------------+
| REPEATABLE-READ       | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set, 2 warnings (0.01 sec)
```

下面用例子说明一下这四种隔离级别：

**1. read-uncommitted**

更改隔离级别为read-uncommitted：

```sql
mysql> set session tx_isolation='read-uncommitted';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> select @@tx_isolation;
+------------------+
| @@tx_isolation   |
+------------------+
| READ-UNCOMMITTED |
+------------------+
1 row in set, 1 warning (0.00 sec)
```

首先，准备一些测试数据：

```sql
mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   25 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.00 sec)
```

客户端A：

```sql
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   25 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.00 sec)
```

客户端B：

```sql
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> update user set age=52 where name='zhangsan';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

客户端A：

```sql
mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   52 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.00 sec)
```

可以看到，客户端B的事务还没有提交，在客户端A的事务内就看到了更新的数据。

客户端B：

```sql
mysql> rollback;
Query OK, 0 rows affected (0.02 sec)
```

客户端A：

```sql
mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   25 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.00 sec)
```

由于客户端B的事务回滚，客户端A读取的数据又变成了原始数据，因此上一次客户端A读取的数据变成了脏数据。在并发事务中，这种读取数据的问题就叫做脏读。

**2. read-commited**

要解决上面的问题，可以把数据库的隔离级别改成read-commited。

客户端A：

```sql
mysql> set session tx_isolation='read-committed';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

再按照上述步骤测试一下，发现脏读问题已经解决，在事务B没有commit之前，事务A不会读取到脏数据。

下面演示一下不可重复读的问题。

客户端A：

```sql
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)
mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   25 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.00 sec)
```

客户端B：

```sql
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> update user set age=52 where name='zhangsan';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```

客户端A：

```sql
mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   52 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.00 sec)
mysql> rollback;
Query OK, 0 rows affected (0.00 sec)
```

可以看到在客户端B的事务提交前后，客户端A读取到的数据不一样了。也就是重复读取相同的数据有不同的结果。

个人理解，脏读也属于不可重复读的一个范畴，只是脏读在事务B未提交之前就导致两次读取数据不一样，不可重复读在事务B提交之后导致两次读取结果不一样。还有就是脏读之所以叫脏数据，是因为这条数据没有真正的在数据库中保存过，这是事务的一个中间状态。而不可重复读两次读取不同的数据实际都已经存在于数据库中了。

**3. repeatable-read**

要解决不可重复读的问题，可以将数据库的隔离级别改为repeatable-read。

客户端A：

```sql
mysql> set session tx_isolation='repeatable-read';
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
1 row in set, 1 warning (0.00 sec)
```

再按照上述步骤测试一下，发现不可重复读的问题已经解决，在事务B没有commit之后，事务A读取的数据没有变化，关闭这个事务重新打开一个事务才会读到更新后的数据。

下面演示一下幻读的问题。

客户端A：

```sql
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   25 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.00 sec)
```

客户端B：

```sql
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into user values(6,'shell',30);
Query OK, 1 row affected (0.01 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

客户端A：

```sql
mysql> insert into user values(6,'shell',30);
ERROR 1062 (23000): Duplicate entry '6' for key 'PRIMARY'
mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   25 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.00 sec)
```

可以看到，对于客户端A来说，命名没有id为6的数据，但是还是插入失败，再查询一下还是没有啊，感觉产生了幻觉，这就是幻读问题。幻读和不可重复读的区别在于，不可重复读重点是更新后的读取，幻读重点是插入删除这些操作，解决不可重复读，只需要对对应的数据行加锁就行了。解决幻读则需要对整张表加锁。

如果两个事务B没有提交之前事务A执行插入会如何呢？我们来看一下：

客户端A：

```sql
mysql> insert into user values(6,'shell',30);



```

可以看到如果插入的id和事务B一样，那么事务A的操作会被阻塞，直到事务B提交commit后，才会报错：

客户端A：

```sql
mysql> insert into user values(8,'svn',32);




ERROR 1062 (23000): Duplicate entry '8' for key 'PRIMARY'
```

如果客户端A插入到的数据事务B不冲突，那么会立即返回成功：

客户端A：

```sql
mysql> insert into user values(9,'svn',32);
Query OK, 1 row affected (0.00 sec)
```

**4. serializable**

要解决幻读的问题，可以将数据库的隔离级别改为serializable。

客户端A：

```sql
mysql> set session tx_isolation='serializable';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

再按照上述步骤测试一下，发现幻读的问题已经解决，当事务B尝试insert的事务，被阻塞，也就是事务A将整张表锁住了。直到事务A提交commit以后，事务B的操作才会返回结果。

在这种情况下，只允许一个事务在执行，其它事务必须等待这个事务执行完后才能执行。没有并发，只是单纯的串行。

### 5. 总结

1. mysql中默认事务隔离级别是可重复读时并不会锁住读取到的行;
2. 事务隔离级别为读提交时，写数据只会锁住相应的行;
3. 事务隔离级别为可重复读时，如果有索引（包括主键索引）的时候，以索引列为条件更新数据，会存在间隙锁间隙锁、行锁、下一键锁的问题，从而锁住一些行；如果没有索引，更新数据时会锁住整张表;
4. 事务隔离级别为串行化时，读写数据都会锁住整张表;
5. 隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大，鱼和熊掌不可兼得啊。对于多数应用程序，可以优先考虑把数据库系统的隔离级别设为Read Committed，它能够避免脏读取，而且具有较好的并发性能。尽管它会导致不可重复读、幻读这些并发问题，在可能出现这类问题的个别场合，可以由应用程序采用悲观锁或乐观锁来控制。



> 参考：
>
> http://www.runoob.com/mysql/mysql-transaction.html
>
> https://www.cnblogs.com/huanongying/p/7021555.html
>
> https://blog.csdn.net/taylor_tao/article/details/7063639