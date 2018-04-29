# Spring入门AOP和IOC学习笔记

### 概述

Spring框架的核心有两个：

- Spring容器作为超级大工厂，负责管理、创建所有的Java对象，这些Java对象被称为Bean。

- Spring容器管理容器中Bean之间的依赖关系，使用一种叫做“依赖注入”的方式来管理bean之间的依赖关系。

Spring有两个核心接口：`BeanFactory`和`ApplicationContext`,`ApplicationContext`是`BeanFactory`的子接口、它们都可以代表Spring容器，Spring容器是生成Bean实例的工厂，并管理容器中的Bean。建议优先使用ApplicationContext。除非对内存非常关键的应用再考虑使用BeanFactory。当系统创建ApplicationContext的时候，默认会预初始化所有Singleton Bean，这就意味着前期创建ApplicationContext时将有较大的性能开销，但一旦ApplicationContext初始化完成，程序后面获取singleton Bean实例时，就拥有较好的性能。为`<bean/>`元素指定`lazy-init="true"`那么就不会预初始化Singleton bean了。

### IOC

Inversion Of Control，控制反转，也可以叫依赖注入。A对象需要调用B对象的方法的情景，这种情形称为依赖，即A对象依赖B对象。使用依赖注入不仅可以为Bean对象注入普通的属性值，还可以注入其他Bean引用。通过这种依赖注入，Java EE应用中的各种组件不需要以硬编码方式耦合在一起，甚至无需使用工厂模式。当某个Java实例需要其他Java实例时，系统自动提供所需要的实例，无需程序显式获取。

**好处**

先来说说传统使用java实例的不足，一般有两种方式：

- 通过new关键字实例化一个对象；
- 通过工厂模式生产一个实例对象；

第一种方式必然导致调用者和被依赖对象存在硬编码耦合，非常不利于项目升级的维护；第二种比第一种好很多，但是调用组件需要主动通过工厂去获取被依赖的对象，这就会带来调用组件与被依赖工厂的耦合。

那么IOC有什么好处呢?

调用者无需主动获取被依赖的对象，只要被动接受Spring容器为调用者的成员变量即可。总体来说就是主动变为被动，所以被称为控制反转。

**场景**

依赖注入一般有以下两种：

- 设值注入：IoC容器使用成员变量的setter方法来注入被依赖对象；
- 构造注入：IoC容器通过构造器来注入被依赖对象；

建议采用设值注入为主，构造注入为辅的注入策略。对于依赖关系无需变化的注入，尽量采用构造注入；而其他依赖关系的注入，则考虑采用设值注入。

使用IoC容器的三个基本要点：

- 应用程序的各组件面向接口编程，这样就可以将组件之间的耦合关系提升到接口层次，从而有有利于项目后期的发展；
- 应用程序的各组件不再由程序主动创建，而是由Spring容器来负责产生并初始化；
- Spring采用配置文件或注解来管理Bean的实现类、依赖关系，Spring容器则根据配置文件或注解，利用反射来创建实例，并为之注入依赖关系。



### AOP

Aspect Oriented Programming，面向切面编程，用于在模块化方面的横切关注点。AOP和OOP（Object Oriented Programming）互为补充，可以这么理解：面向对象编程是从静态角度考虑程序结构，面向切面编程则是从动态角度考虑运行过程。

简单的说，它是一个拦截器可以拦截一些过程，当一个方法执行，Spring AOP可以拦截一个方法的执行，在这个方法执行的前后添加一些功能。

**AOP中的一些术语**

- 切面（Aspect）：切面组织多个Advice，Advice放在切面中定义。
- 连接点（JoinPoint）：程序执行过程中明确的点，如方法的调用，或者异常的抛出，在Spring AOP中，连接点总是方法的调用。
- 增强处理（Advice）：AOP框架在特定的切入点执行的增强处理，处理有“before”、“after”、“after-returning”、“around”、“after-throwing”等；
- 切入点（Pointcut）：可以插增强处理的连接点。当某个连接点满足指定要求时，该连接点将被添加增强处理（Advice），该连接点也就变成了切入点。

以一个bean配置为例：

```xml
<aop:config>
		<!-- order:指定切面bean的优先级，值越小，优先级越高 -->
		<aop:aspect id="fourAdviceAspect" ref="fourAdviceBean" order="2">
			<aop:around method="processTask" pointcut="execution(* com.wangjun.aop.xml.*.*(..))"/>
			<aop:before method="authority" pointcut="execution(* com.wangjun.aop.xml.*.*(..))"/>
			<aop:after-returning method="log" returning="rvt" pointcut="execution(* com.wangjun.aop.xml.*.*(..))"/>
		</aop:aspect>
	</aop:config>
```

其中`<aop:aspect/>`标签就是切面，此标签下面的`<aop:around/>`、`<aop:before/>`这些就是增强处理，那么在哪里进行增强处理呢？`pointcut`属性就定义了切入点，也就是在哪里进行增强处理。这里的表达式比如`execution(* com.wangjun.aop.xml.*.*(..))`含义如下：

1. 指定在com.wangjun.aop.xml包中任意类方法；
2. 第一个\*表示返回值不限，第二个\*表示类名不限；
3. 第三个\*表示方法名不限，圆括号中的(..)表示任意个数、类型不限的形参。

还有一些术语：

- 引入：将方法或字段添加到被处理的类中。Spring允许将新的接口引入到任何被处理的对象中，例如，你可以使用一个引入，使任何对象实现isModify接口，以此来简化缓存。
- 目标对象：被AOP框架增强处理的对象。如果AOP采用的是动态AOP实现，那么该对象就是一个被代理的对象；
- AOP代理：AOP框架创建的对象，也可以是cglib代理，代理就是堆目标对象的加强。
- 织入（Weaving）：将增强处理添加到目标对象中，并创建一个被增强的对象的过程就是织入。Spring和其他纯javaAOP框架一样，在运行时完成织入。

**使用场景**

日志记录、审计、声明式事务、安全性和缓存等。

**AspectJ和Spring AOP的区别**

两种实现AOP的方式。

AspectJ是静态实现AOP的，即在编译阶段对程序进行修改，需要特殊的编译器，具有较好的性能；

Spring AOP是动态实现AOP的，即在运行阶段动态生成AOP代理，纯java实现，因此无需特殊的编译器，但是通常性能较差。

目前Spring已经对AspectJ进行了很好的集成。

**Spring实现AOP的方式**

- 基于注解的“零配置”方式：使用@Aspect、@Pointcut等注解标注切入点和增强处理；
- 基于XML配置文件的管理方式：使用Spring配置文件来定义切入点和增强处理；



### 学习路径

1. 构建AOP和IOC的demo；
2. 安装spring工具套件STS；
3. 使用无注解方式进行AOP；
4. 使用xml配置方式进行AOP；



