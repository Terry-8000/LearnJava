# MAC下GitHub问题

## 1. 每次push都需要重新输入用户名密码

#### 背景

最近改了一下某个项目的名字，之后每次push都需要重新输入用户名密码，鼓捣了半天，终于在网上找到了解决办法。

#### 解决方案

1. 在命令行输入命令:

```
git config --global credential.helper store
```

这一步会在用户目录下的.gitconfig文件最后添加:

```
[credential]
        helper = store
```

2. 现在push你的代码 ，这时会让你输入用户名密码，这一步输入的用户名密码会被记住，下次再push代码时就不用输入用户名密码了。

```shell
 vim ~/.git-credentials
```

可以看到在这个文件存储了用户密码。

