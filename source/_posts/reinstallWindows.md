---
title: 重装windows系统
date: 2020-12-29 21:05:00
categories: 
- computer
    - system
        - Windows

tags: 
- reinstall
- windows
- system
- tutorial

description: 这篇教程的目的是指导小白重装windows系统。
---  


<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=418550511&auto=1&height=66"></iframe></div>

# 小白就一键重装
自己用过黑鲨装机软件，360装机软件等，都有绑定，需要重装之后卸载绑定的软件。

# U盘重装
但强烈不建议一键重装，除非纯小白，建议制作启动U盘：
可用微软官方的制作工具，https://www.microsoft.com/zh-cn/software-download/windows10。

实测使用的是[Rufus](https://rufus.ie/)软件制作U盘，在[msdn网站](https://msdn.itellyou.cn/)下载操作系统ISO文件。
目标系统类型选择UEFI启动(Legacy BIOS启动时间更长)，分区类型选择GPT(比MBR好在可以识别和使用2T以上磁盘)，Legacy BIOS+MBR组合较旧，不推荐。
用软件Rufus制作U盘启动盘（大约30-60分钟），制作完成后插入需装系统的电脑（台式机建议插后面的USB接口）
重启/开机，在出现Dell/Lenovo等商家界面后迅速按F2//F11/F12/ESC等键进入BIOS界面（可查询各厂家进入BIOS界面的快捷键），选择U盘对应的名称，一步步按照指导进行系统安装。


notes：
- 不同电脑主板进入BIOS界面的快捷键不一样，需要查询。
- 进入BIOS界面后，有些主板不显示U盘，可以UEFI设置的“Secure Boot”选项disable掉。
- 有些主板不支持U盘是NTFS文件格式使用UEFI启动，但FAT32最大支持4G文件，所以网上有建议U盘分区，把系统文件放在NTFS区，启动文件放在FAT32区，但我亲试用Rufus制作的NTFS格式U盘可以在Dell inspire 3881上使用（UltraISO制作的就不行）。
- 如果选择U盘名称后键盘鼠标不工作，可以换个USB接口试试，可能系统内的USB驱动版本不匹配。