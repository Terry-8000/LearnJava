# SpringBoot入门

### 1. SpringBoot简介

spring boot是一种全新的Java web框架，目的是简化Spring应用的初始搭建和开发过程，让开发者写更少的配置，程序更快的启动和运行，致力于成为快速开发应用领域的领导者。

从它的名字也可以看出，更像是一个引导程序，就跟我们傻瓜式的安装电脑软件一样，next，next...很快我们就可以搭建起一个Spring应用。

### 2. Spring产生背景

在使用Spring和Spring MVC框架的时候，我们需要手动配置很多东西，我们更希望的是“约定大于配置”，就是说系统、类库、框架应该假定合理的默认值，而非要求提供不必要的配置，使用Spring和Spring MVC进行很多配置，不仅增加了工作量，而且在跨平台部署的时候容易出现问题。由于这些问题的存在，Spring Boot应运而生，使用Spring Boot我们可以很快的创建一个基于Spring的项目，让这个Spring项目跑起来只需要很少的配置就可以了。

### 3. Spring Boot的优缺点

Spring Boot可以独立运行Spring项目，它可以以jar包的形式来运行，`java -jar xxx.jar`就可以运行，很方便。并且可以内嵌Tomcat，这样我们无需以war包的形式部署项目。Spring Boot通过starter能够帮助我们简化maven配置。总的来说，Spring Boot大致有以下优缺点。

**优点：**

1. Spring Boot使编码变简单；
2. Spring Boot使配置变简单；
3. Spring Boot使部署变简单；
4. Spring Boot使监控变简单；

**缺点：**

1. 缺少注册、服务发现等外围方案；
2. 缺少外围监控集成方案；
3. 缺少外围安全管理方案；
4. 缺少REST落地的URI规划方案；
5. 比较适合做微服务，不太适合比较大型的项目；
6. 集成度较高，使用过程中不太容易了解底层。

Spring Boot只是微服务框架的起点，配合Spring Cloud可以快速搭建微服务。

###  4. 搭建一个简单的Spring Boot工程

1. 新建一个maven工程；

2. 在pom文件中添加spring boot的依赖：

   ```xml
   <parent>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-parent</artifactId>
     <version>2.0.2.RELEASE</version>
   </parent>
   ...
   <dependencies>
     ...
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
     </dependency>
   </dependencies>
   ```

3. 在App.java中编写服务，这是是程序的入口，通过简单的注释就可以发布一个服务。

   ```java
   package com.wangjun.spring.springboottest;

   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.ResponseBody;

   @Controller
   @EnableAutoConfiguration
   public class App {
   	@RequestMapping("/")
   	@ResponseBody
   	String home() {
   		return "Hello World!";
   	}
   	
   	@RequestMapping("/name")
   	@ResponseBody
   	String getName() {
   		return "spring boot test";
   	}

   	public static void main(String[] args) {
   		SpringApplication.run(App.class, args);
   	}
   }
   ```

4. 然后点击运行，可以在控制台看到输出了Spring Boot字样：

   ```

        .   ____          _            __ _ _
       /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
      ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
       \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
        '  |____| .__|_| |_|_| |_\__, | / / / /
       =========|_|==============|___/=/_/_/_/
       :: Spring Boot ::        (v2.0.2.RELEASE)
   ```
   运行成功后会显示：

   ```
   Tomcat started on port(s): 8080 (http) with context path
   ```

   ​

5. 在浏览器打开localhost:8080，可以看到页面返回Hello World! 打开localhost:8080/name，可以看到页面返回spring boot test。

OK。就是这么简单！spring boot内置了servlet容器，所以不需要想传统方式那样，先部署到容器再启动容器，只需要运行main函数即可。

**配置yml文件：**在Java的路径下新建resources文件夹，里面新建application.yml文件，在eclipse中将recourse文件夹设置为Source Folder。

在yml里面写一点配置：

```
server:
 port: 8030
```

