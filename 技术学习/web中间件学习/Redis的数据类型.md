# Redis的数据类型

Redis支持的数据类型有：

- 字符串String
- 字符串列表list
- 哈希hash
- 字符串集合set
- 有序字符串集合sorted set

比较常用的是字符串和哈希类型。

### 字符串String

特点：

- 二进制保存的，存入和获取的数据相同；
- Value最多可以容纳的数据长度是512M；

### 哈希Hash

特点：

- key，value的map容器；
- 每一个hash可以存储4294967295个键值对

  