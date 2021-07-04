[TOC]



# 1. 简介

## 1.1 应用部署方式演变

在部署应用程序方式上，主要经历了3个时代

### 1.1.1 传统部署

互联网早期，会直接将应用程序部署在物理机上

- 优点

  简单，不需要其他技术的参与

- 缺点

  不能为应用程序定义资源使用边界，很难合理地分配计算资源，而且程序之间容易产生影响

### 1.1.2 虚拟化部署

可以在一台物理机上运行多个虚拟机，每个虚拟机都是独立的一个环境

- 优点

  程序环境不会相互产生影响，提供了一定程度的安全性

- 缺点

  增加了操作系统，浪费了部分资源

### 1.1.3 容器化部署

与虚拟化类似，但是共享了操作系统

- 优点

  可以保证每个容器拥有自己的文件系统、CPU、内存、进程空间等

  运行应用程序所需要的资源都被容器包装并和底层基础架构解耦

   容器化的应用程序可以跨云服务商、跨Linux操作系统发行版进行部署

![image-20210624181347745](images/image-20210624181347745.png)

容器化部署方式带来了很多便利，但是也会出现一些问题，比如说:

- 一个容器故障停机了，怎么样让另外一个容器立刻启动去替补停机的容器
- 当并发访问量变大的时候，怎么样做到横向扩展容器数量

这些容器管理的问题统称为容器编排问题，为了解决这些容器编排问题，就产生了一些容器编排的软件:

- Swarm:Docker自己的 容器编排工具
- Mesos:Apache的一个资源统一管控的工具，需要和Marathon结合使用
- Kubernetes：Google开源的容器编排工具



## 1.2 K8S简介

Kubernetes是一个全新的基于容器技术的分布式架构领先方案，是谷歌严格保密十几年的秘密武器--Borg系统的一个开源版本，于2014年9月发布第一个版本，2015年7月发布第一个正式版本

Kubernetes的本质是一组服务器集群，它可以在集群的每个节点上运行特定的程序，来对节点的容器进行管理。它的目的就是实现资源管理的自动化，主要提供了如下的主要功能:

- 自我修复

  一旦某个容器奔溃，能够在1秒左右迅速启动新的容器

- 弹性伸缩

  可以根据需要，自动对集群中正在运行的容器数量进行调整

- 服务发现

  服务可以通过自动发现的形式找到它所依赖的服务

- 负载均衡

  如果一个服务启动了多个容器，能够自动实现请求的负载均衡

- 版本回退

  如果发现新发布的程序版本有问题，可以立即回退到原来的版本

- 存储编排

  可以根据容器自身的需求自动创建存储卷

![image-20210624195847496](images/image-20210624195847496.png)

## 1.3 K8S组件

一个K8S集群主要是由控制节点(master)、工作节点(node)构成，每个节点上都会安装不同的组件。

 master:集群的控制平面，负责集群的决策

- ApiServer

  资源操作的唯一入口，接收用户输入的命令，提供认证、授权、API注册和发现等机制

- Scheduler

  负责集群资源调度，按照预定的调度策略将Pod调度到相应的node节点上

- ControllerManger

  负责维护集群的状态，比如程序部署安排、故障检测、自动扩展，滚动更新等

- Etcd

  负责存储集群中各种资源对象的信息

node:集群的数据平面，负责为容器提供运行环境

- Kubelet

  负责维护容器的生命周期，即通过控制docker，来创建、更新、销毁容器

- KubeProxy

  负责提供集群内部的服务发现和负载均衡

- Docker

  负责节点上容器的各种操作

![image-20210624202310303](images/image-20210624202310303.png)

下面以部署一个nginx服务来说明K8S系统各个组件调用关系:

1. 首先要明确，一个K8S环境启动之后，master和node都会将自身的信息存储到etcd数据库中
2. 一个nginx服务的安装请求首先会被发送到master节点的apiServer组件
3. apiServer组件会调用scheduler组件来决定到底应该把这个服务安装到哪个node节点上，在此时，它会从etcd中读取各个node节点的信息，然后按照一定的算法进行选择，并将结果告知apiServer
4. apiServer调用controller-manager去调度Node节点安装nginx服务
5. kubelet接收到指令后，会通知docker，然后由docker来启动一个nginx的pod，pod是K8S的最小操作单元，容器必须跑在pod中
6. 一个nginx服务就运行了，如果需要访问nginx,就需要通过kube-proxy来对pod产生访问的代理，这样，外界用户就可以访问集群中的nginx服务了。

## 1.4 K8S概念

