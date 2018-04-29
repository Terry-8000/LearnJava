### 问题1：配置eclipse的workspace

**场景：**

不下心删除了workspace下面的隐藏文件夹`.metadata`，导致无法再次打开eclipse了，会报错Could not write metadata for …，提醒日志在.metadata下面的log中。于是想重新配置一下workspace，使其自动生成.metadata文件夹。

**解决方案：**

在应用程序中找到eclipse.app右键—>显示包内容，进入eclipse的安装目录，找到这个文件，

```
Eclipse.app/Contents/Eclipse/configuration/.settings/org.eclipse.ui.ide.prefs
```

修改`RECENT_WORKSPACES`参数，重新打开eclipse即可。

### 问题2：Eclipse无法反编译maven中依赖的jar包中的Class文件

**场景：**

eclipse可以看到jdk的源码，但是看不到maven下载下来的外部依赖的源码，比如Spring源码。

**解决方案：**

打开help->Eclipse Marketplace，搜索jd，出来的只有一个Enhanced Class Decompiler 3.0.0，点击install重启就好了。