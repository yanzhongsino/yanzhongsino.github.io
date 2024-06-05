---
title: 三代基因组测序PacBio CLR和CCS模式下机数据的预处理
date: 2024-06-04
categories: 
- omics
- genome
- genome sequencing
- third generation sequencing
tags: 
- genome sequencing
- third generation sequencing
- TGS
- long read sequencing
- continuous long reads
- CLR
- circular consensus sequencing
- CCS
- PacBio
- SMRT
- subreads
- fasta
- data preprocessing
description: 介绍三代基因组测序PacBio CLR和CCS下机数据，命令规则，预处理方式。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=105136&auto=1&height=32"></iframe></div>

# 1. SMRT测序模式
PacBio sequel II平台支持CLR（Continuous Long Reads）和CCS（Circular Consensus Sequencing）两种测序模式。 （sequel平台好像只支持CLR模式）

1. CLR模式适用超长插入片段文库（>20kb），存储有效数据的文件一般命名为* .subreads.bam。对subreads不再进行后续处理。
2. CCS模式则适用于普通长度插入片段文库(1-20kb)，存储数据的* .subreads.bam文件（**subreads**）需要一致性校正后生成CCS reads或HiFi reads用于下游分析。

# 2. CLR模式
1. CLR模式下机数据
- *subreads.bam是以bam格式存储CLR测序数据的下机原始数据。
- 因为每条插入片段只测了一次，不需要进行reads的一致性校正，生成consenses reads这一步骤（Pacbio CCS模式生成的subreads则需要合并生成CCS reads才能用于下游分析）。
2. CLR模式数据文件命名规则
- 文件名：P01TYD19110566-1_r64031_20191018_094227_2_B01.subreads.bam的含义
- P01TYD19110566-1：文库号，即library ID。
- r64031_20191018_094227：样本所在run的ID号码，编号唯一，格式为"r机器号_日期_时间"。
- 2_B01：cell编号。
3. CLR模式数据预处理
- CLR的subreads没有单碱基质量数据，所以bam文件转化为fa文件即可（而不必转为fq文件）。
- 可以用`bam2fasta`直接把*subreads.bam转换成fasta格式文件用于后续分析。`bam2fasta sample.subreads.bam -c 9 -o sample.subreads`命令会生成sample.subreads.fasta.gz文件，`-c 9`代表压缩程度为9。

# 3. CCS模式
1. CCS模式下机数据
- 拿到PacBio CCS模式测序数据时，例如下载公共数据尤其是早期数据时，一定要弄清楚是subreads，还是HiFi reads。
- 对于近期从测序服务商那里得到的数据一般都是运行完CCS软件后的HiFi reads。（文件为下机的ccs.bam或者hifi_reads.bam），可直接用于下游分析。
- 需要注意的是，如果拿到的是subreads.bam数据，则subreads数据需要通过一致性校正后（intra-molecular consensus），得到一条唯一的read，称为**CCS read**，再用于下游分析。这个校正过程会显著提升测序准确率。
2. CCS模式数据的几种类型reads
- polymerase reads：对环状的SMRTbell文库分子测序得到的reads，可能多次读取环状分子。（保存在zmws.bam文件）
- subreads：从polymerase reads序列中去掉哑铃状的测序接头序列，可以得到多条subreads序列。（保存在subreads.bam文件，被废弃的adapter和低质量序列保存在scraps.bam文件）
- CCS reads：环状一致性序列，从一个polymerase reads上的多条subreads序列，进行一致性校正得到的一条准确率更高的序列。（保存在CCS.bam文件）
- HiFi reads (High fidelity reads)：质量值大于Q20的CCS reads，即为HiFi reads。（保存在hifi_reads.bam文件）

<img src="https://cehs.mit.edu/sites/default/files/images/3Part_Reads_Def_0.png" title="CCS reads处理" width="50%" align=center/>

**<p align="center">Figure 1. CCS reads处理，图片来源： https://cehs.mit.edu/pacific-biosciences-sequel</p>**

3. 一致性校正（subreads to CCS reads）
- 如果拿到的是subreads数据，则需要进行一致性校正，把数据转化为CCS reads。
- PacBio官方smrtlink软件中的CCS工具可将subrads.bam转换为ccs.bam，命令如下:`ccs *.subreads.bam  *.ccs.bam`。
- 分割文件并行转换可以加速，下面代码示例为分成10份运行：：

```
ccs *.subreads.bam *.ccs.1.bam --chunk 1/10 -j <THREADS>
ccs *.subreads.bam *.ccs.2.bam --chunk 2/10 -j <THREADS>
...
ccs *.subreads.bam *.ccs.10.bam --chunk 10/10 -j <THREADS>
pbmerge -o *.ccs.bam movie.ccs.*.bam
```

4. CCS模式数据预处理
- 使用CCS read或HiFi reads（通常做一致性校正后reads质量都大于Q20，所以CCS reads就是HiFi reads）可以进行下游分析，有时下游分析需要fa格式。
- CCS reads没有单碱基质量数据，所以bam文件转化为fa文件即可（而不必转为fq文件）。
- 可以用`bam2fasta`直接把*ccs.bam转换成fasta格式文件用于后续分析。`bam2fasta sample.ccs.bam -c 9 -o sample.ccs`命令会生成sample.ccs.fasta.gz文件，`-c 9`代表压缩程度为9。

# references
1. 一致性校正：https://freewechat.com/a/MzA3MjM5NTE4Mg==/2247483719/1
2. CCS模式测序：https://cloud.tencent.com/developer/article/2346853

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>