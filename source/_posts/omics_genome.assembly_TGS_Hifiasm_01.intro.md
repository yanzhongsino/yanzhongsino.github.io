---
title: 用Hifiasm组装基因组：（一）简介Hifiasm软件和HiFi数据
date: 2024-06-19
categories: 
- omics
- genome
- genome assembly
tags:
- genome assembly
- third generation sequencing
- TGS
- Hifiasm
- HiFi
- PacBio
- Hi-C

description: 为啥Hifiasm是用HiFi数据组装基因组的不二选择? 本文简介了HiFi数据，组装基因组的软件选择，以及介绍Hifiasm软件。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2154795903&auto=1&height=32"></iframe></div>

# 1. HiFi数据
HiFi reads（High Fidelity reads）是2019年由PacBio公司推出的基于环化一致性序列（Circular Consensus Sequencing，CCS）模式产生的既兼顾长读长（10-20kb的长度）又具有高精度（>99%准确率）的测序结果。非常适合用于基因组组装。

1. HiFi数据预处理
- 可以用`bam2fasta`直接把下机数据ccs.bam或者hifi.bam转换成fasta格式文件用于后续分析。
- `bam2fasta sample.ccs.bam -c 9 -o sample.ccs`命令会生成sample.ccs.fasta.gz文件，`-c 9`代表压缩程度为9。

2. 用HiFi数据组装基因组的软件选择
- 2024年发表在Genome Research上的一篇[文章](https://genome.cshlp.org/content/34/2/326) 对11种针对HiFi测序技术的组装工具的评估结果显示，**hifiasm**和**hifiasm-meta**分别是组装真核基因组和宏基因组的最佳工具。
- 文章显示，在真核生物基因组组装中，hifiasm在不同方法比较的组装基因组均具有更高的连续性、完整性和准确性；HiCanu、Verkko与LJA次之，但Verkko与LJA具有组装的contig较短等缺陷；NextDenovo仅对单倍体基因组具有更好的性能。
- 宏基因组组装评估中，hifiasm-meta以及metaflye的组装错误最少，但是在面对复杂宏基因组时hifiasm-meta的完整性及连续性明显优于metaflye，但同时也会保留部分冗余的序列。

目前来说，**Hifiasm软件**是用HiFi数据组装基因组的不二选择。

# 2. Hifiasm软件
1. 简介
- Hifiasm是一个利用PacBio HiFi数据进行从头组装基因组，获得单倍体基因组的组装工具。
- 由哈佛大学李恒团队在2021年2月份开发，首次发表在Nature Methods上。2022年在Nature biotechnology上发表论文，在Hifiasm中引入了Hi-C Integrated assembly 模式。
- Hifiasm被设计用于PacBio HiFi数据组装基因组，使用在分型组装图（pahsed assembly graph）中表示单倍体信息的算法。
2. 特点和优势
- 运行速度很快。半天时间可以组装一个人类基因组。
- 可以接受一种数据类群的多个输入文件（如多个HiFi数据文件），并且合并作为一个文件输入和多个文件输入的结果不同，建议就保持多个文件输入。
- 倾向于尽量组装更长的contigs。
- 能够更好地解决片段重复（segmental duplications）
- 可以利用Hi-C数据或/和亲本二代Illumina测序数据获得解析良好的单倍型组装。
- 也可以利用Oxford Nanopore数据获得端粒到端粒的组装。
3. Hifiasm简化了组装流程
- 内置了清除haplotigs之间的重复（duplications）的程序，无需第三方工具（如purge_dups）。
- 组装完成后无需polish工具（如pilon，racon）进行polish。


# 3. references
1. Hifiasm manual：https://hifiasm.readthedocs.io/_/downloads/en/latest/pdf/
2. Hifiasm介绍：https://www.bilibili.com/read/cv18775152/
3. hifiasm组装（多个cell的HiFi输入文件）的不同结果：https://mdnice.com/writing/25f5a8fe3bfe4474ae1bdcab44604da9

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>