你还需要在pom文件中添加依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
</dependency>
```

因为**spring-boot-starter**会自动加载yml文件（application.yml）

接下来重新启动，访问端口就变成了8030。

### 5. 使用jpa操作数据库

在pom中添加依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
```

在appilication.yml中添加数据库配置：

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?serverTimezone=GMT
    username: root
    password: password
  jpa:
    hibernate:
      ddl-auto: create #create 代表在数据库创建表，update 代表更新
    show-sql: true
```

创建一个实体：

```java
package com.wangjun.spring.springboottest;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;


@Entity
public class User {
	@Id
	@GeneratedValue(strategy= GenerationType.AUTO)  //和表的id生成策略相同
	//id要使用javax.persistence下面的，必然会报错No identifier specified for entity
	private Integer id; 
	private String name;
	private Integer age;
	
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

创建Dao接口， springboot 将接口类会自动注解到spring容器中，不需要我吗做任何配置，只需要继承JpaRepository 即可：

```java
package com.wangjun.spring.springboottest;

import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRep extends JpaRepository<User, Integer>{
	
}
```

创建User的controller类，添加查询和添加的方法：

```java
package com.wangjun.spring.springboottest;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@EnableAutoConfiguration
public class UserController {
	
	@Autowired
	private UserRep userRep;
	
	@RequestMapping("/users")
	@ResponseBody
	public List<User> getUserList(){
		return userRep.findAll();
	}
	
	@RequestMapping("/adduser")
	@ResponseBody
	public User addUser(@RequestParam("name") String name, @RequestParam("age") Integer age) {
		User user = new User();
		user.setName(name);
		user.setAge(age);
		return userRep.save(user);
	}
	
}
```

主方法中改为：

```java
package com.wangjun.spring.springboottest;

import org.springframework.boot.SpringApplication;


public class App {

	public static void main(String[] args) {
		SpringApplication.run(UserController.class, args);
	}
}
```

运行程序，通过get方式访问

```
http://localhost:8030/users
```

```
http://localhost:8030/adduser?name="nike"&age=25
```

就可以查询或者删除数据了。

如过想用post请求，只需要在注解中将method的值改为POST：

```java
@RequestMapping(value = "/adduser", method = RequestMethod.POST)
@ResponseBody
public User addUser(@RequestParam("name") String name, @RequestParam("age") Integer age) {
  User user = new User();
  user.setName(name);
  user.setAge(age);
  return userRep.save(user);
}
```

重新启动服务，然后可以在postman中，重新模拟发送post请求。

### 5. 遇到的问题

**1. spring boot发布的post服务报400错误**

使用postman发送请求时，使用raw，里面直接写json：

```json
{"name":"name", "age":22}
```

报错：400 Bad Request。

使用form-data和x-www-form-urlencoded形式发送数据可以正常返回。

**解决方案：**

```java
addUser(@RequestParam("name") String name, @RequestParam("age") Integer age)
```

这种形式定义的方法只能接受表单形式的数据，如果要想接收json数据，可以将其改为：

```java
@RequestMapping(value = "/adduser", method = RequestMethod.POST)
@ResponseBody
public User addUser(@RequestBody User user) {
  return userRep.save(user);
}
```

**2. 启动时报错：No identifier specified for entity**

错误日志：

```shell
Caused by: org.hibernate.AnnotationException: No identifier specified for entity: com.wangjun.spring.springboottest.User
```

**解决方案：**

在网上搜了一波，原来是在实体类中导入的Id类的包不对，之前导入的是：

```java
import org.springframework.data.annotation.Id;
```

应该是：

```java
import javax.persistence.Id;
```

**3. 每次重启项目，数据库数据会被清空**

**解决方案：**

在application.yml文件中，修改

```yaml
ddl-auto: update #create 代表在数据库创建表，update 代表更新
```



> 参考：
>
> https://blog.csdn.net/fly_zhyu/article/details/76407830
>
> https://www.zhihu.com/question/39483566
>
> https://blog.csdn.net/u012702547/article/details/53740047
>
> https://blog.csdn.net/forezp/article/details/61472783