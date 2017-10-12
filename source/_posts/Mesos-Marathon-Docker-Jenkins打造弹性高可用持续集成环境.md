---
title: Mesos+Marathon+Docker+Jenkins打造弹性高可用持续集成环境
date: 2017-10-11 11:50:15
tags: [Jenkins,Mesos,Marathon,Docker,Jenkins]
---

## 前言

去年作为公司的持续集成环境的主要维护人员,将基于Jenkins的持续集成环境搭建起来,并维护了近300个Job,随着持续交付陆续在公司落地,可预见的是构建任务会越来越多,对持续集成环境的要求也越来越高.目前持续集成环境采取的方案是单master节点+多slave节点的传统方式,随着任务量越来越大,暴露出几个明显的问题:

- 1.构建节点数不够：带来的直接影响是如果多任务并发，将会有大量任务由于没有可用构建节点而需要长时间的等待。这对于持续交付要求的快速反馈是非常不利的。
- 2.分发服务器资源利用率不均：这一点与Jenkins自身的分发机制有关，单任务只能指定构建节点，即必须指定服务器。带来的影响是如果构建策略命中的任务只集中在一台服务器构建，那么可能指定的这台服务器基本满载，而另外的分发服务器可能是空闲的。
- 3.环境可用性差：目前环境的维护是纯靠手工的，特别是在服务器出现问题之后，需要人工及时重启，如果维护人员没有及时知晓，可能环境长时间不可用，这对于生产工作也是极为不利的。

基于以上存在的问题，我们引入了Mesos+Marathon+Docker+Jenkins的技术方案来搭建一个弹性高可用的持续集成环境。

## 方案介绍

首先，我们先来了解一下这个方案中涉及到的新工具。

- 1.Mesos

Mesos是Apache下的开源分布式资源管理框架，它被称为是分布式系统的内核。以下是官网给出的架构图。Mesos包括master和agent，其中master负责调度，agent负责执行。agent上任务的执行实际是通过executor来完成的，并且任务不会直接执行在agent的操作系统上，而是运行在一个相对隔离的沙箱环境，这样一方面是保证了安全性，另外一方面也确保了在同一个agent上可能运行多个任务而不会互相影响。这一点相对Jenkins目前的分发策略完全是个颠覆，有了这个机制，一台分发服务器就可以当做多个构建节点来使用了。

![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/mesos.png)



-2.Marathon

马拉松重点解决的问题是服务不可用时的处理。从上面Mesos的架构图也可以看出，Mesos本身是不具备调度任务的功能的，必须得借助Framework来完全，例如上图中的Hadoop和MPI就是属于这类框架。Marathon也是属于调度任务的框架，并且有一个很好的特点，在节点任务执行时候时，可以结合Mesos自动选择其他节点重新执行任务，这样就保证了任务的长时间运行，这对于Jenkins所要求的长时可用非常重要。以下是官网给出的Marathon和Mesos结合使用的架构图。

![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/marathon.png)

- 3.Docker

Docker容器的隔离性、一致性用于快速Jenkins构建环境将会是绝配。这个方案中，我们将使用到两个镜像，一个镜像是Jenkins服务，另一个是Jenkins的构建节点，通过Docker镜像可以很方便的复制出多个一致的构建环境，并且各个环境运行在一台实体服务器上而不会互相影响。

- 4.Jenkins

公司去年的持续集成环境主要就是基于Jenkins(虽然现在已转移到gitlab-ci)。在这个方案中，我们将用到Jenkins的mesos插件，这个插件实际上实现了一个framework，通过这个framework结合Mesos可以帮助我们在单台实体服务器上申请多个构建环境运行构建任务，且各个构建环境之间不会互相影响，后续会详细介绍这个插件的配置。 

以上介绍完了这个方案中涉及到的工具，下面的具体的实现方案。

### 第一步，准备三台机器.

考虑到要使用docker，我这里的准备的是三台CentOS7。

机器1：我们会在这台机器上运行Mesos Master，Marathon和Zookeeper。 
机器2、机器3：这两台机器作为Mesos Slave，运行Jenkins环境以及分发的构建环
境。

先配置三台机器的hosts： 

```
172.26.1.1 master
172.26.1.2 slave1
172.26.1.3 slave2
```

以上IP为虚拟，做演示用，实际配置请根据你的机器的实际IP。

### 第二步，安装相关工具并对服务器做相关配置。

根据上面的安排，我们将在机器1上安装Mesos、Marathon、Zookeeper。

- 1.安装mesos源。

```
#配置Mesos源
rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
```

- 2.安装工具。

```
yum install mesos marathon mesosphere-zookeeper
```

- 3.配置Zookeeper。

```
cd /etc/zookeeper/conf
grep -v "#" zoo_sample.cfg >zoo.cfg
vi zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/var/lib/zookeeper
clientPort=2181
server.1=172.26.1.1:2888:3888
echo 1>myid
vi /etc/mesos/zk
zk://172.26.1.1:2181/mesos
```

