---
title: Linux系统服务器用户账号的管理
date: 2024-01-09
categories: 
- linux
- operation and maintenance
- user
tags:
- server
- user
- useradd
- usermod
- userdel
- passwd
- groupadd
- groupmod
- groupdel
- newgrp

description: Linux系统服务器用户账号的管理相关操作，包括添加新的用户账号、删除账号、修改账号，用户口令（密码）的管理，以及用户组的新增、删除和修改属性。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1934103823&auto=1&height=32"></iframe></div>

# 1. 用户账号的管理
1. 背景
- Linux系统是一个多用户多任务的分时操作系统，任何一个要使用系统资源的用户，都必须首先向系统管理员申请一个账号，然后以这个账号的身份进入系统。
- 用户的账号一方面可以帮助系统管理员对使用系统的用户进行跟踪，并控制他们对系统资源的访问；另一方面也可以帮助用户组织文件，并为用户提供安全性保护。
- 每个用户账号都拥有一个唯一的用户名和各自的口令（即密码）。
- 用户在登录时键入正确的用户名和口令后，就能够进入系统和自己的主目录。

2. 用户账号的管理主要包括以下三个方面的工作：
- 用户账号的添加、删除与修改。
- 用户口令（密码）的管理。
- 用户组的管理。

# 2. 用户账号的添加、删除和修改
新建用户useradd、删除用户userdel、修改用户usermod都是Linux自带命令，需要root权限。
## 2.1. 新建用户 —— useradd
- 添加用户账号就是在系统中创建一个新账号，然后为新账号分配用户号、用户组、主目录和登录Shell等资源。
- 刚添加的账号是被锁定的，无法使用。

1. 语法
- `useradd 选项 用户名`
- 选项用来配置用户信息，如果无参数，则创建的用户无密码、无主目录、不指定 shell 脚本。
- 用户名用来指定新账号的登录名。
- 除了用户名，其他参数都可缺省，`useradd -D`可print缺省值。
2. 选项参数
- -c comment：指定一段注释性描述
- -d 目录：指定用户主目录，如果此目录不存在，则同时使用-m选项，可以创建主目录。
- -g 用户组：指定用户所属的用户组。
- -G 用户组，用户组：指定用户所属的附加组。
- -s Shell文件：指定用户的登录Shell。
- -u 用户号：指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号。
3. 实例
- `useradd -m -d /home/user -s /bin/bash -g group -G adm,root karen`
- -m 指创建用户同时创建用户主目录。
- 新建用户名为karen的用户。
- 新建用户主目录/home/user，/home为默认用户主目录所在父目录。
- 用户登录shell是/bin/bash，属于group用户组（主组），又属于adm和root用户组（附加组）。
- useradd命令涉及的组若不存在，则需要用groupadd先新建组，再新建用户。
4. 内涵
- useradd新建用户就是在系统文件`/etc/passwd`中增加一条记录，记录新用户信息，同时更新其他系统文件如/etc/shadow，/etc/group等。
5. adduser
- centos系统下adduser和useradd命令用法一样。
- ubuntu系统下有adduser是一个perl脚本，使用时出现交互界面，提供选项让用户填写和选择。
- 使用adduser命令的时候，系统会自动地创建与这个用户名名字一样的用户组作为这个用户的初始用户组，还会自动地在/home目录下面创建一个与用户同名的目录作为用户主目录，接着执行"cp /etc/skel  /home/用户名"的操作，实现新增用户的主目录的初始化。
- 用adduser这个命令创建的账号是系统账号，可以用来登录到我们的ubuntu系统。

## 2.2. 删除用户 —— userdel
如果一个用户的账号不再使用，可以从系统中删除。
1. 语法
- `userdel 选项 用户名`
2. 选项参数
- 常用的选项是 -r，它的作用是把用户的主目录一起删除。
3. 内涵
- 删除用户账号就是要将/etc/passwd等系统文件中的该用户记录删除，必要时还删除用户的主目录。

