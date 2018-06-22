# Tomcat常用配置

## 一、server.xml中

### 1. Context配置

```xml
<Context path="/bes" docBase="bes" reloadable="false" /> 
```

每个`<Context>`元素代表了运行在虚拟主机上的单个Web应用。

1. path：即要建立的虚拟目录，注意是/bes，它指定访问Web应用的URL入口，如http://localhost:8080/kaka/。
2. docBase：为实际目录在硬盘上的位置（应用程序的路径或者是WAR文件存放的路径）。
3. reloadable：如果这个属性设为true，Tomcat服务器在运行状态下会监视在WEB-INF/classes和Web-INF/lib目录CLASS文件的改变，如果监视到有class文件被更新，服务器自动重新加载Web应用，这样我们可以在不重起tomcat的情况下改变应用程序。

一个Host元素中嵌套任意多的Context元素。每个Context的路径必须是惟一的，由path属性定义。另外，你必须定义一个path=“”的context，这个Context称为该虚拟主机的缺省web应用，用来处理那些不能匹配任何Context的Context路径的请求。

### 2. Listener配置

```xml
<!-- 当Tomcat启动时，该监听器记录Tomcat、Java和操作系统的信息。该监听器必须是配置的第一个监听器。 -->
<Listener className="org.apache.catalina.startup.VersionLoggerListener" />
<!-- Tomcat启动时，检查APR库，如果存在则加载。APR，即Apache Portable Runtime，是Apache可移植运行库，可以实现高可扩展性、高性能，以及与本地服务器技术更好的集成。 -->
<Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" /> 
<!--  与类加载器导致的内存泄露有关。 -->
<Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
```

Listener(即监听器)定义的组件，可以在特定事件发生时执行特定的操作；被监听的事件通常是Tomcat的启动和停止。

## 二、context.xml中

#### 在tomcat 5.5之后：

不推荐在server.xml中进行配置Context，而是在/conf/context.xml中进行独立的配置。因为server.xml是不可动态重加载的资源，服务器一旦启动了以后，要修改这个文件，就得重启服务器才能重新加载。而context.xml文件则不然，tomcat服务器会定时去扫描这个文件。一旦发现文件被修改（时间戳改变了），就会自动重新加载这个文件，而不需要重启服务器。

### 1. Loader配置

```xml
<Loader delegate="false" loaderClass="com.huawei.bes.common.extension.ext.adapter.openas.ExtensionWebappClassLoader"/>
```

tomcat的类加载机制，delegate为true的时候遵循JVM的类加载机制，即双亲委派模型。如果为false则不遵循JVM的类加载机制，即优先使用自己的类加载器，如果无法加载，再委托给父类加载器。

### 2. Resources配置

用于配置数据源，比如数据库URL地址，用户名密码等

### 3. WatchedResource配置

```xml
<WatchedResource>WEB-INF/web.xml</WatchedResource>  
```

监控资源文件，如果web.xml文件改变了，则自动重新加载应用。




