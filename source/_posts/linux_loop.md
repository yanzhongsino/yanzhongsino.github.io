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

# shell的几种循环语法间的差异
## for-loop
1. `for i in $(cat file.txt);do echo $i; done`
- for循环每次都是以空格/制表符为分割符输出，所以i变量不是一整行内容，而是单个单元格的内容。想要获取一整行内容，可以先将空格换成别的字符，输出时再替换回来。比如`for i in $(sed 's/\t/#/g' file.txt); do echo $i |sed 's/#/\t/g'; done`
- 【更耗内存】cat会一次性把file.txt所有内容输入到内存中；

## while-loop
1. `while read line; do echo $line; done < file.txt`
- 【更省内存】按行读取文件，读取到line变量是一整行内容。

2. `cat file.txt | while read line; do echo $line; done`
- 【更耗内存】cat会一次性把file.txt所有内容输入到内存中；
因为管道|的使用，while read line在子shell中按行调用，line变量也是一整行内容。
这个和多线程控制脚本batch_run.sh冲突。
