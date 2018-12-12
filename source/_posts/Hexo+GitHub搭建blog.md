---
title: Hexo+GitHub搭建blog
date: 2017-02-07 19:58:48
tags: [Hexo,GitHub Page,Windows]
---

Hexo和GitHub就不过多介绍了，网上同类教程也不少，只是因为公司有代理，记录一下搭建过程中的注意点。机器为64位Win7。

## 安装说明

### 1. 安装Git

>下载地址：https://git-scm.com/download/win。
>双击安装完成之后最好将安装路径配置到环境变量中，这样直接就可以在cmd中执行git命令了。

### 2. 安装node

>下载地址：https://nodejs.org/dist/v6.9.5/node-v6.9.5-x64.msi

### 3. 配置Git
<!-- more -->
>因为后面Hexo初始化以及后面Hexo发布都会用到Git，所以如果访问GitHub需要代理，需要提前配置。

``` bash
$  git config --global user.name “AngryTester”
$  git config --global user.email “thx_phila@yahoo.com”
$  git config --global http.proxy http://127.xx.xx.xx
$  git config --global https.proxy http://127.xx.xx.xx
```

### 4. NPM安装Hexo（代理环境下）

>新建一个目录blog，并从cmd中进入blog目录，然后执行如下命令,过程中可能有警告，不用care：

``` bash
$ npm install -g hexo-cli 
```

### 5. Hexo初始化

``` bash
$ hexo init 
```

### 6. Hexo生成静态文件

``` bash
$ hexo g 
```

### 6. Hexo本地发布

``` bash
$ hexo s 
```

>访问 http://localhost:4000 就能访问到博客首页了。

### 7.Hexo发布到GitHub Page

>首先需要申请一个Github库，库名必须为AngryTester.github.io这种格式，AngryTester为你的用户名。然后修改blog目录下的_config.yml文件，修改如下字段:


```
deploy:
  type: git
  repository: https://github.com/AngryTester/AngryTester.github.io.git
  branch: master
```

>然后执行如下命令远程发布：

``` bash
$ hexo d
```

>过程中可能需要输入Git用户名密码。

>访问 https://angrytester.github.io/ 就能访问到你的blog了。










