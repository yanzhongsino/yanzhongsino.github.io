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
- nohup

description: shell的几种循环语法的差异，循环语法内常用的命令，命令连接符和后台运行。
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

# 2. 循环语法内常用命令符号
## 2.1. 命令连接符
1. 命令连接符简介

一行for/while循环中，执行多条命令的代码常写成这种形式：`for i in $(cat file.txt);do grep $i sum.txt && echo $i; j=$i+3;done`。

包含了三条命令：
- `grep $i sum.txt`
- `break`
- `echo $i`

其中`grep $i sum.txt`和`echo $i`以`&&`相连，`j=$i+3`与其他两条命令以`;`相连。

2. 命令连接符的区别
- `&&`和`;`都是用于连接同一行的两条命令的；代表前一条命令执行完再接着执行下一条命令；区别在于`&&`需要上一条命令执行成功才执行下一条命令；`;`则只是命令分隔符，上一条命令执行成功与否不影响下一条命令的执行。
- 另一个命令符`||`，当用`||`连接两条命令时，表示前一条命令执行成功时，停止执行后面的命令；只有当前一条命令执行失败时，才执行后一条命令。

## 循环语法中嵌套`nohup command &`后台执行
1. 不能直接使用`&`
- 在for/while循环内直接使用`&`符号，会使程序提前进入后台，无法进入下一循环。
2. 用`echo | bash`
- 可以使用`echo "nohup command &" | bash`的格式来使命令后台执行，然后快速开始下一命令。
- 如果有多个命令，还可以使用&&连接符，写成`echo "nohup command1 && command2 && command3 &" | bash`

3. 一个例子
- 在这个例子中，会直接把`bash run1_a.sh && bash run1_b.sh && echo "run1 done`放入后台运行，然后开始下一循环，即`bash run2_a.sh && bash run2_b.sh && echo "run2 done`；很快就会把1-9的所有循环都放进后台运行。

```shell
for i in [1-9]*;
    do
        echo "nohup bash run${i}_a.sh && bash run${i}_b.sh && echo "run${i} done" &" | bash
    done
```

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>