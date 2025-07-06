---
title: 在Linux系统中设置环境变量PATH
date: 2021-04-20 16:20:15
categories:
- linux
- basics
- PATH
tags:
- linux
- PATH

description: 这篇博客记录了Linux系统下环境变量PATH的设置。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5264843&auto=1&height=32"></iframe></div>

# 1. 环境变量PATH设置
## 1.1. PATH的解释
PATH是环境变量，一个存放有（可执行）命令和程序的目录集合，目录间以冒号:分隔。

操作系统接到用户输入命令时，会**从前到后依次**查看PATH存储的目录，查找与输入命令同名的可执行文件，**一旦查找到就停止查找**，全部目录都查不到就报错。

`echo $PATH`可以查看当前存放了哪些目录。

## 1.2. 设置环境变量
### 1.2.1. 临时设置，当前终端有效
1. 添加目录到环境变量
`PATH=$PATH:/home/usrname/opt/software/`命令把目录`/home/software/`添加到PATH后，并用冒号:分隔，即重新给PATH赋值。

`PATH=$PATH:/home/software/`和`PATH=/home/software/:$PATH`区别在于把目录`/home/software/`置于PATH的最前面还是最后面，最前面代表最优先使用这个目录下的命令。

2. 声明环境变量
`export PATH`命令声明变量，使其对系统shell可识别

以上两条命令可以合并成`export PATH=$PATH:/home/software/`或者`export PATH=/home/software/:$PATH`，此时目录`/home/software/`下的命令可以被当前shell直接调用。

但这种设置方式只在当前shell有效，关掉shell就失效。

### 1.2.2. 当前用户永久设置
登录服务器时，会自动运行当前用户下`~/.bashrc`和`~/.bash_profile`文件内的命令。

所以把`export PATH=/home/software/:$PATH`写入`~/.bashrc`或者`~/.bash_profile`文件就可以使目录`/home/software/`对于当前用户永久生效。

第一次写入后需要用运行命令`source ~/.bashrc`来刷新环境变量，之后就不再需要。

### 1.2.3. 所有用户永久设置
与前一步同理，不过写入的文件是/etc/profile或者/etc/bashrc，需要root用户修改。

第一次修改需要`source /etc/profile`刷新，之后不再需要。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>