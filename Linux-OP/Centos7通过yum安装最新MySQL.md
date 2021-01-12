# 1. **去官网查看最新安装包**

```javascript
https://dev.mysql.com/downloads/repo/yum
```

# 2.**下载MySQL源安装包**

```javascript
wget http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

# 3.安装MySql源

```shell
yum -y install mysql57-community-release-el7-11.noarch.rpm
```

# 4.查看一下安装效果

```shell
yum repolist enabled | grep mysql.*
```

# 5.**安装MySQL服务器**

```shell
yum install mysql-community-server
```

# 6.**启动MySQL服务**

```shell
systemctl start  mysqld.service
```

# 7.查看MySQL的运行状态

```shell
systemctl status mysqld.service
```

# 8. **初始化数据库密码**

```shell
grep "password" /var/log/mysqld.log
```

# 9. 使用初始密码登录

```shell
mysql -uroot -p
```

# 10.修改密码

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

# 11.数据库授权

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

# 12.设置开机启动

```shell
systemctl enable mysqld
systemctl daemon-reload
```