- 4.配置Marathon。

```
cd /etc/marathon/conf
vi hostname
172.26.1.1
vi master
zk://172.26.1.1:2181/mesos
vi zk
zk://172.26.1.1:2181/marathon
```

- 5.配置Mesos Master。

```
cd /etc/mesos-master
vi cluster
#输入任意你定义的集群的名称，如：vms
vi hostname
#输入master的服务器IP，如：172.26.1.1
vi ip
#输入master的ip，如：172.26.1.1
```

- 6.启动Master上的相关服务。

```
systemctl start zookeeper
systemctl start mesos-master
systemctl start marathon
```

访问http://172.26.1.1:5050就能看到Mesos UI界面，如下图所示：

![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/mesosui.png)

点击`Frameworks`标签页，可看到已注册了一个framework，即我们在master上启动的marathon。

![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/frame.png)

以上master的配置完成。
 
下面对两个slave进行配置，根据上面的安排，我们需要在两个slave上安装docker和mesos。

- 1.安装mesos源和docker源。

```
#配置Mesos源
rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
```

- 2.安装docker和mesos。

```
yum install docker mesos
```

- 3.配置mesos-slave。

```
#设置slave支持docker容器。
cd /etc/mesos-slave
vi containerizers
docker,mesos
#设置执行器注册超时时间。
vi executor_registration_timeout
5mins
vi hostname
172.26.1.2（或172.26.1.3）
vi ip
172.26.1.2（或172.26.1.3）
#设置zookeeper地址。
vi /etc/mesos/zk
zk://172.26.1.1:2181/mesos
```

- 4.启动。

```
systemctl start mesos-slave
systemctl start docker
```

启动完之后，再进入Mesos UI界面刷新查看，再点击`Agents`标签页，可以看到两个slave机器都已经连接上了。

![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/agent.png)

从首页可以看到目前可用的资源总量。

![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/resource.png)

至此，服务器的配置结束。

### 第三步，通过marathon来发布一个jenkins应用。

首先，我们需要做一个jenkins镜像，可参考如下`Dockfile`：

```
#基础镜像是centos7
FROM hub.com/centos:7.dumb-init
 
#镜像所有者
MAINTAINER angrytester "thx_phila@yahoo.com"
 
#指定用户
USER root
 
#新建相关目录
RUN mkdir -p /jenkins/jdk1.7.0_75
 
#安装mesos marathon
RUN rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm\
&& yum install -y mesos
 
COPY jdk1.7.0_75 /jenkins/jdk1.7.0_75/
 
COPY jenkins.war /jenkins/
 
#添加启动脚本至根目录
COPY start.sh /
 
#添加相关权限
RUN chmod -R 777 /jenkins/jdk1.7.0_75/\
&& chmod -R 777 /jenkins/jenkins.war
 
#设置JENKINS_HOME环境变量，jenkins启动时自动从JENKINS_HOME下读取配置文件
ENV JENKINS_HOME /jenkins/jenkins_home
 
#设置JAVA_HOME MAVEN_HOME ANT_HOME PATH环境变量
ENV JAVA_HOME /jenkins/jdk1.7.0_75
ENV PATH $PATH:$JAVA_HOME/bin
 
#暴露8080端口
EXPOSE 8080
 
#容器启动时运行脚本
CMD ["/bin/sh","/start.sh"]
```

`start.sh`的内容如下：

```
#!/bin/sh
 
#jenkins启动命令
/jenkins/jdk1.7.0_75/bin/java -jar /jenkins/jenkins.war
```

说明：镜像里面安装了mesos，主要是为后面在jenkins中使用mesos插件准备，因为mesos插件需要用到mesos的库文件。

访问172.26.1.1:8080可以看到marathon的UI界面，点击`Create Application`按钮，`ID`输入：Jenkins，`CPU`输入：0.2，`Memory`输入256，其他默认。

![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/marathonui.png)

点击`Docker Container`，`Image`输入上一步中我们打好的镜像，`Network`选择Host，其他默认。
 
 点击`Volumes`设置容器存储挂载，如下图：
 
 ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/volumn.png)
 
 因为镜像中jenkins的主目录设置为/jenkins/jenkins_home，所以将容器里的对应目录挂载到本地服务器的/jenkins目录下，这样容器里jenkins产生的所有数据都会在本地服务器的/jenkins目录下了。
 
最后点击Create Application按钮，等几分钟，就可以看到任务状态变成了运行中。（需要注意，上面打的镜像我们需要提前docker pull到服务器上，因为marathon会直接docker run镜像，而不会做pull操作）

 ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/running.png)

点击任务查看任务详情，可以看到任务是在哪一台服务器上运行。
 
  ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/jenkins.png)
 
 输入对应服务器IP:8080，就可以访问到对应的jenkins了。
 
在Mesos UI首页，也可以看到正在运行的jenkins任务。

  ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/runningjenkins.png)

