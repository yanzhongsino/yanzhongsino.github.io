---
title: Linux的常用命令连接符（command connector）
date: 2022-01-16 19:30:54
categories:
- linux
- shell
tags:
- linux
- shell
- command connector

description: 记录Linux常用的命令连接符
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2117115&auto=1&height=32"></iframe></div>

# 1. 常用命令连接符

多条命令放在一行时，需要在命令之间使用命令连接符，包括`;`,`&&`,`||`等。

## 1.1. `;`
`;`是最简单的命令连接符。

`command1; command2; command3`表示多个命令顺序执行，命令间没有逻辑联系。

## 1.2. `&&`
`&&`逻辑与命令连接符。

`command1 && command2 && command3`表示从左往右按顺序执行命令，如果中间有命令执行失败，则中断在失败的命令处不继续往下执行。即左边的命令执行成功才执行右边的命令；当左边命令执行失败就不会执行右边命令。

## 1.3. `||`
`||`逻辑或命令连接符。

`command1 ||command2 || command3`表示从左往右按顺序执行命令，如果中间有命令执行成功，则中断在成功的命令处不继续往下执行。即左边的命令执行成功时，停止执行右边的命令；只有当左边的命令执行失败时，才执行右边的命令。


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>