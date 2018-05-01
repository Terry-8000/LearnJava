### Linux安装Tomcat

```shell
# 下载tar包
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.30/bin/apache-tomcat-8.5.30.tar.gz
# 解压
tar -zxvf apache-tomcat-8.5.30.tar.gz
# 重命名(个人习惯)
mv apache-tomcat-8.5.30 tomcat 8
cd tomcat8/bin
# 启动tomcat服务
sh startup.sh
# 停止tomcat 服务
sh shutdown.sh

# 查询tomcat进程
ps -ef | grep tomcat
```

> 参考：https://blog.csdn.net/lcyaiym/article/details/76696192

