---
title: 基于Jenkins构建持续交付管道（一）
date: 2017-02-24 08:57:27
tags: [Jenkins,Pipeline,持续交付]
---


## 前言

持续交付流程的持续指的是不断产出能够工作的成果。与敏捷思想一致，过程中通过不断对产出成果进行验证，避免与最终客户实际需求有过大偏差。

持续交付管道是持续交付的实践之一，持续交付管道的核心包括自动化、管道化和持续化。自动化的意义在于提升效率，管道化的意义在于节约流程控制所需要的人工成本，持续化的目的不断产出成果，加强过程结果监控。

以常见的Java项目为例，从代码检入到最后代码投产到生产环境，大致会包含单元测试，静态扫描，自动部署，功能测试，性能测试，安全测试、自动投产等一系列检查步骤。这里将自动部署和自动投产也列为检查步骤之一，是因为我认为自动部署和自动投产都不应该仅以执行完部署或投产的脚本就算成功，而是以部署完或投产之后环境符合预期，才能作为这两个步骤的成功条件。

持续交付管道的实现就是将各个检查步骤尽可能自动化，并串成一条线，通过管道中的自动反馈，能够快速直接地反馈当前工作成果的实际情况，比如是已经测试通过了还是连单元测试都没通过。

基于Jenkins的持续交付管道实现方式有两种，其中第一种相对好理解也好实现，但是非官方，第二种是官方推荐的管道实现方式。

## 关于Jenkins
<!-- more -->
关于Jenkins，各种资料网上已经非常齐全了，这里仅从我个人理解谈谈Jenkins。

Jenkins自身并没有构建的功能，Jenkins更像是一个调度器，通过调度其他工具，来完成构建任务。

Jenkins调度其他工具主要是依赖自身的插件系统，可以理解为Jenkins系统仅是为各种工具提供了一个图形界面，用于配置基于各个工具的构建任务。所以业界对Jenkins的评价中有一条比较负面的就是说在Jenkins的使用过程中有可能陷入“插件地狱”，主要因为Jenkins的各种插件之间的依赖关系非常复杂，使用A插件可能需要B插件已经安装，并且可能要求B插件的版本必须大于指定版本，必须依赖Jenkins的在线安装，否则手工安装插件的工作量特别大。


### Jenkins的启动

Jenkins的启动非常简单，从官方下载的Jenkins镜像是一个War包，所以可以用业界常见的容器比如Tomcat启动，同时,Jenkins内置了一个Jetty，所以也可以不依赖容器启动。

>利用内置Jetty容器启动
>
>`java -jar jenkins.war`

所以，只需要有JDK环境，就可以启动一个Jenkins服务。

>同时可以在启动时候指定一些参数，例如默认的Jenkins启动是在8080端口，如果想另外指定启动端口，可以通过命令参数httpPort。
>
>`java -jar jenkins.war --httpPort=8081`

则Jenkins就启动在8081端口了，本地可通过localhost:8081就可以访问到Jenkins服务了。

### Jenkins的第一次配置

Jenkins的第一次启动需要有一些默认配置，例如管理员账号，代理等。

下图是访问Jenkins第一次启动的页面：

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%80%EF%BC%89/2.png)

需要输入默认管理员密码，这密码从哪里获取呢？

>可以从提示的路径，如上图所示，C:\Users\XX\.jenkins\secrets\initialAdminPassword，XX为你当前用户名。

>也可以从启动日志中获取，如下图所示：

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%80%EF%BC%89/3.png)

输入密码之后下一步，到达代理设置页面，如果你的网络有代理，可以点击代理设置配置代理，否则也可以直接点击直接跳过插件安装。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%80%EF%BC%89/4.png)

在配置完代理之后下一步，会进入插件安装选择页面，分别有两个选项，第一个选项是安装推荐插件，第二个选项是自己选择安装，建议选择第二个选项。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%80%EF%BC%89/6.png)

这里介绍几个常用的插件，如图：

- Subversion Plug-in：SVN插件，需要有这个插件才能从SVN中检出代码。

- Pipeline:官方推荐的管道插件，也是咱们的持续交付管道要用到的。

- Maven Integration plugin：MAVEN插件，需要有这个插件才能新建MAVEN类型的Job。

- Git plugin：Git插件，需要有这个插件才能从Git中检出代码。


可以按照个人需求选择安装插件，待安装完成后，进入最后一步设置，设置管理员用户密码。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%80%EF%BC%89/8.png)

设置完成之后正式进入Jenkins首页。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%B8%80%EF%BC%89/9.png)

























