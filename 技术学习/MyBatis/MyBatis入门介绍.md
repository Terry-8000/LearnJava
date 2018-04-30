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

**代码实例：**

1）建立PO类。

```java
package com.wangjun.mybatis.test.mybatis;

public class User {
private Integer id;
private String name;
private Integer age;
public User(String name, Integer age) {
super();
this.name = name;
this.age = age;
}
//必须要有这个无参构造器，不然根据UserMapper.xml中的配置，在查询数据库的时候，将不能呢过反射构造出User实例
public User() {
}

public Integer getId() {
return id;
}

public void setId(Integer id) {
this.id = id;
}

public String getName() {
return name;
}

public void setName(String name) {
this.name = name;
}

public Integer getAge() {
return age;
}

public void setAge(Integer age) {
this.age = age;
}
}
```

2）建立Mapper。

```java
package com.wangjun.mybatis.test.mybatis;

public interface UserMapper {
	public void insertUser(User user);
	public User getUser(Integer id);
}
```

3\)建立配置文件。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
		<setting name="cacheEnabled" value="false"></setting>
		<setting name="useGeneratedKeys" value="true"></setting>
		<setting name="defaultExecutorType" value="REUSE"></setting>
	</settings>
	<typeAliases>
		<typeAlias alias="User" type="com.wangjun.mybatis.test.mybatis.User"/>
	</typeAliases>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <!-- 新版本的jdbc建议使用com.mysql.cj.jdbc.Driver -->
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <!-- 不加 ?serverTimezone=GMT 可能会有数据库时区和系统时区不一致导致的问题 -->
        <property name="url" value="jdbc:mysql://localhost/test?serverTimezone=GMT"/>
        <property name="username" value="root"/>
        <property name="password" value="password"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="UserMapper.xml"/>
  </mappers>
</configuration>
```

4\)建立映射文件。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.wangjun.mybatis.test.mybatis.UserMapper">
	<insert id="insertUser" parameterType="User">
		insert into user(name,age) values(#{name},#{age})
	</insert>
	<select id="getUser" resultType="User" parameterType="java.lang.Integer">
	  select * from user where id = #{id}
	</select>
</mapper>
```

5\)测试类

```java
package com.wangjun.mybatis.test.mybatis;

import java.io.IOException;
import java.io.Reader;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

public class MyBatisUtil 
{
	private final static SqlSessionFactory sqlSessionFactory;
	static {
		String resource = "configuration.xml";
		Reader reader = null;
		try {
			reader = Resources.getResourceAsReader(resource);
		} catch (IOException e) {
			e.printStackTrace();
		}
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
	}
	public static SqlSessionFactory getSqlSessionFactory() {
		return sqlSessionFactory;
	}
}
```



```java
package com.wangjun.mybatis.test.mybatis;

import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

public class TestMapper { 
	static SqlSessionFactory sqlSessionFactory = null;
	static {
		System.out.println(111);
		sqlSessionFactory = MyBatisUtil.getSqlSessionFactory();
	}
	
	public void testAdd() {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		try {
			UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
			User user = new User("wangjun", new Integer(25));
			userMapper.insertUser(user);
			sqlSession.commit();
		}finally {
			sqlSession.close();
		}
	}
	public void getUser() {
		SqlSession sqlSession = sqlSessionFactory.openSession();
		try {
			UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
			User user = userMapper.getUser(1);
			System.out.println("name:" + user.getName() + "|age:" + user.getAge());
		}finally {
			sqlSession.close();
		}
	}
	
	public static void main(String[] args) {
		TestMapper tm = new TestMapper();
		tm.testAdd();
		tm.getUser();
	}
}
```

运行结果：

```bash
name:wangjun|age:25
```

补充，POM文件配置：需要依赖jdbc和myBaits

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.wangjun.mybatis</groupId>
	<artifactId>test.mybatis</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>test.mybatis</name>
	<url>http://maven.apache.org</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>

		<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>6.0.6</version>
		</dependency>

		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.4.5</version>
		</dependency>
	</dependencies>
</project>

```

### Spring整合MyBatis步骤：

上述步骤的1,2，3步不变。只需要配置Spring文件：

1. 将MyBatis配置文件的environments配置移动到了Spring的配置文件中。针对MyBstis，注册org.mybatis.Spring.SqlsessionFactoryBean类型的bean，以及用于映射接口的org.mybatis.Spring.mapper.MapperFactoryBean。
2. MyBatis的配置文件简化。

**测试**

Spring整合MyBatis很简单，我们可以看到除了MyBaits配置文件的更改并没有太大变化。其实Spring整合MyBatis的优势主要在于使用上，我们来看看Spring中使用MyBatis的用法：

```java
//TODO 补充Java代码
```

我们可以看到，在Spring中使用MyBatis非常方便，用户甚至无法察觉自己正在使用MyBatis，而这一切相对于独立使用MyBatis时必须要做的冗余操作来说无非是打打简化了我们的工作量。

