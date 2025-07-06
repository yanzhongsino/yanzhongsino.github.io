---
title: Linux系统中所有环境变量失效的修复方案
date: 2025-07-05
categories:
- linux
- basics
- PATH
tags:
- linux
- PATH

description: Linux系统下环境变量PATH设置错误导致所有环境变量异常，所有命令失效，记录修复方案。
---

<div align="middle"><iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/3UrURlJ4FyHTQVPHsQsii9?utm_source=generator" width="50%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe></div>

# 1. 环境变量设置错误导致的异常
1. 在Linux系统中通常通过修改`~/.bashrc`文件来设置环境变量。有时误操作了.bashrc文件导致所有环境变量异常，甚至常见Linux系统命令（如cat，vim，ls）都无法使用。
2. 许多Linux系统命令（如cat，vim，ls）本来默认已设置为环境变量`$PATH`。
3. 但是如果添加自定义环境变量时，比如执行命令`export PATH=/home/software/:$PATH`时删除了后面的`$PATH`，变成执行命令`export PATH=/home/software/`就会导致操作变成把环境变量替换成`/home/software/`这唯一一个目录。

# 2. 修复方案
## 2.1. 执行命令导致错误
1. 如果只是执行了命令`export PATH=/home/software/`导致异常，那么重开新的shell即可恢复正常。
## 2.2. 修改文件导致错误
1. 如果把命令`export PATH=/home/software/`写进了`~/.bashrc`文件，那么重开新的shell也无法恢复Linux系统命令（如cat，vim，ls）的使用。
2. 此时需要修改`~/.bashrc`文件，但由于`vim`命令无法使用，修改无法完成。
3. 修复方案
- 使用绝对路径`/usr/bin/vi ~/.bashrc`来修改文件。
- 如果找不到`vi`的绝对路径，还可以执行命令`export PATH=/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin`，手动临时把Linux系统命令在的目录加入环境变量。此时`vim`命令可以使用，把`~/.bashrc`文件修改即可。
4. 最后执行`source ~/.bashrc`更新环境变量的设置，就可以正常使用了。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>