- Master

  集群控制节点，每个集群需要至少一个master节点负责集群的管控

- Node

  工作负载节点，由master分配容器到这些node工作节点上，然后node节点上的docker负责容器的运行

- Pod

  K8S的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器

- Controller

  控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等

- Service

  pod对外服务的统一入口，下面可以维护着同一类的多个pod

- Label

  标签，用于对pod进行分类，同一类pod会拥有相同的标签

- NameSpace

  命名空间，用来隔离pod的运行环境

![image-20210624204551566](images/image-20210624204551566.png)

# 2. 集群环境搭建

## 2.1 环境规划

### 2.1.1 集群类型

K8S集群大体上分为两类：一主多从和多主多从

- 一主多从

  一台Master节点和多台Node节点，搭建简单，但是有单机故障风险，适合用于测试环境

- 多主多从

  多台Master节点和多台Node节点，搭建麻烦，安全性高，适合用于生产环境

![image-20210624205146395](images/image-20210624205146395.png)

### 2.1.2 安装方式

K8S有多种不舒服方式，目前主流的方式有kubeadm、minikube、二进制包

- minikube

  一个用于快速搭建单节点K8S的工具

- kubeadm

  一个用于快速搭建kubernetes集群的工具

- 二进制包

  从官网下载每个组件的二进制包，依次去安装，此方式对于理解K8S组件更加更有效

  

### 2.1.3 主机规划

![image-20210624205700767](images/image-20210624205700767.png)

## 2.2 环境搭建

### 2.2.1 环境初始化

1. 检查操作系统的版本

   此方式下安装K8s要求Centos版本在7.5及以上

2. 主机名解析

   测试的时候配置`/etc/hosts`文件，企业中推荐使用内部DNS服务器

3. 时间同步

   K8s要求集群中的节点时间必须精确一致，这里直接使用`chronyd`服务从网络同步时间，企业中建议配置内部的时间同步服务器

4. 禁用`iptables`和`firewalld`服务

   K8s和docker在运行中会产生大量的iptables规则，为了不让系统规则跟它们混淆，直接关闭系统的规则

   ```bash
   # 关闭firewall服务
   systemctl stop firewalld
   systemctl disable firewalld
   
   # 关闭iptables服务
   systemctl stop iptables
   systemctl disabele iptalbes
   ```

5. 禁用`selinux`

   selinux是linux系统下的一个安全服务，如果不关闭它，在安装集群中会产生各种各样的奇葩问题

   ```bash
   # 编辑 /etc/selinux/config文件，修改SELINUX的值为disabled
   # 注意修改完毕之后需要重启linux服务
   SELINUX=disabled
   ```

6. 禁用swap分区

   swap分区指的是虚拟内存分区，它的作用是在物理内存使用完之后，将磁盘空间虚拟成内存来使用。启用swap设备会对系统产生非常负面的影响，因此K8s要求每个节点都要禁用swap设备 ，但是如果因为某些原因确实不能关闭swap分区，就需要在集群安装过程中通过明确的参数进行配置说明

   ```bash
   # 编辑分区配置文件 /etc/fstab,注释掉swap分区一行
   # 注意修改完毕之后需要重启Linux服务
   ```

7. 修改linux的内核参数

   ```bash
   # 修改linux的内核参数，添加网桥过滤和地址转发功能
   # 编辑/etc/sysctl.d/kubernetes.conf文件，添加如下配置
   net.bridge-nf-call-ip6tables = 1
   net.bridge-nf-call-iptables = 1
   net.ipv4.ip_forward = 1
   ```

   重新加载配置

   ```bash
   sysctl -p
   ```

   加载网桥过滤模块

   ```bash
   modprobe br_netfilter
   ```

   查看网桥过滤模块是否加载成功

   ```bash
   lsmod | grep br_netfilter
   ```

8. 配置ipvs功能

   K8s中的service有两种代理模型，一种基于iptables的，一种是基于ipvs的，两者比较的话，ipvs的性能明显要高一些，但是如果要使用它，需要手动载入ipvs模块

   ```bash
   # 1.安装ipset和ipvsadm
   yum install ipset ipvsadm -y
   
   # 2.添加需要加载的模块写入脚本文件
   cat <<EOF > /etc/sysconfig/modules/ipvs.modules
   #!/bin/bash
   modprobe -- ip_vs
   modprobe -- ip_vs_rr
   modprobe -- ip_vs_wrr
   modprobe -- ip_vs_sh
   modprobe -- nf_connntrack_ipv4
   EOF
   
   # 3.为脚本文件添加执行权限
   chmod +x /etc/sysconfig/modules/ipvs.modules
   
   # 4.执行脚本文件
   /bin/bash	/etc/sysconfig/modules/ipvs.modules
   
   # 5.查看对应的模块是否加载成功
   lsmod | grep -e ip_vs -e nf_conntrack_ipv4
   
   ```

