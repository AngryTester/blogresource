---
title: Maven构建反应堆及在Jenkins中的用法
date: 2017-02-09 08:15:52
tags: [Maven,加速构建,Jenkins]
---

## 问题

持续交付管道的意义之一在于快速反馈，但是随着咱们的项目越来越庞大，动辄几十万行代码，构建速度越来越慢，无法满足快速反馈的需求。还好，各种构建工具都提供了按需构建的方案。接下来结合业界使用最广泛的构建工具-Maven，介绍如何通过构建反应堆来提升构建速度。

反应堆(Reactor)是多模块的Maven项目中包含所有构建模块的抽象概念，所以，使用构建反应堆的前提是你的项目是多模块架构的。

## 用法

（前提是配置好Maven环境变量）

命令中中输入mvn -h可以看到Maven的各种用法，与构建反应堆相关的几个参数如下：

### 构建指定模块所依赖的其他模块
<!-- more -->
```
-am,--also-make
```
     
### 构建依赖指定模块的其他模块

```
-amd,--also-make-dependents	  
```   

### 指定模块

```
-pl,--projects <arg>    
```
　　

### 如果你需要指定构建module1，module2，则构建命令为：

```　　
mvn clean install -pl module1,module2
```
　　

### 如果你需要指定构建module1和依赖module1的模块，则构建命令为：

```　
mvn clean install -pl module1 -amd
```
　　

### 如果你需要指定构建module1和module1依赖的模块，则构建命令为：

```
mvn clean install -pl module1 -am
```

## Jenkins中的用法

在Jenkins中只需要选择Maven项目，然后在构建选项中选中：

```
Incremental build - only build changed modules
```
 
![](https://raw.githubusercontent.com/AngryTester/blog/master/maven.png)


即每次构建会根据本次版本更新涉及模块拼出本次构建命令，如上图所示，如果本次变更模块为p1,p2,则选中增量构建选项后的构建命令为：


```
mvn clean install -pl p1,p2 module1 -amd
```

