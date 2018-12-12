---
title: 基于Jenkins构建持续交付管道（四）
date: 2017-03-01 08:24:44
tags: [Jenkins,Pipeline,持续交付]
---

## 前言

上一章介绍了如何基于build pipeline plugin插件搭建Jenkins的持续交付管道，但是实际上是通过另外新建一个视图来将原本没有关系的Job串联在一起，构成“管道”视图。这种方式的好处在于实现简单，不需要变更使用习惯。
　　

但是随着持续交付管道应用的深入，渐渐会发现这种方式的种种弊端：

- 与“管道流”的初衷违背。由于各个交付节点实际上是各自分离的独立Job，所以每一个Job会在都是在自己的独立工作空间进行构建任务。这与“管道流”的初衷-上一节点的合格交付物流入下一节点-是违背的，因为各个Job的交付物其实是没有直接关联的；

- Job维护工作量大。由于每一个交付管道的节点数众多，意味着每一条交付管道需要你维护多个Job，然后再将各个Job串联。因此，你可能需要花费大量时间来维护多个Job配置；

- 交付反馈流程可能存在冗余。由于各个Job都有自己独立的工作空间，各个工作空间是无法直接交互的。为了尽早反馈，我们希望将反馈节点尽量分割开，比如单元测试和静态扫描我们会分离成两个Job，但是静态扫描实际会依赖单元测试的产物（主要是覆盖率jacoco文件），所以导致单元测试可能会在单元测试和静态扫描环节重复执行，同样的情况还存在于编译打包和部署环节。

　　
<!-- more -->
因此，我们适时考虑Jenkins官方推荐的Pipeline As Code。

## Pipeline

Jenkins的Pipeline As Code基于pipeline插件。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E5%9B%9B%EF%BC%89/1.png)

在安装完插件之后，在新建Job的时候就可以选择pipeline类型了。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E5%9B%9B%EF%BC%89/2.png)

新建一个pipeline类型Job，看一下Job的配置，会发现和之前介绍的常规Job有所区别。除了触发策略，另一部分就是pipeline的脚本，从这里也能看出来相比传统Job，配置会更简单。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E5%9B%9B%EF%BC%89/3.png)

pipeline的核心就是管道脚本。那么脚本如何设计呢？


Jenkins提供了几个pipeline脚本的例子，可以选择GitHub+Maven的例子。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E5%9B%9B%EF%BC%89/4.png)


下面以上面的例子讲解一下脚本的写法。

- 第一行node：代表后面跟的大括号里的构建脚本将在哪个节点上运行，如果是空，则默认在master节点，否则可以写做node(“slave”)，这里的slave是你的节点名称。

- def  mvnHome：定义一个变量mvnHome，因为Jenkins的Pipeline脚本是基于Groovy语言的，所以里面可以直接使用Groovy的一些语言特性，包括这里的定义变量以及后面的逻辑语句。

- stage('Preparation')：定义一个交付节点，代表后面跟的大括号里的脚本统一定义为一个交付节点，节点名称为“Preparation”，有点类似之前方案中的持续交付管道的Job名称。

- git 'https://github.com/jglick/simple-maven-project-with-tests.git':获取指定git版本库的代码。这种格式是如何定义的呢？可以通过下方的Pipeline Syntax链接跳转到新页面进行调试。

例如如果希望获取某个git库的代码，选择git后填写好相应信息，点击Generate Pipeline Script，就会生成对应代码：

![](https://raw.githubusercontent.com/AngryTester/blog/master/git.jpg)


上面的例子不用输入git用户名密码，是因为配置了git的SSH Key。可参考：http://blog.csdn.net/fenglailea/article/details/39317513。


- mvnHome = tool 'M3'：变量赋值。M3是哪里配置的呢？在装好Maven插件之后，系统管理里增加Maven时的命令。如下图：

![](https://raw.githubusercontent.com/AngryTester/blog/master/maven.jpg) 

“M3”就应该替换成“apache-maven-3.3.9”。

- stage('Build')：定义另一个交付节点，代表后面跟的大括号里的脚本统一定义为一个交付节点，节点名称为“Build”。

- if (isUnix())：判断服务器是否为Unix系列，与后面的大括号共同形成逻辑判断语句。

- sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"：如果服务器为Unix系列，则执行这段bash，${mvnHome}为前面定义的变量mvnHome的值。

- bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)：如果服务器不为Unix（即为Windows系列），则执行这段bat。

- stage('Results')：定义另一个交付节点，代表后面跟的大括号里的脚本统一定义为一个交付节点，节点名称为“'Results'”。

- junit '**/target/surefire-reports/TEST-*.xml'：发布junit测试结果，后面为junit执行产生的xml文件地址。

- archive 'target/*.jar'：存档文件，后面内容为存档文件路径。


除了示例脚本，Jenkins还提供了编写脚本的帮助工具，通过脚本配置框下面的Pipeline Syntax或者Job首页的Pipeline Syntax均可进入。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E5%9B%9B%EF%BC%89/5.png)

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E5%9B%9B%EF%BC%89/6.png)

例如，我需要从一个Git库中检出代码，可以选择下拉选择Git，然后输入Git库地址，点击生成脚本按钮，就会在下面生成对应的脚本，这段脚本就可以加到你们的pipeline中了。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E5%9B%9B%EF%BC%89/7.png)



除了在线编写脚本，也可以通过版本库直接读取管道脚本：

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E5%9B%9B%EF%BC%89/8.png)


## 后记

从上面的示例脚本可以看出来，Build节点的产物如Jar包在最后的Result节点就可以直接存档了。这与管道流的概念不谋而合，下一个节点的产物应该来自于上一个节点。

- 以上示例脚本相当于完成了3个Job，所以维护工作相比之前的方式大大减少了。

- 由于整个管道都是在同一个工作空间，所以各个交付节点的产物都可以复用。之前存在的单元测试需要执行两遍的冗余就不存在了。

- 减少冗余之后管道执行时间自然有所缩短。