9. 重启服务器

### 2.2.2 安装docker

1. 切换镜像源头

   ```bash
   wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
   ```

   

2. 查看当前镜像源中支持的docker版本

   ```bash
   yum list docker-ce --showuplicates
   ```

   

3. 安装特定版本的docker-ce

   ```bash
   # 必须指定 --setopt=obsoletes=0,否则yum会自动安装更高版本
   yum install--setopt=obsletes=0 docker-ce-18.06.3.ce-3.el7 -y
   ```

   

4. 添加一个配置文件

   ```bash
   # docker在默认情况下使用的是Cgroup Driver为cgroupfs,而kubernetes推荐使用systemd来代替#cgroupfs
   mkdir /etc/docker
   cat <<EOF > /etc/docker/daemon.json
   {
   	"exec-opts":["nataive.cgroupdriver=systemd"]
   	"registry-mirrors":["https://kn0t2bca.mirror.aliyuncs.com"]
   }
   EOF
   
   ```

   

5. 启动docker

   ```bash
   systemctl restart docker
   systemctl enable docker
   ```

   

6. 检查docker状态和版本

   ```bash
   docker version
   ```

### 2.2.3 安装K8s组件

```bash
#由于K8s的镜像源在国外，速度比较慢，这里切换成国内的镜像源
#编辑/etc/yum.repos.d/kubernetes.repo,添加下面的配置
[kubernetes]
name=Kubernetes
baseurl=http://mirror.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
       
# 安装kubeadm、kubelet和kubectl
yum install --setopt=obsoletes=0 kubeadm-1.17.4.0 kubelet-1.17.4-0 kubectl-1.17.4-0 -y

# 配置kubelet的cgroup
# 编辑/etc/sysconfig/kubelet,添加下面的配置
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MODE="ipvs"

# 设置kubelet开机自启
systemctl enable kubelet


```

### 2.2.4 准备集群镜像

![image-20210629220251260](images/image-20210629220251260.png)

### 2.2.5 集群初始化

下面的操作只需要在master节点上执行即可

1. 创建集群

   ```bash
   kubeadm init \
   --kubernetes-version=v1.17.4 \
   --pod-network-cidr=10.244.0.0/16 \
   --service-cidr=10.96.0.0/12 \
   --apiserver-advertise-address=192.168.109.100
   ```

2. 创建必要的文件

   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

下面的操作只需要在node节点上执行即可

将node节点加入集群

![image-20210629220045269](images/image-20210629220045269.png)

```bash
# 查看集群的状态，此时的集群状态为NotReady,这是因为还没有配置网络插件
kubectl get nodes
```

### 2.2.6 安装网络插件

![image-20210629221422894](images/image-20210629221422894.png)



## 2.3 服务部署

```bash
# 部署nginx
kubectl create deployment nginx --image=nginx:1.14-alpine
# 暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort
# 查看服务状态
kubectl get pods,svc
# 最后在电脑上访问部署的nginx服务
```

![image-20210630093141487](images/image-20210630093141487.png)



# 3. 资源管理

## 3.1 资源管理介绍

在K8s中，所有的内容都抽象为资源，用户需要通过操作资源来管理K8s

> K8s的本质是一个集群系统，用户可以在集群中部署各种服务，所谓的部署服务，其实就是在K8s集群中运行一个个的容器，并将指定的程序跑在容器中
>
> K8s的最小管理单元是pod而不是容器，所以只能将容器放在pod中，而K8s一般也不会直接管理pod，而是通过Pod控制器来管理pod的
>
> pod可以提供服务之后，就要考虑如何访问pod中服务，K8s提供了Service资源实现这个功能。
>
> 当然，如果pod中的程序的数据需要持久化，K8s还提供了各种存储系统

![image-20210630093942826](images/image-20210630093942826.png)

学习K8s的核心，就是学习如何对集群上的Pod、Pod控制器、Service、存储等各种资源进行操作



## 3.2 YAML语法介绍

![image-20210630094240291](images/image-20210630094240291.png)

![image-20210630094842065](images/image-20210630094842065.png)

![image-20210630095205789](images/image-20210630095205789.png)

![image-20210630095251949](images/image-20210630095251949.png)





## 3.3 资源管理方式



### 3.3.1 命令式对象管理

