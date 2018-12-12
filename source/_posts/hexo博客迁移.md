---
title: hexo博客迁移
date: 2017-10-09 17:12:02
tags: [Hexo]
---

## 写在前面

前段时间为了方便使用docker,将电脑系统由windows换成了ubuntu,加上宝宝出生,一堆事情待做,hexo博客迁移一直没做,今天终于下定决心做迁移,也会陆续将前段时间的一些总结更新到博客.

## 迁移步骤

- 1.环境准备

首先需要准备好新系统上的环境,主要是安装nodejs环境和npm环境.

下载地址:https://nodejs.org/en/download/

tar.xz解压可以直接用`tar xvJf  ***.tar.xz`

<!-- more -->
解压之后做软链接:
```
ln -s /home/xx/node/bin/node /usr/local/bin/node
ln -s /home/xx/node/bin/npm /usr/local/bin/npm
```
`/home/xx/node/bin/node`和`/home/xx/node/bin/npm`分别是解压后的node和npm执行文件的绝对路径.

git环境ubuntu通常自带,不需要另外安装.

- 2.拷贝原有博客目录

需要拷贝的文件和文件夹包括:

```
_config.yml
 package.json
 scaffolds/
 source/
 themes/
```

将以上文件放置到新系统的某一目录下,如`blog`.

- 3.安装hexo.

```
sudo npm install hexo -g
```
- 4.安装hexo模块.

进入blog目录,执行如下命令:

```
sudo npm install
 sudo npm install hexo-deployer-git --save
 sudo npm install hexo-generator-feed --save
 sudo npm install hexo-generator-sitemap --save
```

- 5.测试.

执行如下命令若不返回异常,迁移成功.

```
hexo g
```

若有异常,通常是有部分hexo模块未安装,按照提示用npm安装即可.

## 写在后面

以上迁移操作是在原有博客目录仍存在的情况,若目录不存在以上方法就不适用了,所以决定将博客目录同步到github上,以防丢失.需要同步的目录参考以上需要拷贝的目录即可.
