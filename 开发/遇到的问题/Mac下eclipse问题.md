### 问题：配置eclipse的workspace

**场景：**

不下心删除了workspace下面的隐藏文件夹`.metadata`，导致无法再次打开eclipse了，会报错，提醒日志在.metadata下面的log中。于是想重新配置一下workspace，使其自动生成.metadata文件夹。

**解决方案：**

在应用程序中找到eclipse.app右键—>显示包内容，进入eclipse的安装目录，找到这个文件，

```
Eclipse.app/Contents/Eclipse/configuration/.settings/org.eclipse.ui.ide.prefs
```

修改`RECENT_WORKSPACES`参数，重新打开eclipse即可。