直接使用命令去操作kubernetes资源

```bash
kubectl run nginx-pod --image=nginx:1.17.1 --port=80
```

- kubectl命令

  kubectl是K8s集群的命令行工具，通过它能对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。kubectl命令的语法如下:

  ```bash
  kubectl [command] [type] [name] [flags]
  ```

- command

  指定要对资源执行的操作，例如create、get、delete

- type

  指定资源类型，比如deployment、pod、service

- name

  指定资源的名称，名称大小写敏感

- flags

  指定额外的可选参数

  ```bash
  # 查看所有pod
  kubectl get pod
  # 查看某个pod
  kubectl get pod pod_name
  # 查看某个pod,以yaml格式展示结果
  kubectl get pod pod_name -o yaml
  ```

资源类型

K8s中所有的内容都抽象为资源，可以通过下面的命令进行查看:

```bash
kubectl api-resources
```

经常使用的资源有下面这些:

![image-20210630144022578](images/image-20210630144022578.png)

![image-20210630144103632](images/image-20210630144103632.png)

操作

K8s允许对资源进行多种操作，可以通过`--help`查看详细的操作命令

```bash
kubectl --help
```

经常使用的操作有下面这些:

![image-20210630144411288](images/image-20210630144411288.png)

![image-20210630144348027](images/image-20210630144348027.png)

下面以一个namespace/pod的创建和删除简单演示下命令的使用:

![image-20210630144520928](images/image-20210630144520928.png)



### 3.3.2 命令式对象配置

通过命令配置和配置文件去操作kubernetes资源

```bash
kubectl create/patch -f nginx-pod.yaml
```

1. 创建一个nginxpod.yaml，内容如下:

![image-20210630145305669](images/image-20210630145305669.png)

2. 执行create命令，创建资源:

   ```bash
   kubectl create -f nginxpod.yaml
   ```

   此时发现创建了两个资源对象，分别是namespace和pod

3. 执行get命令，查看资源

   ```bash
   kubectl get -f nginxpod.yaml
   ```

   这样就显示了两个资源对象的信息

4. 执行delete命令，删除资源

   ```bash
   kubectl delete -f nginxpod.yaml
   ```

   此时发现两个资源对象被删除了

5. 总结

   命令式对象配置的方式操作资源，可以简单地认为:命令 + yaml配置文件(里面是命令需要的各种参数)

### 3.3.3 声明式对象配置

通过apply命令和配置文件去操作kubernetes资源

```bash
kubectl apply -f nginx-pod.yaml
```

![image-20210630150117824](images/image-20210630150117824.png)

总结：

其实声明式对象配置就是使用apply描述一个资源最终的状态(在yaml中定义状态)

使用apply操作资源:

- 如果资源不存在，就创建，相当于 `kubectl create`
- 如果资源已存在，就更新，相当于`kubectl patch`

> 扩展：kubectl可以在node节点上运行吗？
>
> kubectl的运行是需要进行配置的，它的配置文件是$HOME/.kube,如果想要在node节点上运行此命令，需要将master上的`.kube`文件复制到node节点上，即在master节点上执行下面操作:
>
> `scp -r  $HOME/.kube node1:$HOME/`



> 使用推荐：三种方式应该怎么用？
>
> - 创建/更新资源
>
>   使用声明式对象配置 `kubectl apply -f xxx.yaml`
>
> - 删除资源
>
>   使用命令式对象配置 `kubectl delete -f xxx.yaml`
>
> - 查询资源
>
>   使用命令式对象管理 `kubectl get(describe) 资源名称`

![image-20210630105738010](images/image-20210630105738010.png)



# 4. 实战入门

本章节将介绍如何在K8s集群中部署一个nginx服务，并且能够对其进行访问

## 4.1 Namespace

Namespace是K8s系统中的一种非常重要的资源，主要作用是用来实现多套环境的资源隔离或者多租户的资源隔离

默认情况下，K8s集群中的所有pod都是可以相互访问的，但是实际中，可能不想让两个pod之间进行相互的访问，那此时可以将两个pod划分到不同的namespace下，K8s通过将集群内部的资源分配到不同的namespace中，可以形成逻辑上的"组"，以方便不同的组的资源进行隔离使用和管理

可以通过K8s的授权机制，将不同的namespace交给不同租户进行管理，这样就实现了多租户的资源隔离。此时还能结合K8s的资源配额机制，限定不同租户能占用的资源，例如CPU使用量、内存使用量等来实现租户可用资源的管理。

![image-20210630152242459](images/image-20210630152242459.png)

