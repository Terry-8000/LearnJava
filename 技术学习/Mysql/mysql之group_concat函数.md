# mysql之group_concat函数

在介绍GROUP_CONCAT之前，我们先来看看concat()函数和concat_ws()函数。

先准备一个测试数据库：

```sql
mysql> select * from scores;
+----+----------+-------+
| id | name     | score |
+----+----------+-------+
|  1 | zhangsan | 100   |
|  2 | lisi     | 90    |
|  3 | wangwu   | 99    |
|  4 | zhangsan | 92    |
|  5 | zhangsan | 88    |
|  6 | lisi     | 89    |
+----+----------+-------+
6 rows in set
```

### concat()函数

concat()函数的功能是将多个字符连接成一个字符串。

**语法：** concat(str1, str2,...)

返回结果为连接参数产生的字符串，如果有任何一个参数为null，则返回值为null。

```sql
mysql> select *,concat(name,score) from scores;
+----+----------+-------+--------------------+
| id | name     | score | concat(name,score) |
+----+----------+-------+--------------------+
|  1 | zhangsan | 100   | zhangsan100        |
|  2 | lisi     | 90    | lisi90             |
|  3 | wangwu   | 99    | wangwu99           |
|  4 | zhangsan | 92    | zhangsan92         |
|  5 | zhangsan | 88    | zhangsan88         |
|  6 | lisi     | 89    | lisi89             |
+----+----------+-------+--------------------+
6 rows in set
-- 加分隔符 & 起别名
mysql> select *,concat(name,':',score) as info from scores;
+----+----------+-------+--------------+
| id | name     | score | info         |
+----+----------+-------+--------------+
|  1 | zhangsan | 100   | zhangsan:100 |
|  2 | lisi     | 90    | lisi:90      |
|  3 | wangwu   | 99    | wangwu:99    |
|  4 | zhangsan | 92    | zhangsan:92  |
|  5 | zhangsan | 88    | zhangsan:88  |
|  6 | lisi     | 89    | lisi:89      |
+----+----------+-------+--------------+
6 rows in set
```

### concat_ws()函数

concat()函数加分隔符比较麻烦，如果有10个字段连接起来，就得写9个分隔符，concat_ws()函数就是为了解决这个问题。concat_ws就是concat with separator。

**语法：** concat_ws(separator, str1, str2, ...)

第一个参数指定分隔符。需要注意的是分隔符不能为null，如果为null，则返回结果为null。

```sql
mysql> select *,concat_ws(':',name,score) as info from scores;
+----+----------+-------+--------------+
| id | name     | score | info         |
+----+----------+-------+--------------+
|  1 | zhangsan | 100   | zhangsan:100 |
|  2 | lisi     | 90    | lisi:90      |
|  3 | wangwu   | 99    | wangwu:99    |
|  4 | zhangsan | 92    | zhangsan:92  |
|  5 | zhangsan | 88    | zhangsan:88  |
|  6 | lisi     | 89    | lisi:89      |
+----+----------+-------+--------------+
6 rows in set
```

### group_concat函数

明白了concat()和concat_ws()函数，我们来看一下GROUP_CONCAT()函数。它的功能就是将group by产生的同一个分组中的值连接起来，返回一个字符串结果。如果单独使用，那么就将指定字段所有的值连接起来。

**语法：** 
```
group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc ][separator '分隔符'] )
```

说明：通过使用distinct可以排除重复值；如果希望对结果中的值进行排序，可以使用order by子句；separator是一个字符串值，缺省为一个逗号。

我们知道可以使用group by语句对结果进行分组处理：

```sql
mysql> select * from scores group by name;
+----+----------+-------+
| id | name     | score |
+----+----------+-------+
|  2 | lisi     | 90    |
|  3 | wangwu   | 99    |
|  1 | zhangsan | 100   |
+----+----------+-------+
3 rows in set
```

但是我们只能看到zhangsan的第一个成绩，如果我想看到所有的成绩呢？

```sql
mysql> select *,group_concat(score) from scores group by name;
+----+----------+-------+---------------------+
| id | name     | score | group_concat(score) |
+----+----------+-------+---------------------+
|  2 | lisi     | 90    | 90,89               |
|  3 | wangwu   | 99    | 99                  |
|  1 | zhangsan | 100   | 100,92,88           |
+----+----------+-------+---------------------+
3 rows in set

-- 将分组结果按升序排序，并使用分隔符 ：
mysql> select *,group_concat(score order by score separator ':') from scores group by name;
+----+----------+-------+--------------------------------------------------+
| id | name     | score | group_concat(score order by score separator ':') |
+----+----------+-------+--------------------------------------------------+
|  2 | lisi     |    90 | 89:90                                            |
|  3 | wangwu   |    99 | 99                                               |
|  1 | zhangsan |   100 | 88:92:100                                        |
+----+----------+-------+--------------------------------------------------+
3 rows in set

-- 上面展示了以name分组后所有的score，现在多加展示一个id
mysql> select *,group_concat(concat_ws(':',id,score) order by id) from scores group by name;
+----+----------+-------+---------------------------------------------------+
| id | name     | score | group_concat(concat_ws(':',id,score) order by id) |
+----+----------+-------+---------------------------------------------------+
|  2 | lisi     |    90 | 2:90,6:89                                         |
|  3 | wangwu   |    99 | 3:99                                              |
|  1 | zhangsan |   100 | 1:100,4:92,5:88                                   |
+----+----------+-------+---------------------------------------------------+
3 rows in set

-- 单独使用
mysql> select group_concat(score) from scores;
+---------------------+
| group_concat(score) |
+---------------------+
| 100,90,99,92,88,89  |
+---------------------+
1 row in set
```



> 参考：
>
> https://blog.csdn.net/mary19920410/article/details/76545053