---
title: linux命令放后台，以及screen多视窗管理
date: 2025-08-06
categories:
- linux
- shell
tags:
- linux
- screen
- nohup

description: linux命令放后台运行，以及用screen创建和管理多视窗。
---

<div align="middle"><iframe data-testid="embed-iframe" style="border-radius:12px" src="https://open.spotify.com/embed/track/4NppPPzU9DK8OvQGnhbB4w?utm_source=generator" width="50%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe></div>

# 1. linux放后台不挂断运行命令
1. `nohup`
- nohup的英文全称是no hang up，意思是不挂起，用于在系统后台不挂断地运行命令，关闭终端或断开SSH连接，不会中断程序运行。
2. `&`
- & 符号用于将命令放到后台执行，意味着在当前窗口可以不用等待当前命令完成，马上执行其他命令。
- 但是当用户退出终端或者关闭终端窗口时，后台运行的命令也会随之停止。
3. `nohup command &`
- nohup和&配合使用，用来将命令放在后台不挂断运行。

# 2. screen
- 如果需要长时间运行并监控的命令，可以用screen来新建窗口，专用于运行此命令。
Screen可以看作是窗口管理器的命令行界面版本。它提供了统一的管理多个会话的界面和相应的功能。
- 在Screen环境下，所有的会话都独立的运行，并拥有各自的编号、输入、输出和窗口缓存。用户可以通过快捷键在不同的窗口下切换，并可以自由的重定向各个窗口的输入和输出。
- Screen实现了基本的文本操作，如复制粘贴等；还提供了类似滚动条的功能，可以查看窗口状况的历史记录。窗口还可以被分区和命名，还可以监视后台窗口的活动。

1. 使用示例
- `screen -S sname`: 新建名为sname的窗口。
- 然后在新窗口中运行任意程序，像在普通终端运行程序一样。
- `ctrl a+d`: 分离当前窗口，并将其移动到后台，并返回主窗口。
- `screen -r sname`：恢复到名为sname的窗口。
- `screen -ls`：列出当前所有screen窗口。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>