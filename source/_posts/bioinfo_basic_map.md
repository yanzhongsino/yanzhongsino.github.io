---
title: 二代和三代reads的常规map操作
date: 2024-08-27
categories: 
- bioinfo
- basic
- map
tags:
- map
- operation
- bwa
- minimap2
- samtools
description: 记录用bwa对二代reads、用minimap2对三代reads进行mapping的常规操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5255807&auto=1&height=32"></iframe></div>

# bwa和samtools对Illumina reads进行mapping

```shell
bwa index ref.fa # 为参考序列建立索引，会生成ref.fa.amb, ref.fa.ann. ref.fa.bwt, ref.fa.pac, ref.fa.sa 五个文件。
bwa mem -t 8 ref.fa sample_R1.fq.gz sample_R2.fq.gz | samtools sort -@ 8 -m 8G |samtools view -@ 8 -F4 -O BAM -o sample.bam & # 进行mapping，mapping后用samtools sort排序，用samtools view -F4删除没mapping上的数据使输出结果文件变小，最后输出sample.bam
samtools index sample.bam # 为sample.bam建立索引，生成索引文件sample.bam.bai。在IGV等软件查看必须要有索引文件。
```

# minimap2对PacBio或Nanopore三代reads进行mapping

```shell
minimap2 -t 8 -ax map-pb ref.fa input.clr.fa | samtools view -@ 8 -bS > clr.aln.bam # PacBio CLR reads
minimap2 -t 8 -ax map-hifi ref.fa input.hifi.fa | samtools view -@ 8 -bS > hifi.aln.bam # PacBio CCS reads，即HiFi reads
minimap2 -t 8 -ax map-ont ref.fa input.ont.fa | samtools view -@ 8 -bS > ont.aln.bam # Oxford Nanopore reads
```

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" 