![image-20210630152422752](images/image-20210630152422752.png)

下面是namespace资源的具体操作:

- 查看

  ![image-20210630152529909](images/image-20210630152529909.png)

  


  ![image-20210630153039998](images/image-20210630153039998.png)

  

- 创建

  ![image-20210630153142018](images/image-20210630153142018.png)

- 删除

  ![image-20210630153151935](images/image-20210630153151935.png)

- 使用配置的方式

  首先准备一个yaml文件:ns-dev.yaml

  ![image-20210630153312059](images/image-20210630153312059.png)

  

## 4.2 Pod



Pod是K8s集群进行管理的最小单元，程序要运行必须部署在容器中，而容器必须存在于Pod中。

Pod可以认为是容器的封装，一个Pod中可以存在一个或多个容器

![image-20210630153501924](images/image-20210630153501924.png)

K8s在集群启动之后，集群中的各个组件也都是以Pod方式运行的，可以通过下面的命令查看:

```bash
kubectl get pod -n kube-system
```

**创建并运行**

![image-20210630153924250](images/image-20210630153924250.png)

**查看pod信息**

![image-20210630154100580](images/image-20210630154100580.png)

**访问pod**

![image-20210630155132114](images/image-20210630155132114.png)

**删除指定的pod**

![image-20210630155351687](images/image-20210630155351687.png)

**基于配置操作**

![image-20210630155527440](images/image-20210630155527440.png)



## 4.3 Label

Label是K8s集群中的一个重要概念，作用是在资源上添加标识，用来对它们进行区分和选择

Lable的特点:

- 一个Label以key/value键值对的形式附加到各种对象上，如Node、Pod、Service等
- 一个资源对象可以定义任意数量的Label，同一个Label也可以被添加到任意数量的资源对象上
- Label通常在资源对象定义时确定，当然也可以在对象创建后动态添加或者删除

可以通过Label实现资源的多维度分组，以便灵活、方便地进行资源分配、调度、配置、部署等管理工作

> 一些常用的Label示例如下:
>
> - 版本标签
>
>   `"version":"release","version":"stable"`
>
> - 环境标签
>
>   `"environment":"dev","environment":"test","environment":"pro"`
>
> - 架构标签
>
>   `"tier":"frontend","tier":"backend"`

标签定义完毕之后，还要考虑到标签的选择，这就要使用到Label Selector,即:

- Label用于给某个资源对象定义标识
- Label Selector用于查询和筛选拥有某些标签的资源对象

当前有两种Label Selector

- 基于等式的Label Selector

  name = slave :选择所有包含Label中key="name"且value="slave"的对象

  env!=production: 选择所有包括Label中的key="env"且value不等于"production"的对象

- 基于集合的Label Selector

  name in (master,slave): 选择所有包含Label中的key="name"且value="master"或"slave"的对象

  name not in (frontend):选择所有包含Label中的key="name"且value不等于"frontend"的对象

标签的选择条件可以使用多个，此时将多个 Label Selector进行组合，使用逗号进行分隔即可。例如:

```bash
name=salve,env!=production
name not in (frontend),env!=production
```

**命令方式**

![image-20210630162011318](images/image-20210630162011318.png)

**配置方式**

![image-20210630165417841](images/image-20210630165417841.png)

## 4.4 Deployment

在K8s中，Pod是最小的控制单元，但是K8s很少直接控制Pod，一般都是通过Pod控制器来完成的。Pod控制器用于Pod的管理，确保pod资源符合预期的状态，当pod的资源出现故障时，会尝试进行重启或重建pod

在K8s中pod控制器的种类有很多，本章节只介绍一种:Deployment



![image-20210630165603448](images/image-20210630165603448.png)

**命令操作**

![image-20210630182928378](images/image-20210630182928378.png)

![image-20210630191856020](images/image-20210630191856020.png)

**配置操作**

![image-20210630192001886](images/image-20210630192001886.png)

![image-20210630192218094](images/image-20210630192218094.png)

## 4.5 Service

虽然每个Pod都会分配一个单独的Pod IP,然而却存在如下两个问题:

- Pod IP会随着Pod的重建产生变化
- Pod IP仅仅是集群内可见的虚拟IP,外部无法访问

这样对于访问这个服务带来了难度，因此K8s设计了Service来解决这个问题

Service可以看作是一组同类Pod对外的访问接口。借助Service，应用可以方便地实现服务发现和负载均衡

![image-20210630200440100](images/image-20210630200440100.png)

**操作一:创建集群内部可访问的Service**

![image-20210630200947239](images/image-20210630200947239.png)

