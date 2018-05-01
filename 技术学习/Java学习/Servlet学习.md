# Servlet学习笔记

### 简介

servlet通常被称为服务端小程序，是运行在服务器端的程序，用于处理及响应客户端的需求。

Servlet是个特殊的java类，这个类必须继承HtppServlet，每个Servlet可以响应客户端的请求。Servlet提供不同的方法用于影响客户端请求：

- doGet：用于响应客户端的get请求； 
- doPost：用于响应客户端的post请求；
- doPut：用于响应客户端的put请求；
- doDelete：用于响应客户端的delete请求；

大部分时候，Servlet对所有请求的响应都是完全一样的，这个时候，我们就可以采用重写service()方法来代替上面的几个方法。

`void service(HttpServletRequest req, HttpServletResponse resp)`

一般情况下，在MVC应用中，Servlet扮演控制器的角色：

- Model：对应JavaBean；
- View：对应JSP页面；
- Controller：对应Servlet；

### Filter介绍

Filter可以认为是Servlet的一种“加强版”。它主要是对用户请求进行预处理，也可以对HttpServletResponse进行后续处理，是个典型的处理链。使用Filter的完整流程一般是：Filter对用户请求进行预处理，接着请求交给Servlet进行处理并生成响应，最后Filter再对服务器响应进行后续处理。

Filter可以拦截多个请求或响应，一个请求或响应也可以被多个Filter拦截。

创建Filter类需要实现`javax.servlet.Filter`接口，该接口定义了如下三个方法：

- void init(FilterConfig config)
- void destory()
- void doFilter(ServletRequest request, ServletResponse respone, FilterChain chain)

### Listener介绍

当web应用再web容器中运行时，web应用内部会不断的发生各种事件：web应用被启动、web应用被停止、用户session开始、用户session结束、用户请求到达等，通常来这些web事件对开发者是透明的，但是Servlet API提供了大量的监听器来监听web应用的内部事件，从而允许当web内部事件发生时回调事件监听器内的方法。使用Listener需要两步（和Servlet、Filter一样）：

- 定义Listener实现类；
- 通过注解或web.xml配置。

常用的web事件监听器接口有如下几个：

- ServletContextListener：用于监听web应用的启动和关闭；
- ServletContextAttributeListener：用于监听ServletContext范围（applocation）内属性的改变；
- ServletRequestListener：用于监听用户请求；
- ServletRequestAttributeListener：用于监听ServletRequest范围（request）内属性的改变；
- HttpSessionListener：用于监听用户session的开始和结束；
- HttpSessionAttributeListener：用于监听HttpSession范围（session）内属性的改变；

