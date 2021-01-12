# 1. 编辑密码文件

```shell
vi /etc/my.cnf
```

# 2. 在[mysqld]下最后一行添加一段代码

```shell
skip-grant-tables
```

# 3.重启mysql服务

```shell
service mysqld restart
```

# 4. 用户无密码登录

```shell
mysql -uroot -p # 直接回车，不用输入密码
```

# 5. 选择mysql数据库

```shell
use mysql;
```

# 6. 修改root密码

```sql
update user set authentication_string=password('123456') where user='root';
```

# 7.刷新权限

```sql
flush privileges;
```

# 8.退出mysql

```shell
quit
```

# 9.再次编辑密码文件

```shell
vi /etc/my.cnf
#删除skip-grant-tables
```

# 10. 重启mysql

```shell
service mysqld restart
```