**操作二:创建集群外部也可以访问的Service**

![image-20210630201347586](images/image-20210630201347586.png)

**删除Service**

![image-20210630201400391](images/image-20210630201400391.png)



**配置方式**

![image-20210630201625978](images/image-20210630201625978.png)



# 5. Pod详解

## 5.1 Pod介绍

### 5.1.1 Pod结构

![image-20210701085312108](images/image-20210701085312108.png)

每个Pod都可以包含一个或者多个容器，这些容器可以分为两类：

- 用户程序所在的容器，数量可多可少

- Pause容器，这是每个Pod都会有的一个根容器，它的作用有两个:

  - 可以以它为依据，评估整个Pod的健康状态

    - 可以在根容器上设置IP地址，其他容器都共享此IP,以实现Pod内部的网络通信

    > 这里是Pod内部的通讯，Pod之间的通讯采用虚拟二层网络技术来实现，当前环境用的是Flannel



### 5.1.2 Pod定义

下面是Pod的资源清单：

![image-20210701090327947](images/image-20210701090327947.png)

在K8s中基本所有资源的一级属性都是一样的，主要包含5部分:

- apiVersion <string>

  版本，由K8s内部定义，版本号必须可以用`kubectl api-version`查询到

- kind <string>

  类型，由K8s内部定义，版本号必须可以用`kubectl api-resources`查询到

- metadata <Object>

  元数据，主要是资源标识和说明，常用的有name,namespace,labels等

- spec <Object>

  描述，这是配置中最重要的一部分，里面是对各种资源配置的详细描述

- status <Object>

  状态信息，里面的内容不需要定义，由K8s自动生成

在上面的属性中，spec是接下来研究的重点，下面是spec的常见子属性:

- containers <[]Object>

  容器列表，用于定义容器的详细信息

- nodeName <String>

  根据nodeName的值将pod调度到指定的Node节点上

- nodeSelector <map[]>

  根据NodeSelector中定义的信息选择将该Pod调度到包含这些label的Node上

- hostNetwork <boolean>

  是否使用主机网络，默认为false，如果设置为true，表示使用宿主机网路

- volumes <[]Object>

  存储卷，用于定义Pod上面挂载的存储信息

- restartPolicy <string>

  重启策略，表示Pod在遇到故障时的处理策略



## 5.2 Pod配置

本小节主要来研究`pod.spec.containers`属性，这也是pod配置中最为关键的一项配置

![image-20210701094745458](images/image-20210701094745458.png)

### 5.2.1 基本配置

### 5.2.2 镜像拉取

`imagePullPolicy`用于设置镜像拉取策略，K8s支持配置3种拉取策略

- `always`

  总是从远程仓库拉取镜像(一直用远程的)

- `IfNotPresent`

  本地有则使用本地镜像，本地没有则从远程仓库拉取镜像(本地有就本地，本地没有就远程)

- `Never`

  只使用本地镜像，从不去远程仓库拉取，本地没有就报错(一直使用本地)

> 默认值说明
>
> - 如果镜像tag为具体版本号，默认策略是：`IfNotPresent`
> - 如果镜像tag为：latest,默认策略是：`always`

### 5.2.3 启动命令

command用于在pod中的容器初始化完毕之后运行一个命令

进入某个pod的某个容器的命令：

```bash
kubectl exec pod名称 -n 名称空间 -it -c 容器名称 /bin/sh
```

> 特别说明
>
> ​	通过上面发现command已经可以完成启动命令和传递参数的功能，为什么这里还要提供一个`args`选项用于传递参数呢？这其实跟docker有点关系，`K8s`中的command、`args`两项其实是实现覆盖`Dockerfile`中`ENTRYPOINT`的功能.
>
> - 如果command和`args`均没有写，那么用`Dockerfile`的配置
> - 如果command写了，但`args`没有写，那么`Dockerfile`默认的配置会被忽略，执行输入的command
> - 如果command没写，但是`args`写了，那么`Dockerfile`中配置的`ENTRYPOINT`的命令会被执行，使用当前`args`的参数
> - 如果command和`args`都写了，那么`Dockerfile`的配置被忽略，执行command并追加上`args`参数



### 5.2.4 环境变量



env,环境变量，用于在pod中的容器设置环境变量

这种方式不是很推荐，推荐将这些配置单独存储在配置文件中

### 5.2.5 端口设置

ports支持的子选项：

![image-20210701153657327](images/image-20210701153657327.png)

![image-20210701153735752](images/image-20210701153735752.png)

![image-20210701153747267](images/image-20210701153747267.png)



