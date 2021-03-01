[TOC]

# 1.JVM与Java体系结构

## 1.1 JVM的整体结构

![image-20210222212403204](./images/1.png)

![image-20210222212441814](./images/2.png)

+ HotSpot VM

## 1.2 JVM的架构模型

![image-20210226162611330](./images/3.png)

![image-20210226163530635](./images/4.png)

## 1.3 JVM的生命周期

![image-20210226182330714](./images/5.png)

![image-20210226182405362](./images/6.png)

## 1.4 JVM的发展历程(虚拟机的种类)

![image-20210226212144243](./images/7.png)

![image-20210227112908015](./images/8.png)



![image-20210227113207916](./images/9.png)

![image-20210227113624610](./images/10.png)

![image-20210227113924634](./images/11.png)

![image-20210227114401475](./images/12.png)

![image-20210227114622329](./images/13.png)

![image-20210227114658390](./images/14.png)

![image-20210227114917325](./images/15.png)

![image-20210227115139853](./images/16.png)

![image-20210227115338009](./images/17.png)

![image-20210227115616597](./images/18.png)

![image-20210227115924293](./images/19.png)

![image-20210227120041482](./images/20.png)



# 2. 类加载子系统

## 2.1 内存结构概述



## 2.2 类加载器与类的加载过程

### 2.2.1 类加载子系统作用

![image-20210227121223049](./images/21.png)

### 2.2.2 类加载器ClassLoader角色

![image-20210227121610869](./images/22.png)

### 2.2.3 类的加载过程

![image-20210227121801536](./images/23.png)

![image-20210227121831397](./images/24.png)

#### 2.2.3.1 类的加载过程一：Loading

![image-20210227122154430](./images/25.png)

![image-20210227122334718](./images/26.png)

#### 2.2.3.2 类的加载过程二：Linking

![image-20210227122554763](./images/27.png)

#### 2.2.3.3 类的加载过程三：Initialization

![image-20210227124337016](./images/28.png)

 ## 2.3 类加载器分类

![image-20210227145556843](./images/29.png)

![image-20210227145640925](./images/30.png)

![image-20210227155655721](./images/31.png)

![image-20210227155947108](./images/32.png)

![image-20210227160141656](./images/33.png)

![image-20210227160632240](./images/34.png)

![image-20210227160710629](./images/35.png)

## 2.4ClassLoader的使用说明

![image-20210227161258007](./images/36.png)

![image-20210227161448076](./images/37.png)

![image-20210227161523867](./images/38.png)

## 2.5 双亲委派机制

![image-20210227161830063](./images/39.png)

![image-20210227161911890](./images/40.png)

![image-20210227162751544](./images/41.png)

![image-20210227162916157](./images/42.png)

![image-20210227164713401](./images/43.png)

## 2.6 其他

![image-20210227164916234](./images/44.png)

![image-20210227165133926](./images/45.png)

![image-20210227165846715](./images/46.png)

# 3. 运行时数据区及线程

![image-20210227171147209](./images/47.png)

## 3.1 概述

![image-20210227171317795](./images/48.png)

![image-20210227171507767](./images/49.png)

![image-20210227171757671](./images/51.png)

![image-20210227171849736](./images/52.png)

![image-20210227172026471](./images/53.png)

![image-20210227172428213](./images/54.png)

## 3.2 线程

![image-20210227172642371](./images/55.png)

![image-20210227172929915](./images/56.png)

# 4. 程序计数器

## 4.1 PC Register介绍

![image-20210227175734174](./images/57.png)

 ![image-20210227180005517](./images/58.png)

![image-20210227180120739](./images/59.png)

![image-20210227180444696](./images/60.png)

## 4.2 举例说明

![image-20210227180715348](./images/61.png)

![image-20210227181433793](./images/62.png)

## 4.3 两个常见问题

![image-20210227181518487](./images/63.png)

注意上图中两个问题是同一个问题，只是问法不一样

![image-20210227181722280](./images/64.png)

![image-20210227182126803](/Users/cuiguangsong/go/src/docs/Language/Java/JVM/images/65.png)

# 5. 虚拟机栈

## 5.1 虚拟机栈概述

![image-20210227184710561](./images/66.png)

![image-20210227184815339](./images/67.png)

![image-20210227184927624](./images/68.png)

![image-20210227185941838](./images/70.png)

![image-20210227190013834](./images/71.png)

![image-20210227190320102](./images/72.png)

![image-20210227190413968](./images/73.png)

![image-20210227190728651](./images/74.png)

## 5.2 栈的存储单位

![image-20210227191300808](./images/75.png)

![image-20210227191533792](./images/76.png)

![image-20210227191657739](./images/77.png)

![image-20210227192413799](./images/78.png)

![image-20210227194659858](./images/79.png)

![image-20210227195522170](./images/80.png)

## 5.3 局部变量表

![image-20210227195725145](./images/81.png)

![image-20210227201323203](./images/82.png)

![image-20210227202706029](./images/83.png)

![image-20210227202959649](./images/84.png)

![image-20210227203620937](./images/85.png)

![image-20210227203839954](./images/86.png)

![image-20210227204326209](./images/87.png)

## 5.4 操作数栈

![image-20210227204654303](./images/88.png)

![image-20210227204858884](./images/89.png)

![image-20210227205052581](./images/90.png)

![image-20210227205351201](./images/91.png)

## 5.5 代码追踪

![image-20210227205553585](./images/92.png)

![image-20210227205752296](./images/93.png)

## 5.6 栈顶缓存技术(Top-of-Stack Cashing)

![image-20210227210940550](./images/94.png)

## 5.7 动态链接

![image-20210227211510690](./images/95.png)

![image-20210227212653829](./images/96.png)

![image-20210227212957126](./images/97.png)

该看56

## 5.8 方法的调用：解析与委派



## 5.9 方法返回地址



## 5.10 一些附加信息



## 5.11 栈的相关面试题



