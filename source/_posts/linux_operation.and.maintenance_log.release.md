---
title: Linux系统服务器清理系统日志
date: 2024-05-26
categories: 
- linux
- operation and maintenance
- log
tags:
- server
- log

description: Linux系统服务器清理系统日志的操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2193514&auto=1&height=32"></iframe></div>

# 1. 查看系统日志记录情况【需要root权限】
服务器运行久了系统日志会满，有时开机会提示【SEL IS FULL】。这个时候就需要清理系统日志。
系统日志存放在目录`/var/log/journal/`下面。
1. `ll /var/log/journal/`查看系统日志文件
2. `du -t 100M /var`查看`/var`目录下面所有文件，包括系统日志和缓存等。 
3. `journalctl --disk-usage`命令查看，得到`Archived and active journals take up 1.9G in the file system.`表示系统日志文件占了1.9G。

# 清理系统日志文件
1. journalctl命令会自动维护剩余日志文件，按文件大小或产生时间自动维护。【推荐】
- `journalctl --vacuum-size=10M`命令表示只保留最近时间10Mb的日志文件；
- `journalctl --vacuum-time=1w`命令表示只保留最近一周的日志文件。
2. 也可以单独一次清理
- `rm -rf /var/log/journal/*`命令直接删除`/var/log/journal/`目录下的所有文件
3. 清理完之后再使用`journalctl --disk-usage`命令查看，会发现日志文件容量变小。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>