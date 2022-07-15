---
title: Linux中CPU相关概念
date: 2022-05-02
categories:
- linux
- basics

tags:
- linux
- CPU
- 物理CPU
- 逻辑CPU
- 核心

description: 记录Linux运维中CPU相关概念和查看CPU的命令
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283094&auto=1&height=32"></iframe></div>

# 1. CPU相关概念
中央处理单元(central processing unit, CPU)

在不同的情景中使用的CPU常常代表不同含义，主要区别物理CPU和逻辑CPU。

1. **逻辑CPU(processor数量)**
- 由于物理CPU，核心(core)，SMT产生的计算能力翻倍，计算机的逻辑CPU的数量(也是常常说的总线程数)，符合下面的公式：
**总的逻辑CPU数(processor数量) = 物理CPU数量(计算机主板的CPU硬件/插槽socket的数量) * 每颗物理CPU的核心数(core) * 每个核心的同时多线程数量/超线程数量**

2. **物理CPU(physical CPU)**
- 计算机主板上实际插入的CPU硬件个数，也是主板物理CPU插槽(socket)的数量。
- 通常只有服务器上才会有多个物理CPU，一般的办公/家用计算机主板上只有一个物理CPU。

3. **核心(core)**
- 核心是在物理CPU上的硬件。
- 单个物理CPU上，可能存在单个/多个核心数量(所谓的单核、双核、四核等)。

4. **同时多线程技术(simultaneous multithreading, SMT)**
- 每个核心(core)的同时多线程数量。
- 同时多线程技术(simultaneous multithreading, SMT)是实现单个核心(core)同一时刻能够执行多线程数的技术。以充分利用单个核心(core)的计算能力。AMD和其他CPU厂商常用SMT的称呼。
- 超线程技术(hyper–threading, HT)：Intel的称呼，是SMT的一种具体技术实现。
- 在许多情境下，这个技术产生的计算能力又被称为**虚拟核心(virtual core)**，或者**逻辑处理器(logical processor)**，或者直接称作**线程(thread)**。

# 2. 查询CPU相关信息
1. CPU相关详细信息在文件`/proc/cpuinfo`中
- `cat /proc/cpuinfo| grep "processor"| wc -l` #查看逻辑CPU数量，即processor数量
- `cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l` #查看物理CPU数量
- `cat /proc/cpuinfo | grep "cpu cores" | uniq` #查看单个物理CPU的核心(core)的数量

2. `lscpu`命令会列出CPU相关信息

```shell
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              32 # 逻辑CPU总数，即processor数量
On-line CPU(s) list: 0-31
Thread(s) per core:  2 #每个核心的同时多线程数量
Core(s) per socket:  8 #每个物理CPU的核心数量
Socket(s):           2 #物理CPU数量/插槽数量
NUMA node(s):        2
Vendor ID:           GenuineIntel
CPU family:          6
Model:               62
Model name:          Intel(R) Xeon(R) CPU E5-2650 v2 @ 2.60GHz
Stepping:            4
CPU MHz:             3287.925
CPU max MHz:         3400.0000
CPU min MHz:         1200.0000
BogoMIPS:            5187.48
Virtualization:      VT-x
L1d cache:           32K
L1i cache:           32K
L2 cache:            256K
L3 cache:            20480K
NUMA node0 CPU(s):   0-7,16-23
NUMA node1 CPU(s):   8-15,24-31
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm cpuid_fault epb pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase smep erms xsaveoptdtherm ida arat pln pts md_clear flush_l1d
```

# 3. references
1. wiki：CPU：https://en.wikipedia.org/wiki/Central_processing_unit
2. CPU介绍：https://zhuanlan.zhihu.com/p/86855590

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>