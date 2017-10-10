---
title: SSH Secure File Transfer的编码问题
date: 2017-04-06 11:46:44
tags: [SSH Secure File Transfer，编码]
---

## 问题

在用SSH Secure File Transfer从linux服务器传输zip包到windows后，解压报文件已损坏。

## 解决

查Eidt-Settings-Global Settings-File Transfer-Mode

之前在找其他问题时手贱把mode设置成了ASCII,一般建议设置成Auto selec（自动选择），问题解决！

