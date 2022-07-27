---
title: 基因组质量评估：（五）mapping法：7. 统计BAM文件深度的软件mosdepth
date: 2022-07-25
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- mapping
- sam
- bam
- depth
- biosoft
- mosdepth

description: mapping法评估基因组组装质量。这篇文章主要介绍了统计BAM文件深度的软件mosdepth的安装和使用。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1514742&auto=1&height=32"></iframe></div>

# 1. mosdepth简介
mosdepth（https://github.com/brentp/mosdepth）是用于WGS，exome，targeted sequencing的BAM/CRAM文件的测序深度计算的软件，主要是用Nim语言写的（第一次听说这种语言）。

mosdepth可以得到的数据包括：

1. 每个碱基深度的计算速度是samtools depth的约2倍。——对于 30X 基因组，大约需要 25 分钟的 CPU 时间。
2. 给定窗口大小的平均每个窗口深度，可用于 CNV calling。
3. 给定区域的 BED 文件的每个区域的平均值。
4. 给定窗口大小的每个区域累积覆盖率（cumulative coverage）直方图的平均值或中值。
5. 对于每个染色体和全基因组，在给定阈值或以上覆盖的碱基比例分布。
6. 合并相邻碱基的量化输出，只要它们落在相同的覆盖范围内，例如（10-20）。
7. 阈值输出以指示在给定阈值下每个区域中有多少个碱基被覆盖。
8. 每条染色体和每条染色体指定区域内的平均深度的总结。
9. 一个d4文件（比bigwig好）。

# 2. 下载安装
1. 直接下载已编译文件

```
wget https://github.com/brentp/mosdepth/releases/download/v0.3.3/mosdepth
chmod +x mosdepth
./mosdepth -h
```

# 3. 使用
1. 准备
`samtools index sample.bam` # 生成bam文件的索引文件sample.bam.bai

2. 计算深度
`mosdepth -t 4 out sample.bam`

参数：
- -t 4：线程，需要<=4
- out：输出文件的前缀
- sample.bam：待分析的bam文件
- --by sample.bed：指定区域的bed文件，我分析整个基因组，没加这个参数。

还有许多参数等着探索...

# 4. 结果
1. out.mosdepth.summary.txt

文件包含每条染色体和整个基因组的信息，长度，mapped 碱基数量，平均深度，最小深度和最大深度。示例：
```
chrom	length	bases	mean	min	max
MCscaf001	12541575	440858440	35.15	0	3561
MCscaf002	20211832	749371193	37.08	0	8181
... ...
MCscaf265	25000	897138	35.89	0	84
MCscaf266	25000	913514	36.54	0	82
total	256218469	10404013912	40.61	0	92318
```

2. out.mosdepth.global.dist.txt

文件包含累积分布，指示给定覆盖率阈值下覆盖的总碱基的比例。包含三列：染色体/total，覆盖水平，该级别覆盖的碱基比例。示例：
```
MCscaf001	1961	0.00
MCscaf001	1960	0.00
MCscaf001	1957	0.00
... ...
total	2	0.99
total	1	1.00
total	0	1.00
```

还可以用脚本`python scripts/plot-dist.py \*global.dist.txt`画图，输出`dist.html`，可以看出整个基因组的覆盖度的分布。

3. out.per-base.bed.gz

每个碱基的输出数据。

4. out.per-base.bed.gz.csi


# 5. references
1. https://github.com/brentp/mosdepth

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>