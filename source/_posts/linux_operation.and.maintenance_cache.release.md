---
title: Linux系统服务器清理缓存
date: 2024-01-09
categories: 
- linux
- operation and maintenance
- cache
tags:
- server
- cache

description: Linux系统服务器释放缓存的操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2193514&auto=1&height=32"></iframe></div>

# 1. 释放缓存的步骤
服务器运行久了可以释放内存缓存，来增加运行速度，释放缓存需要root权限。

1. 查看内存和缓存使用情况
- `free -h`
- buff/cache一栏是缓存占用量
2. 同步
- `sync`
- 释放内存前先使用sync命令做同步，以确保文件系统的完整性。
- 这一步将所有未写的系统缓冲区写到磁盘中，包含已修改的 i-node、已延迟的块 I/O 和读写映射文件。否则在释放缓存的过程中，可能会丢失未保存的文件。
3. 释放缓存
- `echo 1 > /proc/sys/vm/drop_caches`
- 释放缓存，释放后可free -h再次查看缓存释放的效果。可以看到buff/cache一栏显示的缓存占用量大大减少。
- 可以使用`echo 1 > /proc/sys/vm/drop_caches`和`echo 2 > /proc/sys/vm/drop_caches`和`echo 3 > /proc/sys/vm/drop_caches`多次释放缓存，使用第一次效果最好。

# 2. 释放缓存示例

```
root# free -h
              total        used        free      shared  buff/cache   available
Mem:           314G         11G        191G        388K        111G        301G
Swap:           61G         60G         38M
root# sync
root# echo 1 > /proc/sys/vm/drop_caches
root# free -h
              total        used        free      shared  buff/cache   available
Mem:           314G         11G        297G        396K        6.2G        301G
Swap:           61G         60G         38M
root# sync
root# echo 2 > /proc/sys/vm/drop_caches
root# free -h
              total        used        free      shared  buff/cache   available
Mem:           314G         11G        298G        396K        5.3G        301G
Swap:           61G         60G         38M
root# sync
root# echo 3 > /proc/sys/vm/drop_caches
root# free -h
              total        used        free      shared  buff/cache   available
Mem:           314G         11G        299G        396K        4.0G        301G
Swap:           61G         60G         38M
```


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>