## 2.3. 修改用户 —— usermod
修改用户账号就是根据实际情况更改用户的有关属性，如用户号、主目录、用户组、登录Shell等。
1. 语法
- `usermod 选项 用户名`
2. 选项参数
- 常用的选项包括-c, -d, -m, -g, -G, -s, -u以及-o等，这些选项的意义与useradd命令中的选项一样，可以为用户指定新的资源值。
3. 实例
- `usermod -s /bin/ksh -g developer -G train -d /disk/karen karen`
- 此命令将用户karen的登录Shell修改为ksh，主目录改为/disk/karen，用户组改为developer，添加新的附加组train。
4. 修改用户名
- 有些系统可以使用选项`-l 新用户名`指定一个新的账号，即将原来的用户名改为新的用户名。

# 3. 用户口令（密码）的管理
用户账号刚创建时没有口令，被系统锁定无法使用，必须指定口令后才能使用，即使是指定空口令。
超级用户可以为自己和其他用户指定口令，普通用户只能用它修改自己的口令。
1. 指定用户密码的命令格式
- `passwd 选项 用户名`
2. 选项
- -l：锁定口令，即禁用账号。
- -u：口令解锁。
- -d：使账号无口令。
- -f：强迫用户下次登录时修改口令。
3. 修改用户密码的实例
- `passwd karen`
- 普通用户修改自己的口令时，passwd命令会先询问原口令，验证后再要求用户输入两遍新口令，如果两次输入的口令一致，则将这个口令指定给用户；而超级用户为用户指定口令时，就不需要知道原口令。
4. 为用户指定空口令：`passwd -d karen`，这个命令将用户 karen 的口令删除，这样用户 karen 下一次登录时，系统就不再允许该用户登录了。
5. 锁定用户，使其不能登录：`passwd -l karen`

# 4. 用户组的管理
- 每个用户都有一个用户组，系统可以对一个用户组中的所有用户进行集中管理。不同Linux 系统对用户组的规定有所不同，如Linux下的用户属于与它同名的用户组，这个用户组在创建用户时同时创建。
- 用户组的管理涉及用户组的添加(groupadd)、删除(groupdel)和修改(groupmod)。组的增加、删除和修改实际上就是对/etc/group文件的更新。
## 4.1. 新增用户组 —— groupadd
1. 语法：`groupadd 选项 用户组`
2. 选项参数
- -g GID：指定新用户组的组标识号（GID）。
- -o：一般与-g选项同时使用，表示新用户组的GID可以与系统已有用户组的GID相同。
3. 实例
- `groupadd group1`：此命令向系统中增加了一个新组group1，新组的组标识号是在当前已有的最大组标识号的基础上加1。
- `groupadd -g 101 group2`：此命令向系统中增加了一个新组group2，同时指定新组的组标识号是101。

## 4.2. 删除用户组 —— groupdel
删除已有用户组的命令很简单：`groupdel group1`命令即删除用户组group1了

## 4.3. 修改用户组属性 —— groupmod
1. 语法：`groupmod 选项 用户组`
2. 选项参数
- -g GID：为用户组指定新的组标识号（GID）。
- -o：一般与-g选项同时使用，表示用户组的新GID可以与系统已有用户组的GID相同。
- -n 新用户组：将用户组的名字修改为新名字。
3. 实例
- `groupmod -g 102 group2`：此命令将组group2的组标识号修改为102。
- `groupmod –g 10000 -n group3 group2`：此命令将组group2的标识号改为10000，组名修改为group3。

## 4.4. 切换用户组 —— newgrp
如果一个用户同时属于多个用户组，那么用户可以在用户组之间切换，以便具有其他用户组的权限。

1. 用户可以在登录后，使用命令newgrp切换到其他用户组，这个命令的参数就是目的用户组。
2. 实例：`newgrp root`
3. 这条命令将当前用户切换到root用户组，前提条件是root用户组确实是该用户的主组或附加组。

# 5. 用户账号管理的集成工具
Linux提供了集成的系统管理工具userconf，它可以用来对用户账号和用户组进行统一管理。


# 6. reference
1. https://www.runoob.com/linux/linux-user-manage.html

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>