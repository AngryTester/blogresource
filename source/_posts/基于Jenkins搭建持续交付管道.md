---
title: 基于Jenkins搭建持续交付管道
date: 2017-02-09 08:42:21
tags: [Jenkins,持续交付]
---


## Jenkins2.0

持续交付管道是支撑持续交付流程的基础设施，除了Jenkins，业界还有其他的一些持续集成工具/平台如ThoughtWorks的Go平台以及Travis CI，主要用途就是为了打造交付流水线，所以本身就是支持管道化构建的。在这一点上，Jenkins相对会落后一些，但是在Jenkins2.0版本，及时推出了对管道化构建的支持，即Build Pipeline。通过Jenkins的管道化支持，你可以将你的交付过程中的各个环节串联起来，组成属于你们项目组的交付流程。如下图所示：

![](https://raw.githubusercontent.com/AngryTester/blog/master/pipeline.png)

## 建设过程

Jenkins的安装和启动就不赘述了，需要使用到的插件是Build Pipeline View，可能存在依赖插件，建议将Jenkins做成一个镜像，便于其他团队复用，避免大量重复配置。

### 逐个建立构建任务

<!-- more -->
将你的交付过程中的各个任务建立成一个一个Job，例如构建、单元测试、Sonar扫描、测试环境部署、自动化冒烟测试等等。为了更快速更准确地定位原因，建议将各种检查尽量分割成独立的Job，例如单元测试和Sonar扫描，本来是可以归成一个Job执行，但是如果这个Job失败，你无法快速准确定位到是单元测试失败还是Sonar扫描失败。

### 用build pipeline插件将各个Job连接

![](https://raw.githubusercontent.com/AngryTester/blog/master/pipeline1.png)

点击图中的加号按钮，新增视图，选择Build Pipeline View

![](https://raw.githubusercontent.com/AngryTester/blog/master/pipeline2.png)

通过Select Initial Job选择初始Job，点击保存，一个持续交付管道就建好了。

![](https://raw.githubusercontent.com/AngryTester/blog/master/pipeline3.png)


如果你的起始Job还从来没有构建过，你会发现持续交付管道暂时是空的，只要你构建一次，就能看到持续交付管道中已经有了初始Job。

![](https://raw.githubusercontent.com/AngryTester/blog/master/pipeline4.png)

初始Job后续的交付流程则通过对Job的依赖关系的管理来实现。

例如初始Job为ci-build，后续job为ci-unittest，则配置ci-build，选择构建后操作-Trigger parameterized build on other projects，Projects to build输入ci-unittest，勾选Trigger build without parameters，保存后这个job就加入到持续交付管道中了。

![](https://raw.githubusercontent.com/AngryTester/blog/master/pipeline5.png)

现在你就可以直接通过点击持续交付管道中的Run来触发整个管道了。
　　

### 注意

为了保证前序job和后续job是在同一个svn版本号上进行的构建，在对前序job配置时，应该通过传递svn版本号确保前后svn版本号一致。

![](https://raw.githubusercontent.com/AngryTester/blog/master/pipeline6.png)

## 总结

持续交付的主要目的是加速交付速度，提升交付质量，所以只要是基于这个目的的流程都是提倡的方向，持续交付管道只是持续交付流程的形式之一。持续交付管道可以快速反馈项目的质量，指导项目组及时发现项目中的问题，根据缺陷修复成本理论，肯定是可以降低项目总体成本的。