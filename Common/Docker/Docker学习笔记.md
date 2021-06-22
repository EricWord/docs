[TOC]

# 1. 简介

## 1.1 什么是docker?

docker就是容器技术

## 1.2 为什么是docker?

- 优势1:一致的运行环境，更轻松的迁移

  在开发的时候，在本地测试环境可以跑，生产环境跑不起来

- 优势2:对进程进行封装隔离，容器与容器之间互不影响，更高效地利用系统资源

  服务器自己的程序挂了，结果发现是别人程序出了问题把内存吃完了，自己的程序因为内存不够就挂了

- 优势3:通过镜像复制N多个环境一致容器

  公司要弄一个活动，可能会有大量的流量进来，公司需要再多部署几十台服务器

## 1.3 docker与虚拟机的对比

### 1.3.1 虚拟机缺点

- 虚拟机运行软件环境之前必须自身携带操作系统，本身很小的应用程序却因为携带了操作系统而变得非常大，很笨重
- 通过虚拟机在资源调度上经过很多步骤:在调用宿主机的CPU、磁盘等这些资源的时候，拿内存举例，虚拟机是利用Hypervisor去虚拟化内存，整个调用过程是虚拟内存->虚拟物理内存->真正物理内存



### 1.3.2 docker优点

- docker是不携带操作系统的，所以docker的应用就非常轻巧
- docker引擎分配资源直接是 虚拟内存-> 真正物理内存过程
- 

### 1.3.3 两者详细对比

![image-20210622113724277](images/image-20210622113724277.png)





# 2. docker的使用

## 2.1 安装

docker引擎支持主流操作系统 window macOS linux unix

### 2.2.1 bash安装(通用所有平台)

```bash
curl -fsSL get.docker.com -o get-docker.sh

```

## 2.2 启停

### 2.2.1 启动docker服务

```bash
systemctl start docker
```

### 2.2.2 关闭docker服务

```bash
systemctl stop docker
```

### 2.2.3 查看docker的状态

```bash
systemctl status docker
```

### 2.2.4 重启docker服务

```bash
systemctl restart docker
```

### 2.2.5 检查docker是否启动成功

```bash
docker info # 查看docker引擎的版本
```

### 2.2.6 配置开机启动

```bash
systemctl enable docker
```

### 2.2.7 创建docker所属的组

```bash
```



# 3. 核心概念与架构

## 3.1 镜像(image)

### 3.1.1 定义

一个镜像代表着一个软件，如:mysql镜像、redis镜像、 nginx镜像...

### 3.1.2 特点

只读

## 3.2 容器(container)

### 3.2.1 定义

基于某个镜像启动的一个实例称为镜像也可以称之为一个服务

### 3.2.2  特点

可读可写

## 3.3 仓库(repository)

### 3.3.1 定义

用来存储docker中的所有镜像的具体位置

### 3.3.2 远程仓库

docker在世界范围维护一个唯一远程仓库

### 3.3.3 本地仓库

当前自己机器中下载镜像的存储位置



## 3.4 镜像下载加速

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://u9vfcrhl.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 3.5 容器的运行

```bash
docker run 镜像名 # 根据run后面镜像运行一个容器
```

在运行之前先在自己本地仓库中查找对应的镜像，找到的话直接使用，找不到自动去远程仓库下载



## 3.6 docker引擎的相关操作

- `docker info`

  展示docker信息

- `docker version`

  查看docker版本信息

- `docker --help`或者`docker`

  查看docker所有帮助命令

- 执行命令的格式

  `docker [OPTIONS] COMMAND`

  

## 3.7 操作镜像相关的命令

### 3.7.1 查看本机中所有镜像

`docker  images`

- `-a`

  列出所有镜像(包含中间映像层)

- `-q`

  只显示镜像id

### 3.7.2 搜索镜像

`docker search [options] 镜像名`

- `-s 指定值`

  列出收藏数不少于指定值的镜像

- `--no-trunc`

  显示完整的镜像信息

### 3.7.3 从仓库下载镜像

`docker pull 镜像名[:TAG|@DIGEST]`



### 3.7.4 删除镜像

`docker rm 镜像名`

- `-f`

  强制删除

简化写法：`docker rmi 镜像名`



## 3.8 容器相关操作命令

### 3.8.1 查看当前运行的容器

`docker ps`

每一列的含义如下:

- `CONTAINER ID`

  容器id

- `IMAGE`

  基于哪个镜像创建的容器

- `COMMAND`

  容器内执行的命令

- `CREATED`

  容器创建时间

- `STATUS`

  容器当前状态(up/down)

- `PORTS`

  容器内当前服务监听的端口