点击Sandbox，从下图中两个链接就可以看到jenkins的输出日志（jenkins第一次启动所需的密码可以从这里获得）:

  ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/jenkinslog.png)


### 第四步，我们需要对上一步启动的jenkins做一些配置。

安装mesos插件。

  ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/mesosplugin.png)

插件安装完成之后，进入系统管理→系统设置，会看到多了下面这一项：
 
 ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/addcloud.png)
 
 新增一个云：

 Mesos native library path：/usr/lib/libmesos.so
Mesos Master [hostname:port]：172.26.1.1:5050
On-demand framework registration：no
Jenkins Slave CPUs：0.2
Jenkins Slave Memory in MB：30
Minimum number of Executors per Slave：1
Maximum number of Executors per Slave：1
Jenkins Executor CPUs：0.2
Jenkins Executor Memory in MBs：0.2
Label String：mesos



选中`Use Docker Containerizer`，即使用容器来做构建节点。
  
参考以下Dockfile来构建镜像：

```
#基础镜像是centos7
FROM hub.com/centos:7.dumb-init
 
#镜像所有者
MAINTAINER angrytester "thx_phila@yahoo.com"
 
#指定用户
USER root
 
#新建相关目录
RUN mkdir -p /jenkins/jdk1.7.0_75
 
COPY jdk1.7.0_75 /jenkins/jdk1.7.0_75/
 
#添加相关权限
RUN chmod -R 777 /jenkins/jdk1.7.0_75/
 
#设置JENKINS_HOME环境变量，jenkins启动时自动从JENKINS_HOME下读取配置文件
ENV JENKINS_HOME /jenkins/jenkins_home
 
#设置JAVA_HOME MAVEN_HOME ANT_HOME PATH环境变量
ENV JAVA_HOME /jenkins/jdk1.7.0_75
ENV PATH $PATH:$JAVA_HOME/bin
 
#暴露8080端口
EXPOSE 8080
```

Docker Image中输入上面构建的镜像，其他都保持默认值，保存退出。
  
进入Mesos UI页面可以看到新建的Jenkins框架已经注册成功。

 ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/jenkinsframe.png)

首页可以看到当前资源使用情况如下图：

 ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/reourcenew.png)

占用的资源恰好是我们通过marathon启动jenkins时指定的CPU和内存大小。

### 第五步，测试。

我们新建5个job，只做sleep 100操作，构建节点指定为mesos。
启动所有job，就可以看到

 ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/mulnode.png)

如下图，mesos开头的任务就是新起的构建节点，共启动了5个slave：

 ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/mulslave.png)

根据mesos插件的设置，每个slave占用0.2cpu和30m内存，所以最新的资源使用情况如下：

 ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/resourcenewest.png)

两个slave节点的资源使用情况：

 ![](https://raw.githubusercontent.com/AngryTester/blog/master/Mesos%2BMarathon%2BDocker%2BJenkins%E6%89%93%E9%80%A0%E5%BC%B9%E6%80%A7%E9%AB%98%E5%8F%AF%E7%94%A8%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83/slaveresource.png)

在job构建完成后，slave节点会自动回收。
 
以上就是弹性的持续集成环境最直接的表现了，通过这种方案，我们可以将多台实体机器根据自己的需求分割成任意小块，每一块都可以作为一个构建节点。
 
那么，所谓的高可用呢？
 
下面，我们进入jenkins容器启动的宿主机，将jenkins容器实例强制销毁来模拟服务器出问题的情况。

Jenkins经过一段时间不可访问之后，会自动重启，如果服务器A不可用，会自动在服务器B上重启，这就是marathon的作用了。marathon会不停地对服务进行健康检查，让检查到服务不可用时，就会自动转移故障，在其他可用的服务器上启动服务，这样就保证了持续集成环境的高可用性。由于Jenkins采用的文件系统存储,不同服务器上的配置文件同步,可以依赖Jenkins的插件(SCM Sync Configuration Plugin)完成.

## 写在最后

虽然目前公司已经将持续集成环境由Jenkins转到了gitlab-ci平台,但是Jenkins在业界仍然有着不小的用户群体,因此这篇文章纯作为记录参考.

Mesos算是分布式资源调度管理领域非常权威的一个框架，国内也已经有不少大公司在使用，而随着云技术的蓬勃发展，这个框架的应用场景也会越来越丰富。这次借着这次环境研究的机会，也算熟悉了一下这个框架。整个方案最先是由eBay提出，然后国内公司开始效仿，因为涉及到的技术都还没那么普及，所以整个方案的探索过程基本就是一部血泪史。可预见的是，除了应用在持续集成环境的建设中，这个方案中涉及的例如Mesos、Marathon、Zookeeper等等在未来的云平台建设过程中都有望发挥其作用。本文的这个方案还存在诸多不足，例如如何确保高可用同时的数据一致性，都值得我们进一步探索。

 