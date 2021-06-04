[TOC]

# 1. Spring 概念

## 1.1 Spring框架概述

1. Spring是轻量级的开源JavaEE框架

2. Spring可以解决企业应用开发的复杂性

3. Spring有两个核心部分：IOC和AOP

   IOC:控制反转，把创建对象的过程交给Spring进行管理

   AOP:面向切面，不修改源代码进行功能增强

4. Spring特点

   - 方便解耦，简化开发
   - AOP编程支持
   - 方便程序测试
   - 方便和其他框架进行整合
   - 方便进行事务操作
   - 降低API开发难度

![image-20210604145747159](images/image-20210604145747159.png)



## 1.2 入门案例

### 1.2.1 下载Spring 5

1. 选择最新稳定版本

![image-20210604125434042](images/image-20210604125434042.png)

2. 下载地址

   https://repo.spring.io/release/org/springframework/spring/

3. 创建工程

4. 导入相应的jar包

5. 创建一个普通的类，在这个类创建普通方法

6. 创建Spring配置文件，在配置文件中配置创建的对象

   Spring配置文件使用xml格式

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!--配置User对象的创建-->
       <bean id="user" class="net.codeshow.spring5.User"></bean>
   </beans>
   ```

7. 测试代码编写

   ```java
   @Test
       public void testAdd() {
           //1. 加载spring配置文件
           ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
           //2.获取配置创建的对象
           User user = context.getBean("user", User.class);
           System.out.println(user);
           user.add();
   
       }
   ```



# 2. IOC容器

## 2.1 IOC底层原理

### 2.1.1 什么是IOC?

1. 控制反转，把对象创建和对象之间的调用过程，交给Spring进行管理
2. 使用IOC目的:为了耦合降低
3. 入门案例就是IOC实现

### 2.1.2 IOC底层原理

1. xml解析、工厂模式、反射

- 原始方式

  ![image-20210604152915570](images/image-20210604152915570.png)

  

- 工厂模式

  ![image-20210604153211937](images/image-20210604153211937.png)

- IOC方式

  ![image-20210604153736019](images/image-20210604153736019.png)

  

## 2.2 IOC接口(BeanFactory)

1. IOC思想基于IOC容器完成，IOC容器底层就是对象工厂

2. Spring提供了IOC容器实现的两种方式:(两个接口)

   - BeanFactory

     IOC容器基本实现，是Spring内部的使用接口，不提供给开发人员使用

     **加载配置文件的时候不会创建对象，使用对象的时候才创建**

   - ApplicationContext

     BeanFactory接口的子接口，提供更多更强大的功能，一般由开发人员进行使用

     加载配置文件的时候就会创建对象

3. ApplicationContext接口有实现类

   ![image-20210604155414676](images/image-20210604155414676.png)

   两个实现类的区别：

   `FileSystemXmlApplicationContext`在创建对象的时候需要传入配置文件的绝对路径

   `ClassPathXmlApplicationContext`在创建对象的时候需要传入配置文件在`src`目录下的路径

   

## 2.3 IOC操作Bean管理(基于XML)

### 2.3.1 什么是Bean管理？

Bean管理指的是两个操作：

1. 由Spring创建对象
2. 由Spring注入属性

Bean管理操作有两种实现方式：

- 基于xml配置文件方式实现
- 基于注解方式实现



### 2.3.2 基于xml方式创建对象

```xml
<bean id="user" class="net.codeshow.spring5.User"></bean>
```

1. 在Spring配置文件中，使用bean标签，标签里面添加对应属性，就可以实现对象创建

2. 在bean标签有很多属性，常用的属性有以下几种:

   - id属性

     唯一标识

   - class属性

     类的全路径

   - name属性

     作用基本与id属性相同，两者的区别是name属性中可以有特殊符号，而id属性中不能有特殊符号

3. 创建对象的时候，默认也是执行无参数构造方法完成对象的创建



### 2.3.3 基于xml方式注入属性

1. DI:依赖注入，就是注入属性，是IOC的一种具体实现，需要在完成对象创建的基础上执行

   - 第一种注入方式:使用set方法进行注入

   - 第二种注入方式:通过有参构造器注入

     (1). 创建类，定义属性，创建属性对应有参构造方法

     (2). 在Spring配置文件中进行配置

     ```xml
       <!--有参构造注入属性-->
         <bean id="orders" class="net.codeshow.spring5.Orders">
             <constructor-arg name="name" value="电脑"></constructor-arg>
             <constructor-arg name="address" value="China"></constructor-arg>
             <!--        <constructor-arg index="0" value=""></constructor-arg>-->
     
         </bean>
     ```

   - p名称空间注入

     (1). 使用p名称空间注入，可以简化基于xml配置方式

     ![image-20210604175016607](images/image-20210604175016607.png)

2. xml注入其他类型属性

   - 字面量

     (1). null值

     (2).属性值包含特殊符号

3. 注入属性-外部bean

   (1).创建两个类service类和dao类

   (2).在service调用dao里面的方法

   (3).在spring配置文件件中进行配置

4. 注入属性-内部bean和级联赋值

   (1). 一对多关系:部门和员工

   一个部门有多个员工，一个员工属于一个部门，部门是一，员工是多

   (2). 在实体类之间表示一对多关系

   ```xml
    <!--内部bean-->
       <bean id="emp" class="net.codeshow.spring5.bean.Emp">
           <!--设置两个普通属性-->
           <property name="ename" value="lucy"></property>
           <property name="gender" value="女"></property>
           <!--设置对象类型属性-->
           <property name="dept">
               <bean id="dept" class="net.codeshow.spring5.bean.Dept">
                   <property name="dname" value="安保部"></property>
               </bean>
   
           </property>
       </bean>
   ```

   该看13



## 2.4 IOC操作Bean管理(基于注解)













# 3. AOP







# 4. JDBCTemplate







# 5. 事务管理





# 6. Spring5新特性



