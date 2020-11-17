# Elasticsearch相关技术总结
## 1. centos7 搭建Elasticsearch集群环境
https://www.cnblogs.com/reblue520/p/12219116.html
### 1.1 配置jdk环境变量
vim ~/.bash_profile  
追加如下配置，具体路径根据实际情况进行修改
export JAVA_HOME=/opt/jdk-14.0.2  
export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JAVA_HOME/bin  
export CLASSPATH=./://$JAVA_HOME/lib:$JRE_HOME/lib

### 1.2 创建相关的日志和数据目录
mkdir -p /data/es/data  
mkdir -p /data/es/logs  
mkdir -p /data/es/back

### 1.3 创建用于启动ES的用户
groupadd -g 1500 elasticsearch  
useradd -u 1500 -g elasticsearch elasticsearch  
swapoff -a

### 1.4 修改配置文件
echo "fs.file-max = 1000000" >> /etc/sysctl.conf  
echo "vm.max_map_count=262144" >> /etc/sysctl.conf  
echo "vm.swappiness = 1" >> /etc/sysctl.conf  
sysctl -p  
sed -i 's/* soft nofile 65535/* soft nofile 655350/g' /etc/security/limits.conf  
sed -i 's/* hard nofile 65535/* hard nofile 655350/g' /etc/security/limits.conf  
sed -i 's#*          soft    nproc     4096##' /etc/security/limits.d/20-nproc.conf