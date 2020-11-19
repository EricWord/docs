# Scala语法总结
## 1.Scala概述
### 1.1 Scala特点
一门以JAVA虚拟机(JVM)为运行环境并将面向对象和函数式编程的最佳特性结合在一起的静态类型编程语言  
+ 多范式(multi-paradigm)编程语言，支持面向对象和函数式编程
+ 源代码(.scala)会被编译成java字节码(.class)，然后运行于JVM之上，并可以调用现有的java类库，实现两种语言的无缝对接  
### 1.2 开发环境搭建
+ 官方安装包下载地址  
http://www.scala-lang.org/
### 1.3 Scala执行流程分析
![执行流程分析](./images/1-Scala执行流程分析.png)
### 1.4 注意事项
+ 源文件以".scala"为扩展名
+ 程序的执行入口是main()函数
+ 严格区分大小写
+ 方法由一条条语句构成，每个语句后不需要分号
+ 如果在同一行有多条语句，除了最后一条语句不需要分号，其他语句需要分号
### 1.5 输出的三种方式
+ 字符串通过+号连接(类似Java)  
```scala
val name = "xiaoming"
println("name=" + name + "age=" + age + " url=" + url)
```
+ printf用法，字符串通过%传值(类似C)
```scala
val age = 1
printf("age=%d\n",age)
```
+ 字符串通过$引用(类似PHP)
```scala
val url = "www.baidu.com"
println(s"url=$url")
```

## 2.变量
