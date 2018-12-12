---
title: Ansible对Windows系统的支持
date: 2017-02-08 10:11:48
tags: [Ansible,Windows]
toc: true
comment: true
---

因为近期有项目考虑用Ansible做Windows机器上的自动投产，因此对Ansible对Windows系统的支持做了一点研究，记录一下。

## 前言

>从Ansible1.7版本开始支持Windows，原理是通过Windows上的PowerShell和Winrm来远程操作Windows服务器。

## Windows机器配置步骤

### 升级PowerShell

>最好升级到4.0以上
>下载地址：http://www.microsoft.com/en-us/download/details.aspx?id=40855
<!-- more -->

### 配置Winrm

```bash
$powershell.exe -File ConfigureRemotingForAnsible.ps1
```

>ConfigureRemotingForAnsible.ps1下载地址： https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1

>可能会出现netsh相关报错，请把C:\Windows\System32加入到PATH环境变量。

>确认Winrm已开启
```bash
$winrm quickconfig
```
>配置Winrm
```bash
$winrm set winrm/config/service/auth '@{Basic="true"}'
$winrm set winrm/config/service/auth '@{Basic="true"}'
```

### 关闭防火墙

>控制面板-系统和安全-Windows 防火墙

以上Windows机器的配置就完成了。

## 主控机配置步骤

>主控机必须是Linux系统，我用的centos7。

### 安装Ansible

>安装rpm源

```bash
$rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm 
```

>安装ansible
```bash
$yum install -y ansible
```

### 安装pywinrm

>安装pip
```bash
$yum install -y python-pip 
```
>安装pywinrm
```bash
$pip install "pywinrm>=0.1.1"
```

### 配置hosts
```bash
$vi /etc/ansible/hosts
```

>参考如下内容：
```
[win7]
172.XX.X.XX

[win7:vars]
ansible_user= "XX"
ansible_password= "XXXXXXX"
ansible_ssh_port= 5985
ansible_connection= winrm
ansible_winrm_server_cert_validation= ignore
```

## 测试

>主控机测试
```bash
$ansible win7 -m win_ping
```
>应该返回如下内容
```
172.XX.X.XX | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```



暂时完成连接，后续再研究playbook相关及实际应用。




