[TOC]

# Go相关技术总结

## 1.本地编译生成目标系统(Linux)bin文件
下面的两种方式都可以实现
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build  
GOOS=linux GOARCH=amd64 go build
## 2.开发环境搭建
### 2.1 macOS
+ 安装包下载  
https://studygolang.com/dl下载安装包并安装  
+ 环境变量配置  
Vim ~/.bash_profile 添加以下内容  
export GOPATH=/Users/cuiguangsong/my_files/workspace/go  
export GOBIN=$GPPATH/bin  
export PATH=$PATH:$GOBIN  
+ 配置文件生效  
source ~/.bash_profile  
+ 确认配置文件生效  
执行 go env 查看配置是否生效

## 3.struct参数校验框架
github地址  
https://github.com/go-playground/validator  
该框架可以对前端传过来的结构体中的参数进行校验，自己不要在手动写那些校验的函数

## 4.对官方database/sql 库的分析
http://go-database-sql.org/

## 5.限流框架
https://github.com/juju/ratelimit

## 6.proto生成go文件
protoc --go_out=plugins=grpc:. hello.proto

## 7.常用方法封装
### 7.1 获取当天的零点
```go
package main
import (

"errors"
"fmt"
"time"
)
func GetTodayTimeZero(t time.Time) (string, error) {
	timeStr := t.Format("2006-01-02")
	res, err := time.ParseInLocation("2006-01-02 15:04:05", timeStr+" 00:00:00", time.Local)
	if err != nil {
		errMsg := fmt.Sprintf("Failed to call ParseInLocation,reason=[%v]", err)
		fmt.Println(errMsg)
		return "", errors.New(errMsg)

	}
	return res.Format("2006-01-02 15:04:05"), nil
}
func main() {
 GetTodayTimeZero(time.Now())
}
```

## 8. vendor
### 8.1 初始化vendor目录
govendor init  
govendor add +e

### 8.2 更新vendor
govendor update

## 9. MPG线程模型概述

Go语言中的MPG线程模型对两级线程模型进行了一定程度的改进，使它能够更加灵活地进行线程之间的调度。由三个模块组成，如下图所示

![img](E:\Projects\docs\Language\Go\images\1-MPG线程模型.png)

MPG的3个主要模块以及功能

+ Machine模块:

  一个Machine对应一个内核线程，相当于内核线程在Go语言进程中的映射

+ Processor模块:

  一个Processor表示执行Go代码片段所必须的上下文环境，可以理解为用户代码逻辑的处理器

+ Goroutine模块:

  是对Go语言中代码片段的封装，其实是一种轻量级的用户线程

从上图可以看出每一个M都会与一个内核线程绑定，在运行时一个M同时只能绑定一个P,而P和G的关系则是一对多。在运行过程中，M和内核线程之间对应关系不会变化，在M的生命周期内，它只会与一个内核线程绑定，而M和P以及P和G之间的关系都是动态可变的。

在实际的运行过程中，M和P的组合才能够为G提供有效的运行环境，而多个可执行G将会顺序排成一个挂在某个P上面，等待调度和执行，如下图所示

![img](E:\Projects\docs\Language\Go\images\2-实际运行示意图.png)

上图中，M和P共同构成了一个基本的运行环境，此时G0中的代码片段处于正在运行的状态，而右边的G队列处于待运行的状态。

当没有足够的M来和P组合为G提供运行环境时，Go语言会创建新的M。在很多时候M的数量可能会比P要多。在单个Go语言进程中，P的最大数量决定了程序的并发规模，且P的最大数量是由程序决定的。可以通过修改环境变量GOMAXPROCS和调用函数runtime.GOMAXPROCS来设定P的最大值

M和P会适时的组合和断开，以保证待执行G队列能够得到及时运行。比如上图中的G0此时因为网络I/O而阻塞了M，那么P就会携带剩余的G投入到其他M中。这个新的M(下图中的M1)可能是新创建的，也可能是从调度器空闲M列表中获取的，这取决于此时的调度器空闲M列表中是否存在M,这样的机制设计也是为了避免M过多创建。

![img](E:\Projects\docs\Language\Go\images\3-IO阻塞M后运行示意图.png)

当M对应的内核线程被唤醒时，M将会尝试为G0捕获一个P上下文，可能是从调度器的空闲P列表中获取，如果获取不成功，M会把G0放入到调度器的可执行G队列中，等待其他P的查找。为了保证G的均衡执行，非空闲的P运行完自身的可执行G队列后，会周期性从调度器的可执行G队列中获取待执行的G，甚至从其他的P的可执行G队列中掠夺G                                                                                                                                                                                                                             


































