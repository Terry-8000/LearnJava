# Linux常用命令

**ps    命令用于显示当前进程 (process) 的状态**

ps -ef  e：显示所有进程；f：全格式 。

**netstat    显示网络状态**

netstat -anp | grep 8080   查询占用8080端口的进程号  a：所有连线中的socket；n：直接使用IP地址，而不通过域名服务器；p：显示正在使用Socket的程序识别码和程序名称。



### 包管理

apt-get install package-name  安装软件包

apt-cache search package-name  搜索软件源中可安装的软件包



### 解压：

tar -zxvf     eg：tar -zxvf jdk-8u101-linux-x64.tar.gz



### find命令

find -maxdepth     查找的最大深度。eg：find . -maxdepth 2 -name "Wuziqi.class"

find -type   查找某一类型的文件，eg：d - 目录；l - 符号链接文件；f - 普通文件。