### 5.2.6 资源配额

容器中的程序要运行，肯定是要占用一定资源的，比如CPU和内存等，如果不对某个容器的资源做限制，那么它就可能吃掉大量资源，导致其他容器无法运行。针对这种情况，K8s提供了对内存和cpu的资源进行配额的机制，这种机制主要通过resource选项实现。它有两个子选项:

- limits

  用于限制运行时容器的最大占用资源，当容器占用资源超过limits时会被终止，并进行重启

- requests

  用于设置容器需要的最小资源，如果环境资源不够，容器将无法启动

可以通过上面两个选项设置资源的上下限

![image-20210701154428828](images/image-20210701154428828.png)

在这对cpu和memory的单位做一个说明

- cpu

  core数，可以为整数或小数

- memory

  内存大小，可以使用Gi、Mi、G、M等形式



## 5.3 Pod生命周期

我们一般将pod对象从创建至终的这段时间范围称为pod的生命周期，它主要包含下面的过程:

- pod创建过程
- 运行初始化容器(init container)过程
- 运行主容器(main container)过程
  - 容器启动后钩子(post start)、容器终止前钩子(pre stop)
  - 容器的存活性探测(liveness probe)、就绪性探测(readiness probe)
- pod终止过程

![image-20210701155758705](images/image-20210701155758705.png)

在整个生命周期中，Pod会出现5种状态(相位)，分别如下:

- 挂起(Pending)

  `apiserver`已经创建了pod资源对象，但它尚未被调度完成或者仍处于下载镜像的过程中

- 运行中(Running)

  pod已经被调度至某节点，并且所有容器都已经被`kubelet`创建完成

- 成功(Succeeded)

  pod中的所有容器都已经成功终止并且不会被重启

- 失败(Failed)

  所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非0值的退出状态

- 未知(Unknown)

  apiserver无法正常获取到pod对象的状态信息，通常由网络通信失败导致

### 5.3.1 创建和终止

**pod的创建过程**

1. 用户通过kubectl或其他api客户端提交需要创建的pod信息给apiServer
2. apiServer开始生成pod对象的信息，并将信息存入etcd，然后返回确认信息至客户端
3. apiServer开始反映etcd中的pod对象的变化，其他组件使用watch机制来跟踪检查apiServer上的变动
4. scheduler发现有新的pod对象要创建，开始为pod分配主机并将结果信息更新至apiServer
5. node节点上的kubelet发现有pod调度过来，尝试调用docker启动容器，并将结果回送至apiServer
6. apiServer将接收到的pod状态信息存入etcd

![image-20210701162645918](images/image-20210701162645918.png)



**pod的终止过程**

1. 用户向apiServer发送删除pod对象的命令
2. apiServer中的pod对象信息会随着时间的推移而更新，在宽限期内(默认30s),pod被视为dead
3. 将pod标记为terminating状态
4. kubelet在监控到pod对象转为terminating状态的同时启动pod关闭过程
5. 端点控制器监控到pod对象的关闭行为时将其从所有匹配到此端点的service资源的端点列表中移除
6. 如果当前pod对象定义了preStop钩子处理器，则在其标记为terminating后即会以同步的方式启动执行
7. pod对象中的容器进程收到停止信号
8. 宽限期结束后，若pod中还存在仍在运行的进程，那么pod对象会收到立即终止的信号
9. kubelet请求apiServer将此pod资源的宽限期设置为0从而完成删除操作，此时pod对于用户已经不可见

### 5.3.2 初始化容器

初始化容器是在pod的主容器启动之前要运行的容器，主要是做一些主容器的前置工作，它具有两大特征:

- 初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么K8s需要重启它直到成功完成
- 初始化容器必须按照定义的顺序执行，当且仅当前一个成功之后，后面一个才运行

初始化容器有很多应用场景，下面列出常见的几个：

- 提供主容器镜像中不具备的工具程序或自定义代码
- 初始化容器要先于应用容器串行启动并运行完成，因此可用于延后应用容器的启动直至其依赖的条件得到满足

![image-20210701165442553](images/image-20210701165442553.png)

![image-20210701173545414](images/image-20210701173545414.png)

![image-20210701173928207](images/image-20210701173928207.png)

![image-20210701173939260](images/image-20210701173939260.png)

### 5.3.3 钩子函数



### 5.3.4 容器探测

容器探测用于检测容器中的应用实例是否正常工作，是保障业务可用性的一种传统机制。如果经过探测，实例的状态不符合预期，那么K8s就会把该问题实例"摘除"，不承担业务流量。K8s提供了两种探针来实现容器探测，分别是:

