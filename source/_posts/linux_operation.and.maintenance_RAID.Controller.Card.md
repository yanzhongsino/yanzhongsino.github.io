---
title: 服务器的硬件和开机流程的简介，以及BIOS和磁盘阵列控制器的设置
date: 2023-04-19
categories: 
- linux
- operation and maintenance
tags:
- server
- hardware
- RAID
- RAID Controller
- BIOS
- Boot Mode
- USB3.0 Extender

description: 以课题组16服务器为例，介绍服务器的硬件，开机页面，BIOS设置（包括修改启动模式Boot Mode），以及磁盘阵列控制器（RAID Controller）的设置。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=399341112&auto=1&height=32"></iframe></div>

# 1. 服务器介绍
## 1.1. 课题组的16服务器硬件介绍
1. 服务器：惠普HPE_ProLiant_DL580_Gen9

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/HPE_ProLiant_DL580_Gen9_Server_按钮和指示灯.jpg?raw=true" title="HPE_ProLiant_DL580_Gen9_Server_按钮和指示灯" width="80%" />

**<p align="center">HPE_ProLiant_DL580_Gen9_Server_按钮和指示灯</p>**

- 服务器内部能看到外装的Dell的LSI MegaRaid SAS Controller磁盘阵列控制卡(RAID Controller)和USB3.0扩展卡(USB3.0 Extender)。
- 两个外装设备都可拆卸。先按蓝色零件开启固定挡板，使固定挡板弹开，即可把RAID Controller和USB3.0 Extender拔出。装回两张卡后，把固定挡板盖上，并按紧（要同时按服务器后面）直到咔的一声表明固定挡板开关关上。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/HPE_ProLiant_DL580_Gen9_Server_服务器内部.jpg?raw=true" title="HPE_ProLiant_DL580_Gen9_Server_服务器内部" width="80%" />

**<p align="center">HPE_ProLiant_DL580_Gen9_Server_服务器内部</p>**

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/HPE_ProLiant_DL580_Gen9_Server_顶盖说明书.jpg?raw=true" title="HPE_ProLiant_DL580_Gen9_Server_顶盖说明书" width="80%" />

**<p align="center">HPE_ProLiant_DL580_Gen9_Server_顶盖说明书</p>**

2. 磁盘阵列（RAID）：戴尔 Dell_PowerVault_MD1200

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/RAID_Dell_PowerVault_MD1200.jpg?raw=true" title="RAID_Dell_PowerVault_MD1200" width="80%" />

**<p align="center">RAID_Dell_PowerVault_MD1200</p>**

3. 磁盘阵列控制卡：服务器自带的Smart Array P830i Controller
4. 磁盘阵列控制卡（RAID Controller）：外装的Dell的LSI MegaRaid SAS Controller（用来连接服务器和磁盘阵列）
- SAS线（连接磁盘阵列和服务器的数据传输线）两端接口一样。
- 16服务器的磁盘阵列有两根，服务器上面的接口接磁盘阵列上面的接口，服务器下面的接口接磁盘阵列下面的接口，不能混接。
- 如果SAS线在服务器连接处的右边，每根SAS线对应亮黄灯，则表明磁盘阵列卡是正常工作连接的。
- 磁盘阵列卡可以取出，先关机并拔除SAS线，然后打开服务器的顶盖，在右上方插着磁盘阵列卡和USB3.0外接卡。需要先按左上方的蓝色按钮打开固定挡板（参考服务器内部的图片），使得固定阵列卡和USB3.0外接卡的开关打开，才能取出两张卡。装回两张卡后，把固定挡板盖上，并按紧（要同时按服务器后面）直到咔的一声表明开关关上。

5. USB3.0扩展卡
- USB3.0扩展卡比磁盘阵列卡小，用于连接移动硬盘等外部存储设备。
- 20230417开机故障把16服务器的USB3.0外接卡移除了，避免它导致服务器降级影响开机。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/USB3.0扩展卡1.jpg?raw=true" title="USB3.0扩展卡1" width="80%" />

**<p align="center">USB3.0扩展卡1</p>**

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/USB3.0扩展卡2.jpg?raw=true" title="USB3.0扩展卡2" width="80%" />

**<p align="center">USB3.0扩展卡2</p>**

## 1.2. 课题组的16服务器软件介绍
1. 内核
- 内核版本号 kernel 4.18.0-147.el8.x86_64，其他版本与磁盘阵列驱动不匹配，会导致无法识别磁盘阵列。开机时需要选择内核版本，默认是最新的4.18.0-240版本。
- `uname -a` 可以查看内核版本

# 2. 操作
## 2.1. 正常开机界面
开机后进行初始化和自检，依次显示如下界面。

1. 开机过程1EarlyProcessorInitialization

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/开机过程1EarlyProcessorInitialization.jpg?raw=true" title="开机过程1EarlyProcessorInitialization" width="80%" />

**<p align="center">开机过程1EarlyProcessorInitialization</p>**

2. 开机过程2BIOS_Boot
- 在这一步可以通过F9进入System Utilities，F10进入Intelligent Provisioning，F11进入Boot Menu，F12进入Network Boot

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/开机过程2BIOS_Boot1.jpg?raw=true" title="开机过程2BIOS_Boot1" width="80%" />

**<p align="center">开机过程2BIOS_Boot1</p>**

