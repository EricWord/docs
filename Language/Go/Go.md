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


