---
title: docker的四种网络模型
date: 2018-05-17 08:43:53
tags: [docker，网络]
---

## docker自动创建的网络

docker启动后，执行`ifconfig`，可看到多了一个桥接网络-`docker0`。

```
docker0   Link encap:以太网  硬件地址 02:42:a7:b0:df:c3  
          inet 地址:172.172.172.1  广播:0.0.0.0  掩码:255.255.255.0
          inet6 地址: fe80::42:a7ff:feb0:dfc3/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  跃点数:1
          接收数据包:8 错误:0 丢弃:0 过载:0 帧数:0
          发送数据包:43 错误:0 丢弃:0 过载:0 载波:0
          碰撞:0 发送队列长度:0 
          接收字节:536 (536.0 B)  发送字节:6864 (6.8 KB)
```

默认的`inet`应该是`172.17.0.1`，我这里是因为修改了docker的启动参数`bip`，可参考[docker的启动参数配置]().

执行`docker network ls`,可以看到docker默认新建了3个网络：

```
NETWORK ID          NAME                DRIVER              SCOPE
90a48fc6927b        bridge              bridge              local
25e19807e807        host                host                local
717296c87c81        none                null                local
```
运行2个容器：

```
$ docker run -it -d --name=busybox1 busybox
64e5daa13bd1793c2bd3ab973ae8452711d9ca65dda1426bbb89c8ff56915028
$ docker run -it -d --name=busybox2 busybox
7ebc0ef9c5d0066ed6ca926bd6775738c9c4c5cf173ff124f5c02486ce8d5c6e
```

再执行`ifconfig`查看，新增了两个虚拟网卡：

```
enp0s26u1u2c4i2 Link encap:以太网  硬件地址 de:2b:2a:2c:2f:40  
          UP BROADCAST MULTICAST  MTU:1500  跃点数:1
          接收数据包:0 错误:0 丢弃:0 过载:0 帧数:0
          发送数据包:0 错误:0 丢弃:0 过载:0 载波:0
          碰撞:0 发送队列长度:1000 
          接收字节:0 (0.0 B)  发送字节:0 (0.0 B)

veth68e7d15 Link encap:以太网  硬件地址 86:f7:2d:58:9e:7c  
          inet6 地址: fe80::84f7:2dff:fe58:9e7c/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  跃点数:1
          接收数据包:8 错误:0 丢弃:0 过载:0 帧数:0
          发送数据包:30 错误:0 丢弃:0 过载:0 载波:0
          碰撞:0 发送队列长度:0 
          接收字节:648 (648.0 B)  发送字节:4972 (4.9 KB)
```

进入容器内部查看：

```
$ docker attach busybox1
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:AC:AC:02  
          inet addr:172.172.172.2  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: fe80::42:acff:feac:ac02/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:18 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:2689 (2.6 KiB)  TX bytes:578 (578.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

$ docker attach busybox2
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:AC:AC:03  
          inet addr:172.172.172.3  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: fe80::42:acff:feac:ac03/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:38 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:5620 (5.4 KiB)  TX bytes:648 (648.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```
docker引擎为容器分配了虚拟网络设备`eth0`,正好了主机上`ifconfig`中新增的两个网络设备对应，同时分配了`bip`网段的ip。

通过新增的这两个虚拟网络设备和`docker0`形成网络通道，能让容器和主机互联。

通过`docker run`的`-p`参数可将容器内部端口映射到主机端口，访问主机端口即相当与访问对应容器端口。

## docker的none网络

即不使用网络，docker不为容器分配虚拟网络设备和虚拟ip。

使用方式：`docker run --net=none ...`

## docker的host网络

即直接使用主机的网络，网络配置与主机一毛一样。

使用方式：`docker run --net=host ...`

## docker的container网络

即使用已存在的容器网络作为新建容器的网络，即多个容器共用一个虚拟网络(包括虚拟网络设备和虚拟IP完全一致)。

使用方式：`docker run --net=container:容器名 ...`

## docker的自定义bridge网络

之前默认bip和网络代理冲突时我们采取了修改bip来避免冲突，其实还有另外一种方法，即用户可自定义一个bridge网络，然后新建容器指定bridge为用户新建网络。

新建网络命令：

```
docker network create --driver bridge my_network
```

使用方式：`docker run --net=my_network ...`