- 如果按下F9，屏幕上会对应亮起，需等待检测完毕自动进入Boot界面
<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/开机过程2BIOS_Boot2.jpg?raw=true" title="开机过程2BIOS_Boot2" width="80%" />

**<p align="center">开机过程2BIOS_Boot2</p>**

3. 正常开机
- 如果开机后不进行任何操作，正常开机最后会进入等待登录的命令行界面或图形界面，这时正常开机结束，就可以远程登录啦。

## 2.2. 更改启动模式
16服务器默认是UEFI Mode启动。如果服务器开机故障，可以尝试在BIOS Mode下启动以做检测或更改设置。

1. 开机
2. 经过EarlyProcessorInitialization页面，到BIOS Boot页面时，按下F9，等待系统自动进入System Utilities界面。在这个界面可以进行许多系统的设置。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/F9_SystemUtilities.jpg?raw=true" title="F9_SystemUtilities" width="80%" />

**<p align="center">F9_SystemUtilities</p>**

3. 更改启动模式。依次选择System Configuration-BIOS/Platform Configuration(RBSU)-Boot Options-Boot Mode。Boot Mode代表服务器使用什么模式启动。然后从UEFI Mode转变成Legacy BIOS Mode模式。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/BIOSmode_RBSU_BootMode.jpg?raw=true" title="BIOSmode_RBSU_BootMode" width="80%" />

**<p align="center">BIOSmode_RBSU_BootMode</p>**

4. 还可以更改自动开机设置。依次选择System Configuration-BIOS/Platform Configuration(RBSU)-Server Availability-Automatic Power-On。这里的Always Power On/Off是代表通电后总是自动开机/关机，Restore Last Power State代表通电后使用上次的开机/关机状态来决定是否开机。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/BIOSmode_RBSU_ServerAvailability.jpg?raw=true" title="BIOSmode_RBSU_ServerAvailability" width="80%" />

**<p align="center">BIOSmode_RBSU_ServerAvailability</p>**

5. F10保存所有设置后，Ctrl+Alt+Delete重启服务器，这次是以Legacy BIOS Mode模式启动了。

## 2.3. RAID控制卡的设置
1. RAID控制卡的设置需要在Legacy BIOS Mode模式下启动服务器，参考上面一节操作。
2. Legacy BIOS Mode模式启动服务器的前面的初始化和自检过程一样，先开机过程1EarlyProcessorInitialization，再开机过程2BIOS_Boot，第二个页面只有Boot Mode这一行内容不一样，别的都一样。
3. 然后进入下面的页面，Booting in legacy BIOS mode，出现Expandable RAID Controller BIOS时按下ctrl+R进入RAID控制卡设置界面（ps：出现的时间很短，需要尽快按下ctrl+R）。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/开机过程3Bootmode.jpg?raw=true" title="开机过程3Bootmode" width="80%" />

**<p align="center">开机过程3Bootmode</p>**

4. 磁盘阵列控制卡（RAID Controller）设置界面
- 在这个界面可以更改RAID控制卡的设置

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/RAID设置1VirtualDiskManagement.jpg?raw=true" title="RAID设置1VirtualDiskManagement" width="80%" />

**<p align="center">RAID设置1VirtualDiskManagement</p>**

- 空格键可以显示RAID硬盘的详细参数

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/RAID设置1VirtualDiskManagementDetails.jpg?raw=true" title="RAID设置1VirtualDiskManagementDetails" width="80%" />

**<p align="center">RAID设置1VirtualDiskManagementDetails</p>**

- Tab键在不同的标签页之间转换，在Controller Management标签页可以进行控制卡的管理。空格键可以在是否选中高亮的选项之间转换。改变设置之后要APPLY使其生效。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/operation_and_maintenance/RAID设置2ControllerManagement.jpg?raw=true" title="RAID设置2ControllerManagement" width="80%" />

**<p align="center">RAID设置2ControllerManagement</p>**

5. Enable BIOS Stop on Error选项的解释
- 上图中Settings里的Enable BIOS Stop on Error默认是选上的（前面有X符号）。
- 选上Enable BIOS Stop on Error这个选项代表如果开机时有报错信息就让BIOS停止工作（BIOS停止工作意味着无法正常开机），这个时候如果硬盘或者硬盘的扇区出现问题，或者其他任何硬件出现问题，系统发生降级，RAID控制卡会记录这次降级，并认定为Error信息，从而停止BIOS工作，导致无法正常开机。
- 如果不选这个选项，发生系统降级导致Error信息也不会停止BIOS工作。

6. 可能的开机故障处理方案，去除Enable BIOS Stop on Error设置。
- 开机故障：前段时间（2023/04/17）服务器停电关机后开机出现故障，在开机过程2BIOS_Boot这个页面的Power and Thermal Calibration这一步时，屏幕转入无信号页面并一直保持在无信号页面，服务器不断尝试重启（声音变大变小，无限循环）。ctrl+alt+delete命令重启无效，长按电源键强制重启后服务器的健康监测灯变成红色，有时UID灯也亮起。
- 解决方案就是把Enable BIOS Stop on Error设置改为不选中，就顺利开机了。

7. 设置完毕，应用之后Ctrl+Alt+Delete重启，F9进入BIOS界面修改BIOS Mode回UEFI Mode启动，再重启即可。


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>