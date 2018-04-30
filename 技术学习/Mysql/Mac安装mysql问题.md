### 1. 进入mysql后运行命令报错

报错内容： You must reset your password using ALTER USER statement before executing this statement.

**解决方案：**

step 1: SET PASSWORD = PASSWORD\('your new password'\);

step 2: ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;

step 3: flush privileges;

### 2. 更新root密码

```sql
mysql> use mysql;
mysql> UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';
mysql> FLUSH PRIVILEGES;
```





