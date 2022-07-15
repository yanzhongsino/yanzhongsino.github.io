---
title: shell的几种循环语法
date: 2021-10-10 09:39:54
categories:
- linux
- shell
tags:
- linux
- loop
- for
- while

description: shell的几种循环语法的差异
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# 1. shell的几种循环语法间的差异
## 1.1. for-loop
1. `for i in $(cat file.txt);do echo $i; done`
- for循环每次都是以空格/制表符为分割符输出，所以i变量不是一整行内容，而是单个单元格的内容。想要获取一整行内容，可以先将空格换成别的字符，输出时再替换回来。比如`for i in $(sed 's/\t/#/g' file.txt); do echo $i |sed 's/#/\t/g'; done`
- 【更耗内存】cat会一次性把file.txt所有内容输入到内存中；

## 1.2. while-loop
1. `while read line; do echo $line; done < file.txt`
- 【更省内存】按行读取文件，读取到line变量是一整行内容。

2. `cat file.txt | while read line; do echo $line; done`
- 【更耗内存】cat会一次性把file.txt所有内容输入到内存中；
因为管道|的使用，while read line在子shell中按行调用，line变量也是一整行内容。
这个和多线程控制脚本batch_run.sh冲突。

# 2. 循环语法常用命令符号
## 2.1. && ;
一行for/while循环中，执行多条命令的代码常写成这种形式：`for i in $(cat file.txt);do grep $i sum.txt && echo $i; j=$i+3;done`。

包含了三条命令：
- `grep $i sum.txt`
- `break`
- `echo $i`

其中`grep $i sum.txt`和`echo $i`以`&&`相连，`j=$i+3`与其他两条命令以`;`相连。

`&&`和`;`都是用于连接同一行的两条命令的；代表前一条命令执行完再接着执行下一条命令；区别在于`&&`需要上一条命令执行成功才执行下一条命令；`;`则只是命令分隔符，上一条命令执行成功与否不影响下一条命令的执行。

再多介绍一个命令符`||`，当用`||`连接两条命令时，表示前一条命令执行成功时，停止执行后面的命令；只有当前一条命令执行失败时，才执行后一条命令。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>