- liveness probes

  存活性探针，用于检测应用实例当前是否处于正常运行息状态，如果不是，K8s会重启容器

- readiness probes

  就绪性探针，用于检测应用实例当前是否可以接收请求，如果不能，K8s不会转发流量

> livenessProbe决定是否重启容器，readinessProbe决定是否将请求转发给容器

上面两种探针，目前均支持三种探测方式：

- Exec命令

  在容器内执行一次命令，如果命令执行的退出码为0，则认为程序正常，否则不正常

  ![image-20210701185850088](images/image-20210701185850088.png)

- TCPSocket

  将会尝试访问一个用户容器的端口，如果能够建立这条连接，则认为程序正常，否则不正常

  ![image-20210701190331751](images/image-20210701190331751.png)

- HTTPGet

  调用容器内Web应用的URL，如果返回的状态码在200和399之间，则认为程序正常，否则不正常

  ![image-20210701190517333](images/image-20210701190517333.png)

![image-20210701190627318](images/image-20210701190627318.png)

![image-20210701191002616](images/image-20210701191002616.png)

**方式二：TCPSocket**

![image-20210701191045852](images/image-20210701191045852.png)

![image-20210701191334525](images/image-20210701191334525.png)

![image-20210701191347382](images/image-20210701191347382.png)



![image-20210701191753289](images/image-20210701191753289.png)

### 5.3.5 重启策略

pod的重启策略有3种:

- Always

  容器失效时，自动重启该容器，默认值

- OnFailure

  容器终止运行且退出码不为0时重启

- Never

  不论状态如何，都不重启该容器

重启策略适用于pod对象中的所有容器，首次需要重启的容器，将在其需要时立即进行重启，随后再次需要重启的操作将由kubelet延迟一段时间后进行，且反复的重启操作的延迟时长依次为10s、20s、40s、80s、160s和300s，300s是最大延迟时长

![image-20210701192550022](images/image-20210701192550022.png)

## 5.4 Pod调度

### 5.4.1 定向调度

定向调度，指的是利用在pod上生命`nodeName`或者`nodeSelector`，以此将Pod调度到期望的node节点上。注意，这里的调度是强制的，这就意味着即使要调度的目标Node不存在，也会向上面进行调度，只不过pod运行失败而已。

**`NodeName`**

`NodeName`用于强制约束将Pod调度到指定的Name的Node节点上。这种方式，其实是直接跳过Scheduler的调度逻辑，直接将Pod调度到指定名称的节点

![image-20210701193502652](images/image-20210701193502652.png)

**NodeSelector**

NodeSelector用于将pod调度到添加了指定标签的node节点上，它是通过K8s的label-selector机制实现的，也就是说，在pod创建之前，会由scheduler使用MatchNodeSelector调度策略进行label匹配，找出目标node，然后将pod调度到目标节点，该匹配规则是强制约束。

![image-20210701194053323](images/image-20210701194053323.png)

### 5.4.2 亲和性调度

上面介绍的两种定向调度的方式使用起来非常方便，但是也有一定的问题，就是如果没有满足条件的Node，那么Pod将不会被运行，即使在集群中还有可用Node列表也不行，这就限制了它的使用场景

基于上面的问题，K8s还提供了一种亲和性调度(Affinity)。它在NodeSelector的基础上进行了扩展，可以通过配置的形式，实现优先选择满足条件的Node进行调度，如果没有，也可以调度到不满足条件的节点上，使调度更加灵活

Affinity主要分为三类:

- nodeAffinity(node亲和性)

  以node为目标，解决pod可以调度到哪些node的问题

- podAffinity(pod亲和性)

  以pod为目标，解决pod可以和哪些已存在的pod部署在同一个拓扑域中的问题

- podAntiAffinity(pod反亲和性)

  以pod为目标，解决pod不能和哪些已存在pod部署在同一个拓扑域中的问题

> 关于亲和性(反亲和性)使用场景的说明:
>
> - 亲和性
>
>   如果两个应用频繁交互，那就有必要利用亲和性让两个应用尽可能地靠近，这样可以减少因网络通信而带来的性能损耗
>
> - 当应用采用多副本部署时，有必要采用反亲和性让各个应用实例打散分布在各个node上，这样可以提高服务的高可用性。

![image-20210701201047268](images/image-20210701201047268.png)



![image-20210701202134208](images/image-20210701202134208.png)

![image-20210701202509226](images/image-20210701202509226.png)



![image-20210701202621387](images/image-20210701202621387.png)

### 5.4.3 污点和容忍

该看44









