### 简介

Redis是一个高性能的并且开源的使用key-value保存数据的NoSql数据库，它的特点包括：

- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。
- 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。
- Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
- 丰富的特性，Redis还支持 publish/subscribe, 通知, key 过期等等特性。

### 应用场景

- 缓存
- 任务队列
- 应用排行榜
- 网站访问统计
- 数据过期处理
- 分布式集群架构中的session分离

### Linux下安装

```shell
#安装gcc环境，因为redis是c++写的
apt-get install gcc
# 下载最新的安装包
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
# 解压
tar -zxvf redis-4.0.9.tar.gz
cd redis-4.0.9/
#编译
make
```

### 启动

```shell
cd src
#前端启动，进去以后不能操作linux了
./redis-server
#后端启动，需要配置${REDIS_HOME}/redis.conf，将 daemonize no 改为 daemonize yes，设置为后台运行
# 然后加上配置项启动
cd ..
./src/redis-server redis.conf 
```

### 停止

```shell
./src/redis-cli shutdown
```

### 进入redis客户端

```shell
./src/redis-cli
#进入客户端了，可以看到端口号6379
127.0.0.1:6379>
```

### 增删查改数据

```shell
127.0.0.1:6379> set name wangjun
OK
127.0.0.1:6379> get name
"wangjun"
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> set name wangjun2
OK
127.0.0.1:6379> get name
"wangjun2"
127.0.0.1:6379> del name
(integer) 1
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> 
```

> 参考：http://www.runoob.com/redis/redis-tutorial.html