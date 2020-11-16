# MySQL相关技术总结
## 1.Mac下环境搭建
+ 安装包下载  
官网下载mysql dmg文件，安装，并记录下密码
+ 确定MySQL是否已经安装过  
进入/usr/local/mysql/bin,查看此目录下是否有mysql
+ 配置环境变量  
vim ~/.bash_profile
+ 使配置文件生效  
source ~/.bash_profile
+ 修改密码  
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('root123456');

## 2. 授权
### 2.1 添加新用户（允许所有ip访问）
create user '用户名'@'*' identified by '密码';  
grant all privileges on  数据库名称.* to 用户名@'IP' identified by "密码";  
**注意:如果执行命令报错，注意检查分号是中文的还是英文的，是半角的还是全角的**

## 3. 连接远程数据库
mysql -h IP -u用户名 -p密码 -P端口号