---
title: 在Linux系统中替换文本换行符
date: 2021-10-10 09:39:54
categories:
- linux
- text processing
tags:
- linux
- line separator
- xargs
- tr
- paste
- sed
- awk

description: 替换文本的换行符(replace line separator)，把多行文本变成单行文本。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# 替换换行符/多行变单行的几种方法
1. `cat file.txt | xargs echo` #用xargs为echo传入参数，默认是把换行符替换成空格。【推荐】
2. `cat file.txt | tr '\n' ' '` #用tr把换行符替换成空格。【推荐】
3. `cat file.txt | paste -sd " "` #用paste替换；-s是一次处理文件的所有行，而非并行处理每一行；-d指定分割符。
4. `sed ':a;N;s/\n/ /g;ta' file.txt` #用sed处理；sed按行处理所以每次处理会自动添加换行符，`:a`在代码开始处设置标记a，代码执行结尾处通过跳转命令`ta`重新跳转到标号a处，重新执行代码，递归每行；N表示读入下一行。【很慢】
5. `awk 'ORS=" "' file.txt |head -c -1` #用awk处理；ORS(Output Record Separator)设置输出分隔符，把换行符换成空格。head -c -1代表截取文件除了最后一个字符的字符，用于去掉文本末多的分隔符，根据情况使用。
6. `awk BEGIN{RS=EOF}'{gsub(/\n/," ");print}' file.txt` #用awk处理，将RS(record separator)设置成EOF(end of file)，即将文件视为一个记录；再通过gsub函数将换行符\n替换成空格。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>