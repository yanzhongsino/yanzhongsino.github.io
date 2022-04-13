---
title: Linux运维(Ops)命令 —— top
date: 2022-04-11
categories:
- linux
- Ops
tags:
- linux
- Ops
- 运维
- top

description: 记录Linux运维(Ops)命令 —— top的用法
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2117115&auto=1&height=32"></iframe></div>

# 常用运维命令 —— top
`top` 用于交互式监控Linux上动态实时进程，按线程数从大到小排序

## top的默认输出
在shell终端输入`top`回车，就可以查看许多系统运行的实时信息。如下图所示：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/linux_command.top.default.jpg?raw=true" width=90% title="top默认输出示例" align=center/>

**<p align="center">Figure 1. top默认输出示例</p>**

top的默认输出包含两个区域：水平信息的汇总区（红框部分）和垂直信息的任务区（蓝框部分）。汇总区显示有关进程和资源使用情况的统计信息，而任务区显示当前正在运行的所有进程的列表。

下面一一说明各个参数的含义：

### 汇总区
汇总区共有5行。

1. 第一行：任务队列信息。同`uptime`命令输出一致。

|显示|含义|
|---|---|
|top - 15:30:36|系统当前时间(system time)|
|up 29 days, 54 min|系统运行时长(uptime)|
|7 users|当前登录的用户/会话数量(user sessions)|
|load average: 4.92, 4.90, 4.96|系统负载，即任务队列的平均长度。三个数值分别为1min，5min，15min前到现在的平均值|

2. 第二行：任务进程信息 Tasks

|显示|含义|
|---|---|
|498 total|进程总数|
|3 running|正在运行的进程数|
|316 sleeping|睡眠状态的进程数|
|0 stopped|停止状态的进程数|
|0 zombie|僵尸状态的进程数|

3. 第三行：CPU信息 %Cpu(s)。如果有多个CPU，可能会有多行，每行单独显示一个CPU。

|显示|含义|
|---|---|
|6.1 us|用户空间占用CPU百分比|
|0.2 sy|内核空间占用CPU百分比|
|0.0 ni|用户进程空间内改变过优先级的进程占用CPU百分比|
|93.7 id|空闲CPU百分比|
|0.0 wa|等待输入输出的CPU时间百分比|
|0.0 hi|硬件CPU中断占用百分比|
|0.0 si|软中断占用百分比|
|0.0 st|虚拟机占用百分比|

4. 第四行：内存信息 KiB Mem

|显示|含义|
|---|---|
|33012627+total|物理内存总量|
|93989232 free|空闲内存总量|
|27405860 used|使用的物理内存总量|
|20873120+buff/cache|用作内核缓存的内存量|

5. 第五行：交换区信息 KiB Swap

|显示|含义|
|---|---|
|63998972 total|交换区总量|
|63280912 free|空闲交换区总量|
|718060 used|使用的交换区总量|
|30034761+avail Mem|缓冲的交换区总量。内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖，该数值即为这些内容已存在于内存中的交换区的大小,相应的内存再次被换出时可不必再对交换区写入|

### 任务区
任务区每行代表一个进程的信息。

默认情况下仅显示部分列： PID、USER、PR、NI、VIRT、RES、SHR、S、%CPU、%MEM、TIME+、COMMAND 。使用不同参数可以更改显示内容。

这里把每一列的含义列在这里：

|序号|列名|含义|
|---|---|---|
|1|PID|进程id|
|2|PPID|父进程id|
|3|RUSER|真实用户名Real user name|
|4|UID|进程所有者的用户id|
|5|USER|进程所有者的用户名|
|6|GROUP|进程所有者的组名|
|7|TTY|启动进程的终端名。不是从终端启动的进程则显示为 ?|
|8|PR|优先级|
|9|NI|nice值。负值表示高优先级，正值表示低优先级|
|10|P|最后使用的CPU，仅在多CPU环境下有意义|
|11|%CPU|上次更新到现在的CPU时间占用百分比|
|12|TIME|进程使用的CPU时间总计，单位秒|
|13|TIME+|进程使用的CPU时间总计，单位1/100秒|
|14|%MEM|进程使用的物理内存百分比|
|15|VIRT|进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES|
|16|SWAP|进程使用的虚拟内存中，被换出的大小，单位kb。|
|17|RES|进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA|
|18|CODE|可执行代码占用的物理内存大小，单位kb|
|19|DATA|可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb|
|20|SHR|共享内存大小，单位kb|
|21|nFLT|页面错误次数|
|22|nDRT|最后一次写入到现在，被修改过的页面数。|
|23|S|进程状态(D=不可中断的睡眠状态,R=运行,S=睡眠,T=跟踪/停止,Z=僵尸进程)|
|24|COMMAND|命令名/命令行|
|25|WCHAN|若该进程在睡眠，则显示睡眠中的系统函数名|
|26|Flags|任务标志，参考 sched.h|


## top的常用参数



## top的常用交互命令
要终止此会话，您可以按键盘上的“q”。

### 更改任务区的显示内容
通过 f 键可以选择显示的内容。按 f 键之后会显示列的列表，按 a-z 即可显示或隐藏对应的列，最后按回车键确定。
更改列显示顺序
按 o 键可以改变列的显示顺序。按小写的 a-z 可以将相应的列向右移动，而大写的 A-Z 可以将相应的列向左移动。最后按回车键确定。
按列排序
按大写的 F 或 O 键，然后按 a-z 可以将进程按照相应的列进行排序。而大写的 R 键可以将当前的排序倒转。

# references
1. [top的manual](https://man7.org/linux/man-pages/man1/top.1.html)
2. [top命令指南](https://www.booleanworld.com/guide-linux-top-command/)
3. [博客：top输出详解](https://www.jianshu.com/p/af584c5a79f2)