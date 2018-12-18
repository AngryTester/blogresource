---
title: 基于Jenkins构建持续交付管道（三）
date: 2017-02-28 08:57:41
tags: [Jenkins,Pipeline,持续交付]
---

## 前言

上一章中已经介绍了如何基于Jenkins配置一个独立的Job，即完成了交付流程中单个节点的实现，持续交付管道需要将交付流程中的各个节点串起来，再加上适当的反馈。本章将介绍如何将各个独立Job进行串联。

## 各个交付节点的实现

结合上一章的内容，交付流程中的各个节点其实就对应到一个一个的Job。以典型的Java项目为例，如果使用Maven进行构建，可能涉及的节点可能有：编译打包、单元测试、静态扫描、测试部署、功能测试等。分别对应的Maven命令如下：

- 编译打包(跳过单元测试)：mvn clean install -Dmaven.test.skip=true

- 单元测试：mvn clean test

- 静态扫描：mvn sonar:sonar

<!-- more -->
### 测试环境部署

测试环境部署与其他交付节点有所不同，因此涉及与测试服务器的交互，所以测试环境部署任务需要分成两步。第一步是打包，第二步才是部署。但是庆幸的是Jenkins提供了相关插件，让你可以在一个Job中完成整个部署操作。

打包部分可以参考上面的内容，即mvn clean install即可完成。但是之后对测试服务器的操作得依赖另一个插件：publish over ssh。

在插件安装完成之后，需要在Jenkins中对测试服务器进行配置，然后在Job中才可以选择使用。具体操作是：`系统管理>系统设置>Publish over SSH`。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%89%EF%BC%89/1.png)

点击“增加”按钮，参考以下内容输入：

- Name：定义服务器名称，可自定义

- Hostname:服务器IP

- Username:登录用户名

- Remote Directory：默认SSH进入的路径

点击高级，勾选“Use password authentication, or use a different key”，

-Passphrase / Password：登录密码

点击Test Configuration可以测试配置，如果返回“连接成功”，则代表配置正确。

再进入Job设置，在Post Step下选择“Send file or execute commands over SSH”。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%89%EF%BC%89/2.png)

参考以下内容输入：

- Name：下拉选择系统设置中配置的服务器

- Source files：需要传输的文件，相对路径。如工作空间下有一个文件夹aa，需要传输aa下面的bb.war，则这里需要输入aa/bb.war

- Remove prefix：去掉上一条内容前缀，即如果这里输入aa/，这内容变为bb.war。

- Remote directory：文件发送的远程路径，与系统设置中的远程路径组合成完整路径。例如这里输入opt/app，加上系统设置中的/，则完整远程路径为/opt/app

- Exec command：需要执行的命令。以上已经完成了文件的传输，一般是war包，即需要部署的文件已经发送至测试服务器了。还需要在测试服务器上执行一段shell，例如停止应用服务器，传输war包等。就可以在这里配置。

通过以上配置，就可以完成测试环境的部署了。

## 各个交付节点的串联

Jenkins的交付节点串联依赖另一个插件-build pipeline plugin。在安装完这个插件之后，进入Job设置，在“增加构建后操作”下选择“Trigger parameterized build on other projects”：

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%89%EF%BC%89/3.png)

- Projects to build:即当前job构建结束之后要运行的Job名称。

- Trigger when build is：当前Job构建状态为XX时才触发。

- Trigger build without parameters：不需要指定参数构建，如果不传参数，这个必须勾选上，否则串联会有问题。

- Add Parameters：可以指定参数。这里重点需要说明的是SVN Revision参数，如果你需要将前一个Job的SVN版本号传给下一个Job，就可以把这个参数加上。重点解决的是前后如果对同一份代码进行编译和单元测试，可能造成前后版本不一致（编译过程中有人提交）。

这样串联之后，在当前Job执行完之后就会自动触发下一个Job执行了。

## 创建管道视图

上面的步骤实际已经将各个Job串联成一条管道了，但是缺乏一个统一视图。可以通过添加build pipeline视图来创建一个管道视图。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%89%EF%BC%89/4.png)

重点需要配置的是初始Job。其他配置可以根据需要选择。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%89%EF%BC%89/5.png)

配置之后，就能看到一个管道的视图了，点击Run，就会自动运行整条管道了。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%89%EF%BC%89/6.png)

## 后记

以上是管道的主要实现，在实际应用过程中，可能还有一些细节需要优化，例如反馈机制（邮件通知），触发规则等等。可根据各个项目的不同需求自行定制。






