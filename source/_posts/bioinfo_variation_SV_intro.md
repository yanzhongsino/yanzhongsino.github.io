---
title: 结构变异分析
date: 2022-10-09
categories: 
- bioinfo
- variation
- structural variation
tags: 
- structural variation
- SV
- variation
- insertion
- deletion
- Indel
- SNP
- tandem expansion
- tandem contraction
- repeat expansion
- repeat contraction
- biosoft
- Assemblytics
- MUMandCo

description: 介绍了两个基因组间的结构变异的分析，和两个分析软件：MUM&Co和Assemblytics。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=105207&auto=1&height=32"></iframe></div>

# 1. 变异（variation）
不同生物间或者同一种生物不同个体间，染色体上同一位置对应的序列的碱基或者结构不同，或者同一序列在染色体上的位置不同，这种生物间遗传物质的不同被称为变异（variation）。变异是至少两个个体的遗传物质比较的结果。

## 1.1. 变异的类型
基因组上的变异有不同的类型，包括单核苷酸多态性（Single Nucleotide Polymorphisms，SNP），插入缺失（Insertion and Deletion, Indel），和结构变异（structural variation，SV）。

1. 单核苷酸多态性（SNP）
- 指单个核苷酸的碱基的变化，比如A变成了G，或者C变成了T。
2. 插入删除（Indel）
- 指基因组上某个位置发生的较短的线性序列的插入（Insertion）或者删除（Deletion）。插入缺失是两者比较的结果，对一方是插入的，对另一方来说就是删除，所以通常合称为Indel。
- 通常约定插入或缺失长度在50bp以下的被称为Indel，大部分情况下不超过10bp。
3. 结构变异（SV）
- 指长片段的序列变化或位置变化，包括长片段的插入删除（Big Indel），染色体倒位（Inversion），易位（Translocation），串联重复（Tandem repeat），拷贝数变异（copy number variation, CNV）等。

## 1.2. 结构变异（structural variation，SV）
### 1.2.1. 结构变异的定义
结构变异（structural variation，SV）是指基因组上长片段的序列变化和位置关系变化。
过去通常定义为长度大于1000bp的插入删除和倒位，但随着测序的发展，现在的定义更广泛。

### 1.2.2. 结构变异的类型
目前的结构变异通常包括长度大于50bp的长片段序列插入删除（Big Indel），长度大于1000bp的染色体倒位（Inversion），长度大于1000bp的染色体内部或染色体间的序列易位（Translocation），拷贝数变异（copy number variation, CNV），串联重复（Tandem repeat）以及形式更为复杂的嵌合性变异等。

有些标准还包括简单重复序列（SSR），散在的重复（Interspersed duplications）和其他重复序列。

<img src="https://media.springernature.com/full/springer-static/image/art%3A10.1038%2Fnrg2958/MediaObjects/41576_2011_Article_BFnrg2958_Fig1_HTML.jpg?as=webp" width=90% title="结构变异的类型" alt="结构变异的类型" align=center/>

**<p align="center">图1. 结构变异的类型**
图片来源： [paper：Genome structural variation discovery and genotyping](https://www.nature.com/articles/nrg2958)</p>

# 2. 检测结构变异的方法
检测结构变异的方法主要有四种，包括：
1. Read-pair (RP)：通过对双端测序reads的距离（即插入序列长度的分布）或位置关系（即双端reads的正反链情况）的异常值分析来检测结构变异。
2. Split-read (SR)：通过对双端测序reads中一条reads能比对上，另一条不能比对上的情况进行分析来检测结构变异。
3. Read-depth (RD)：利用read的mapping深度来检测基因组拷贝数变异（Copy number variantion，简称CNV）的方法。
4. Assembly (AS)：通过三代测序和de novo assembly来检测大长度和复杂结构的变异。

下图总结了四种方法可以检测的结构变异的类型。

<img src="https://media.springernature.com/full/springer-static/image/art%3A10.1038%2Fnrg2958/MediaObjects/41576_2011_Article_BFnrg2958_Fig2_HTML.jpg?as=webp" width=90% title="结构变异的检测方法" alt="结构变异的检测方法" align=center/>

**<p align="center">图2. 结构变异的检测方法**
图片来源： [paper：Genome structural variation discovery and genotyping](https://www.nature.com/articles/nrg2958)</p>

# 3. 检测结构变异的软件
检测结构变异的软件有很多，个人用过的两个贴在下面：
- MUMandCo：https://yanzhongsino.github.io/2022/07/17/bioinfo_variation_SV_MUMandCo/
- Assemblytics：https://yanzhongsino.github.io/2022/08/02/bioinfo_variation_SV_Assemblytics/


# 4. references
1. https://en.wikipedia.org/wiki/Structural_variation
2. https://www.ncbi.nlm.nih.gov/dbvar/content/overview/#:~:text=Structural%20variation%20(SV)%20is%20generally,copy%20number%20variants%20(CNVs).
3. SV的检测算法和原理-知乎：https://zhuanlan.zhihu.com/p/40290546
4. SV的检测算法和原理-文章：https://www.nature.com/articles/nrg2958
5. SV的检测算法和原理-文章：https://academic.oup.com/bib/article/16/5/852/217239?login=false

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>