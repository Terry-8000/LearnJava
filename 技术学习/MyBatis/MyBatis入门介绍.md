# MyBatis入门介绍

### MyBatis简介

MyBatis是支持普通SQL查询、存储过程和高级映射的持久层框架。MyBatis消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索。MyBatis使用简单的XML或注解用于配置和原始映射，将接口和Java的POJOs映射成数据库中的记录。

一般情况下MyBatis是被Spring整合使用的，但是他也可以独立使用。

### 独立使用MyBatis步骤：

1. 建立PO类。用于对数据库中数据的映射，使程序员更关注对Java类的使用而不是数据库的操作。
2. 建立Mapper。数据库操作的映射文件，也就是我们常常说的DAO，用于映射数据库的操作，可以通过配置文件指定方法对应的SQL语句或者直接使用Java提供饿注解方式进行SQL的指定。
3. 建立配置文件。配置文件主要用于配置程序中可变性高的设置，MyhBatis中的配置文件主要封装在configuration中。
4. 建立映射文件。对应于MyBatis全局配置中的mappers的配置属性。主要用于建立对应数据库操作接口的SQL映射。MyBatis会将这里设定的SQL与对应的Java接口相关联，以保证在MyBatis中调用接口的时候会到数据库中执行相应的SQL来简化开发。

**建立测试类。进行测试**

### Spring整合MyBatis步骤：

上述步骤的1,2，3步不变。只需要配置Spring文件：

1. 将MyBatis配置文件的environments配置移动到了Spring的配置文件中。针对MyBstis，注册org.mybatis.Spring.SqlsessionFactoryBean类型的bean，以及用于映射接口的org.mybatis.Spring.mapper.MapperFactoryBean。
2. MyBatis的配置文件简化。

**测试**

Spring整合MyBatis很简单，我们可以看到除了MyBaits配置文件的更改并没有太大变化。其实Spring整合MyBatis的优势主要在于使用上，我们来看看Spring中使用MyBatis的用法：



