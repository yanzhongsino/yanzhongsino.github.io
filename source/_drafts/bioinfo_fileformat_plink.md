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


# plink

## 基本格式（ped+map）

### ped文件
第一列：Family ID。
第二列：Individual ID。自然群体这列和Family ID是一样的。
第三列：Paternal ID。未提供信息的话这列为0。
第四列：Maternal ID。未提供信息的话这列为0。
第五列：Sex。未提供信息的话这列为0。
第六列：Phenotype。一般来说，直接拿vcf转换的话这列为-9，也就是缺失。
第七列开始就是个体在每个标记位点的基因型。

### map文件
第一列：染色体编号。
第二列：SNP编号。
第三列：遗传距离。未提供信息的话这列为0。
第四列：物理位置。

## 二进制格式（bed+bim+fam）


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>


