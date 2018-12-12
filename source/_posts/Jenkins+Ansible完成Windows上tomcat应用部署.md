---
title: Jenkins+Ansible完成Windows服务器上的tomcat应用部署
date: 2017-02-13 14:22:47
tags: [Jenkins,Ansible,Tomcat]
---

## Jenkins

>Jenkins完成代码自动编译打包的任务。

### Jenkins配置注意

>建议安装的插件包括版本管理（SVN/GIT），邮件，Ansible。
>在打包成功之后，在后置操作中选择调用Ansible playbook。

![](https://raw.githubusercontent.com/AngryTester/blog/master/ansible.jpg)

## Ansible

>Ansible完成war包发送和tomcat的启停。
<!-- more -->
### Ansible脚本参考

![](https://raw.githubusercontent.com/AngryTester/blog/master/d.jpg)

## 注意点

- 脚本中的async和poll是因为tomcat属于长时应用，在Windows上不会退出，所以脚本结束不了，相当于通过强制手段让进程转后台运行。

- 由上面一点带来的问题就是Ansible脚本无法正常结束，因此会在Windows上启动一个powershell.exe进城（Ansible就是通过这个与Windows进行通信），随着部署次数增多，Windows上的powershell进城会越来越多，因此建议每次部署完手动杀一下Windows上的powershell进程。这也是当前这个方案最不完善的地方，但是因为我们通常是在一台服务器上多个tomcat，所以这个方案目前主要解决的同服务器上的多应用部署，每次部署只需要处理一次，所以还好。

- 因此Ansible与Windows通信前需要先关闭Windows防火墙，部署完之后需要开启，所以可以和上一步同时操作。即部署前通过以下命令关闭防火墙：

	`netsh firewall set opmode disable`

	部署完后通过以下命令开启防火墙并杀掉powershell进程：

	`netsh firewall set opmode disable`

	`taskkill /f /im powershell.exe`


##小技巧

>可通过以下shell来轮询环境是否可用

```
STATUS_CODE=0
while [[ $STATUS_CODE != 200 ]]
do
    STATUS_CODE=`curl -o /dev/null -s -w %{http_code} http://localhost:8080/test/`
done
```

>其中http://localhost:8080/test/为你的环境。








