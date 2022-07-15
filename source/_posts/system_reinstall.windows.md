---
title: 重装windows系统
date: 2020-12-29 21:05:00
categories: 
- computer
- system
- Windows

tags: 
- tutorial
- reinstall
- windows
- system

description: 这篇教程的目的是指导小白重装windows系统。
---  


<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=418550511&auto=1&height=66"></iframe></div>

# 1. 小白就一键重装
自己用过黑鲨装机软件，360装机软件等，都有绑定，需要重装之后卸载绑定的软件。
一键重装只能在电脑正常可进入桌面和使用软件的情况下进行，许多情况下不适用。

# 2. U盘重装
但强烈不建议一键重装，除非纯小白，建议在正常运行的电脑上制作启动U盘，然后使用U盘启动和安装需要更换系统的电脑：

## 2.1. 【推荐】微软官方工具
可用微软官方的制作工具，[微软官方工具](https://www.microsoft.com/zh-cn/software-download/windows10)，速度比msdn网站快。

### 2.1.1. 创建启动U盘的步骤
进入网站，根据**使用该工具创建安装介质(USB闪存驱动器、DVD或ISO文件），以在其他电脑上安装Windows10**指引操作，用一个>=8G的空U盘制作U盘启动盘。

1. 插上>=8G的空白U盘（U盘会被格式化）
2. 选择**立即下载工具**，选择**运行**（需要管理员权限）
3. 同意许可条款
4. 选择**为另一台电脑创建安装介质**，选择**下一步**
5. 选择Window10的语言、版本和体系结构（64/32位）
6. 选择介质类型（USB闪存驱动器或ISO文件，其中ISO文件用于创建启动DVD），选择USB闪存驱动器。
7. 等待创建完成

### 2.1.2. U盘启动安装
根据网站上接下来的**使用您所创建的安装介质**，为电脑安装新的Window10系统，步骤如下
1. 需要安装系统的电脑的所有硬盘需要备份。
2. 把制作好的启动U盘连接到等待安装系统的电脑上
3. 重启电脑
4. 有时电脑会自动引导至USB介质启动安装；如果电脑没有自动引导至 USB 或 DVD 介质，可能需要打开引导菜单或在电脑 BIOS 或 UEFI 设置中更改引导顺序。
5. 若要打开引导菜单或更改引导顺序，需要在重启/开机后，在出现Dell/Lenovo等商家界面后立即按下按键（例如 F2、F12、Delete 或 ESC）进入BIOS界面（不同品牌电脑不一样，可查询各厂家进入BIOS界面的快捷键），选择U盘对应名称，一步步按照指导进行系统安装。如果没有看到 USB 或 DVD 介质设备在引导选项中列出，可能需要联系电脑制造商来获取在 BIOS 设置中暂时禁用“安全引导”的说明。
6. 在**安装Windows**页面上，选择语言、时间和键盘首选项，然后选择**下一步**；
7. 选择**安装Windows**。


## 2.2. msdn下载系统镜像+rufus制作启动U盘
这里可以替代上一步中的**创建启动U盘的步骤**；制作完成后**U盘启动安装**与上一步一致。

在[msdn网站](https://msdn.itellyou.cn/)下载需要的操作系统版本的ISO文件；然后用[Rufus](https://rufus.ie/)软件制作U盘。

下好系统镜像和rufus软件后，插上U盘在rufus软件上依次操作制作启动U盘。

几个需要注意的选项：
- 目标系统类型选择UEFI启动(Legacy BIOS启动时间更长)，分区类型选择GPT(比MBR好在可以识别和使用2T以上磁盘)，Legacy BIOS+MBR组合较旧，不推荐。
- 用软件Rufus制作U盘启动盘（大约30-60分钟），制作完成后插入需装系统的电脑（台式机建议插后面的USB接口）


## 2.3. notes：
- 不同电脑主板进入BIOS界面的快捷键不一样，需要查询。
- 进入BIOS界面后，有些主板不显示U盘，可以UEFI设置的“Secure Boot”选项disable掉。
- 有些主板不支持U盘是NTFS文件格式使用UEFI启动，但FAT32最大支持4G文件，所以网上有建议U盘分区，把系统文件放在NTFS区，启动文件放在FAT32区，但我亲试用Rufus制作的NTFS格式U盘可以在Dell inspire 3881上使用（UltraISO制作的就不行）。
- 如果选择U盘名称后键盘鼠标不工作，可以换个USB接口试试，可能系统内的USB驱动版本不匹配。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>