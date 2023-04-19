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


# vcftools

1. 可能的问题
- 如果运行vcftools时遇到`Unrecognized values used for CHROM: HiC_scaffold_2 - Replacing with 0`的提示信息，会使得输出的文件sample.map丢失染色体信息，即sample.map文件的第一列全部变成0（本该是染色体ID）。

2. 可选的解决方案
- 创造染色体映射文件，映射文件包含两列染色体ID：`bcftools view -H sample.vcf | cut -f 1 | uniq | awk '{print $0"\t"$0}' > chrom-map.txt`
- 使用创造的染色体映射文件做映射去运行vcftools：`vcftools --gzvcf sample.vcf.gz --plink --chrom-map chrom-map.txt --out sample`


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>


