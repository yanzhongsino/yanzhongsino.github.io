---
title: Linux系统的服务器开机自动挂载硬盘的设置
date: 2023-04-24
categories: 
- linux
- operation and maintenance
tags:
- server
- mount
- auto

description: Linux系统服务器开机自动挂载硬盘的两种设置方法
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=399341112&auto=1&height=32"></iframe></div>

# Linux系统的服务器开机自动挂载硬盘的设置方法
这里介绍两种方法进行安装了Linux系统的服务器开机后自动挂载硬盘的设置，两种方法各有其特点。使用第一种方法挂载，失败后不影响操作系统运行，还可以手动挂载；第二种方法使用UUID识别硬盘，可以避免因为盘符改变造成的挂载不成功。
## 方法一：在`/etc/rc.local`内添加挂载命令
1. 操作步骤
- `vi /etc/rc.local`打开文件，在文件最后一行添加挂载命令，如挂载`/dev/sdb1`到`/disk1`上则挂载命令为`mount /dev/sdb1 /disk1`，并保存。
- 执行命令`chmod +x /etc/rc.d/rc.local`赋值文件执行权限。
- 下次服务器开机即会执行`/etc/rc.d/rc.local`里的命令。

2. 注意事项
- 在文件`/etc/rc.local`内添加挂载命令，如果输入的挂载命令错误，云服务器重启时不会影响操作系统正常运行。
- 此方法通过盘符进行自动挂载，云硬盘进行挂载卸载操作、云服务器硬重启时盘符会产生改变或者漂移，建议只有一块数据盘（vdb）时采用该方法设置自动挂载。

## 方法二：修改配置文件`/etc/fstab`
1. 操作步骤（root用户）
- 备份`/etc/fstab`文件：`cp /etc/fstab /etc/fstab.bak`
- 查看挂载硬盘的信息（UUID和文件类型）：`blkid /dev/sdb1`
- 显示如下：`/dev/sdb1: UUID="37013e09-db10-4680-ad01-2a141597ce43" BLOCK_SIZE="512" TYPE="xfs" PARTLABEL="primary" PARTUUID="9002b507-946b-496d-8c8f-a09003c96465"`
- 在文件`/etc/fstab`最后一行中写入此硬盘的挂载信息，格式为`UUID=37013e09-db10-4680-ad01-2a141597ce43 /disk1 xfs defaults 0 0`，注意UUID和TYPE一定要填写正确，然后保存。
- 运行命令`cat /etc/fstab`查看挂载信息是否保存

2. 注意事项
- 如果配置文件信息有误，重启云服务器时会进入维护模式，需要修改配置信息正确才能正常进入操作系统。
- 通过将信息写入`/etc/fstab`中进行自动化挂载云硬盘操作时，建议不要使用盘符以及分区id，建议使用文件系统的UUID，因为当云硬盘涉及到挂载和卸载操作时盘符会产生改变或者漂移。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>