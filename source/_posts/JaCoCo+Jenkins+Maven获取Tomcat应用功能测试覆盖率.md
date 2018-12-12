---
title: JaCoCo+Jenkins+Maven获取Tomcat应用功能测试覆盖率
date: 2017-03-15 08:43:35
tags: [Jacoco,精准测试]
---

## 功能测试覆盖率的意义

首先需要明确，将某些数据指标与KPI联系是毫无意义的！！！

数据指标情况只能辅助咱们决策或调整策略，与功能测试覆盖率类似，大家接触较多的可能是单元测试覆盖率，覆盖率够高，不一定代码质量就好，只是开发人员的质量信心能够有一些保障，覆盖率低，则证明有一些代码的分支甚至是没有经过单元测试的，则有必要检查我们比较关注的重要分支是否被测试覆盖，如果有，我觉得即使总体覆盖率低也是可以接受的。所以，这里的指标数据只是帮助我们及时调整策略。

与单元测试覆盖率类似，功能测试覆盖率的用途首先肯定不是用作各种考核。功能测试的对象也是应用代码，只是从更偏向于终端用户的视角运行应用，所以，从分析功能测试覆盖率可预见的几个好处至少包括：

- 可能发现冗余无用代码

- 可能发现重要逻辑测试遗漏

- 可能根据覆盖率情况及时调整测试策略

<!-- more -->
- 可能根据高覆盖率为团队提供质量信心

## JaCoCo的TCP Socket Server模式

JaCoCo主要通过Instrumentation机制来收集覆盖率信息，主要依赖的是Java Agent。有关于Java Agent，详细可参考：http://www.eclemma.org/jacoco/trunk/doc/agent.html。

有三种模式输出覆盖率数据，针对Tomcat应用，我们需要用到tcpserver这种模式，即通过暴露应用的一个端口，从这个端口可以获取到覆盖率数据。

## Tomcat设置

- 首先需要下载jacocoagent.jar，下载地址：http://www.jacoco.org/jacoco/。

下载zip包之后解压获得jacocoagent.jar，将jar包放至tomcat的lib目录下。

- 编辑catalina.bat（windows下、）或catalina.sh（linux），在JAVA_OPT中加入：

```
-javaagent:%CATALINA_HOME%\lib\jacocoagent.jar=includes=*,output=tcpserver,port=2014,address=172.xx.x.xx
```

>172.xx.x.xx为你的应用访问IP，2014为你自定义暴露的端口，用于外部获取覆盖率文件。

至此，Tomcat就配置完成，在之后每次的Tomcat的启动过程中，就能够看到如下的启动日志：

```
信息: Command line argument: -javaagent:D:\apache-tomcat-7.0.57\lib\jacocoagent.jar=includes=*,output=tcpserver,port=2014,address=172.xx.x.xx
```

即通过172.xx.x.xx的2014端口，就能获取到覆盖率文件了，通常是jacoco.exec文件。

## Maven获取覆盖率文件

JaCoCo提供了多种获取覆盖率文件的方式，常见的包括Ant和Maven。这里介绍了是通过Maven命令行的方式：

```
mvn org.jacoco:jacoco-maven-plugin:0.7.9:dump -Djacoco.address=172.xx.x.xx -Djacoco.reset=false -Djacoco.port=2014 -Djacoco.append=true
```

>jacoco.address和jacoco.port为上面的Tomcat应用暴露的IP和端口，jacoco.reset和jacoco.append是指定在dump出覆盖率文件之后不做重置，在下一次获取覆盖率文件的时候直接追加。这对于多次启动应用，需要统计整体功能覆盖率的时候非常有用。

更多可用参数详情可参考：

http://www.eclemma.org/jacoco/trunk/doc/dump-mojo.html

## Jenkins分析覆盖率文件

通过Maven和Ant均可以分析覆盖率文件，甚至如果用过Sonar的同学知道，通过Sonar也可以分析JaCoCo的覆盖率文件。之所以这里用到Jenkins，是在持续交付盛行的情况下，通过Jenkins可以很方便的调度各个任务，更何况Jenkins提供了丰富的插件来支持各种构建任务。

Jenkins分析JaCoCo覆盖率文件，依赖的插件是：JaCoCo plugin（https://wiki.jenkins-ci.org/display/JENKINS/JaCoCo+Plugin）。

安装完插件之后，在Jenkins的Job设置的构建后操作部分，可以选择：Record JaCoCo coverage Report，设置好jacoco.exec，类文件以及源文件的路径，就可以分析指定覆盖率文件生成报告了。

生成的覆盖率报告通过Job页面的Coverage Trend链接可以访问：

![](https://raw.githubusercontent.com/AngryTester/blog/master/JaCoCo%2BJenkins%2BMaven%E8%8E%B7%E5%8F%96Tomcat%E5%BA%94%E7%94%A8%E5%8A%9F%E8%83%BD%E6%B5%8B%E8%AF%95%E8%A6%86%E7%9B%96%E7%8E%87/coverage.png)

可以看到整体覆盖率信息以及每个包的覆盖率信息：

![](https://raw.githubusercontent.com/AngryTester/blog/master/JaCoCo%2BJenkins%2BMaven%E8%8E%B7%E5%8F%96Tomcat%E5%BA%94%E7%94%A8%E5%8A%9F%E8%83%BD%E6%B5%8B%E8%AF%95%E8%A6%86%E7%9B%96%E7%8E%87/package.png)

当然也可以进入具体的类查看类的覆盖率信息。

## 后续

结合Docker技术，可以把对Tomcat的设置打到镜像中。












