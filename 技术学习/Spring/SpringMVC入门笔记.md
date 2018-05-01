# SpringMVC入门笔记

### 简介

Spring MVC是Spring系开源项目中的一个，和IoC配合使用。通过策略接口，Spring框架是高度可配置的，而且支持多种视图技术。Spring分离了控制器、模型对象、分派器以及处理程序对象的角色，这种分离让他们更容易进行定制。Spring MVC解决的问题如下：

- 将web页面的请求传给服务器；
- 根据不同的请求处理不同的逻辑单元；
- 返回处理结果数据并跳转至响应页面；

### Spring MVC实例

1. 配置web.xml。一个web钟可以没有web.xml，它主要用来初始化配置信息：比如welcome页面、servlet、servlet-mapping、filter、listener、启动加载级别等。但是Spring MVC的实现是通过Servlet拦截所有的URL来达到控制的目的，所以web.xml是必须配置的。

   ​

2. 创建Spring配置文件。

3. 创建model。

4. 创建controller。

5. 创建视图文件jsp。

6. 创建servlet配置文件。