---
title: 生物信息学常用文件格式
date: 2021-11-12 16:00:00
categories: 
- bio
- bioinfo
tags: 
- gene set enrichment analysis
- GSEA
- topGO
description: 介绍生物信息学常用文件格式。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>






## 用samtools筛选bam文件
1. 删除指定染色体的reads
- `samtools view -h sample_sort.bam | awk '!($3 ~ /NW_01234|NC_02345/)' | samtools view -b - > sample_sort_filter.bam &`
- 这个例子是删除染色体中包含NW_01234或者包含NC_02345的所有reads。
- 其中，`samtools view -h sample_sort.bam`是输出包含header行的所有行。
- `awk '!($3 ~ /NW_01234|NC_02345/)'`是选出第三列中不包含NW_01234也不包含NC_02345的行。
- `samtools view -b -`输出bam格式文件。



-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>


