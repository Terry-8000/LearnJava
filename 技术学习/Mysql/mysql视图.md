# mysql视图

视图View是一个虚表，它由一个sql查询来定义，可以当做表使用。与持久表不同的是，视图中的数据没有实际的物理存储。

### 视图的作用

- 作为一个抽象装置，程序本身不关心基表结构，只需要按照视图定义来获取数据或者更新数据，因此视图在一定程序上也起到一个安全层的作用；
- 视图能简化用户操作；
-  视图使用户以多种角度看待同一数据；
- 视图对重构数据库提供了一定程度的逻辑独立性；
- 适当的利用视图可以更清晰地表达查询。

### 视图操作实例

```sql
-- 查看基表数据
mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   25 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
|  6 | shell    |   30 |
|  7 | git      |   31 |
|  8 | svn      |   32 |
+----+----------+------+
8 rows in set (0.00 sec)
-- 创建视图
mysql> create view v_user
    -> as
    -> select * from user where age < 30;
Query OK, 0 rows affected (0.03 sec)
-- 查看视图数据
mysql> select * from v_user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   25 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.01 sec)
-- 更新视图数据，也就更新了基表数据
mysql> update v_user set age=26 where name='zhangsan';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
-- 查看更新后的基表数据
mysql> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   26 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
|  6 | shell    |   30 |
|  7 | git      |   31 |
|  8 | svn      |   32 |
+----+----------+------+
8 rows in set (0.00 sec)
-- 查看更新后的视图数据
mysql> select * from v_user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | zhangsan |   26 |
|  2 | lisi     |   26 |
|  3 | wangwu   |   27 |
|  4 | nike     |   28 |
|  5 | lucy     |   29 |
+----+----------+------+
5 rows in set (0.00 sec)
```

> 参考：
>
> https://blog.csdn.net/wangsifu2009/article/details/6719847
>
> 《MySQL技术内幕》

