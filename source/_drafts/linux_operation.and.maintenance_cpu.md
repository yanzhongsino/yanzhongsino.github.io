---
title: Linux中CPU相关概念
date: 2022-04-11
categories:
- linux
- operation and maintenance

tags:
- linux
- 运维
- CPU

description: 记录Linux运维中CPU相关概念和查看CPU的命令
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2117115&auto=1&height=32"></iframe></div>

# CPU
中央处理单元(central processing unit, CPU)。

在不同的情景中使用CPU常常代表不同含义，主要区别物理CPU和逻辑CPU。

## 物理CPU(physical CPU)
计算机主板上实际插入的CPU硬件个数，也是主板物理CPU插槽(socket)的数量。
通常只有服务器上才会有多个物理CPU，一般的办公/家用计算机主板上只有一个物理CPU。

`cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l` #查看物理CPU数量

## 核心(core)
核心是在物理CPU上的硬件。

单个物理CPU上，可能存在单个/多个核心数量(所谓的单核、双核、四核等)。

单个物理CPU上有几个核心就意味着计算机可以同时运行几个进程/线程。

`cat /proc/cpuinfo | grep "cpu cores" | uniq` #查看单个物理CPU的核心(core)的数量

## 同时多线程技术(simultaneous multithreading, SMT)
同时多线程技术(simultaneous multithreading, SMT)是实现单个核心(core)同一时刻能够执行多线程数的技术。以充分利用单个核心(core)的计算能力。

AMD和其他CPU厂商常用SMT的称呼。
超线程技术(hyper–threading, HT)：Intel的称呼，是SMT的一种具体技术实现。

在许多情境下，这个技术产生的计算能力又被称为**虚拟核心(virtual core)**，或者**逻辑处理器(logical processor)**，或者直接称作**线程(thread)**。

## 逻辑CPU
由于物理CPU，核心(core)，SMT产生的计算能力翻倍，计算机的逻辑CPU的数量(也是常常说的总线程数)，符合下面的公式：

总的逻辑CPU数(processor数量) = 物理CPU数 * 每颗物理CPU的核心数(core) * 每个核心的同时多线程数量/超线程数量

`cat /proc/cpuinfo| grep "processor"| wc -l` #查看逻辑CPU数量


`lscpu` #列出CPU相关信息

```
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              32
On-line CPU(s) list: 0-31
Thread(s) per core:  2
Core(s) per socket:  8
Socket(s):           2
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

# references
1. https://en.wikipedia.org/wiki/Central_processing_unit
2. https://zhuanlan.zhihu.com/p/86855590