- `NAMES`

  容器名称

添加`-a`参数可以查看所有运行过的容器

`-q`参数 返回正在运行的容器id

`-qa`参数 返回所有容器的id



### 3.8.2 宿主机端口与容器中端口进行映射

`docker run -p 系统上外部端口号 : 容器内服务监听的端口 镜像名`

注意`-p`可以书写多个，例如:

`docker run -p 系统上外部端口号 : 容器内服务监听的端口 -p 系统上外部端口号 : 容器内服务监听的端口 镜像名`

### 3.8.3 后台启动容器

添加`-d`参数

### 3.8.4 指定容器名称

`docker run --name 容器名称  镜像名称`

### 3.8.5 启停容器

- 开启容器

  `docker start 容器名字或者容器id`

- 重启容器

  `docker restart 容器名或者人容器id`

- 正常停止容器运行

  `docker stop 容器名或者容器id`

- 立即停止容器运行

  `docker kill 容器名或者容器id`

### 3.8.6 删除容器

- 删除容器

  `docker rm -f 容器id或容器名`

- 删除所有容器

  `docker rm -f $(docker ps -aq)`

### 3.8.7 查看容器内进程

`docker top 容器id或者容器名`



### 3.8.8 查看容器内部细节

`docker inspect 容器id`



### 3.8.9 查看容器的运行日志

`docker logs [options] 容器id或者容器名`

- `-t`

  加入时间戳

- `-f`

  跟随最新的日志打印

- `--tail 数字`

  显示最后多少条

### 3.8.10 进入容器内部

`docker exec [options] 容器id 容器内命令`

- `-i`

  以交互模型运行容器，通常与`-t`一起使用

- `-t`

  分配一个伪终端 shell窗口 bash

### 3.8.11 容器和宿主机之间复制文件

- 将宿主机复制到容器内部

  `docker cp 文件或目录 容器id:容器路径`

- 将容器内资源拷贝到主机上

  `docker cp 容器id:容器内资源路径 宿主机目录路径`

### 3.8.12 数据卷(volum)实现与宿主机共享目录

`docker run -v 宿主机的路径|任意别名:/容器内的路径 镜像名`

注意:

1. 如果是宿主机路径必须是绝对路径，宿主机目录会覆盖容器内目录内容
2. 如果是别名则会在docker运行容器时自动在宿主机中创建一个目录，并将容器目录文件复制到宿主机中

### 3.8.13 打包镜像

`docker save 镜像名 -o 名称.tar`



### 3.8.14 载入镜像

`docker load -i 名称.tar`

### 3.8.15 容器打包成新的镜像

`docker commit -m "描述信息" -a "作者信息" (容器id或者名称) 打包的镜像名称:标签`



## 3.9 docker镜像分层原理

### 3.9.1 镜像为什么这么大？

一个软件镜像不仅仅是原来的软件包，镜像包含软件所需要的操作系统依赖、软件自身依赖以及自身软件包

### 3.9.2 为什么docker镜像采用分层镜像原理

docker在设计镜像时每一个镜像都是由n个镜像共同组成

底层原理是UnionFS:

Union文件系统是一种分层，轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层地叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。Union文件系统是Docker镜像的基础。这种文件系统特性:就是一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

### 3.9.3 为什么采用UnionFS

最大的一个好处就是资源共享，采用分层机制实现基础层共享，从而减小docker仓库整体体积

## 3.10 docker容器之间的网络配置

一般在使用docker网桥(bridge)实现容器与容器通信时，都是站在一个应用角度进行容器通信

### 3.10.1 查看docker网桥配置

`docker network ls`

### 3.10.2 创建自定义网桥

`docker create 网桥名称`

注意:一旦在启动容器时指定了网桥之后，日后可以在任何这个网桥关联的容器中使用容器名字进行与其他容器通信

### 3.10.3 删除网桥

`docker network rm 网桥名称`



## 3.11 数据卷

### 3.11.1  特点

- 数据卷可以在容器之间共享和重用
- 对数据卷的修改会立马生效
- 对数据卷的更新不会影响镜像
- 数据卷默认会一直存在，即使容器被删除

### 3.11.2 自定义数据卷目录

`docker run -v 绝对路径:容器内路径`

### 3.11.3 自动创建数据卷

`docker run -v 卷名:容器内路径`

### 3.11.4 docker操作数据卷的指令

- 查看数据卷

  `docker volume ls`

- 查看某个数据卷的细节

  `docker volume inspect 卷名`

- 创建数据卷

  `docker volume crete 卷名`

- 删除所有没有使用的数据卷

  `docker volume prune`

  

  该看14

  

