---
title: 基因组质量评估：（五）mapping法：4. 观察mapped reads的深度分布
date: 2022-07-27
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- organelle
- mitogenome
- plastome
- mapping
- sam
- bam
- depth
- samtools

description: mapping法评估基因组组装质量。这篇文章简单概述了mapping法的其中一个评估指标：depth，主要介绍了如何通过观察mapped reads的深度分布来判断可能的组装错误和组装特征区域，尤其是线粒体的组装质量评估。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=26515663&auto=1&height=32"></iframe></div>

# 1. 前情提要：mapping和depth
此篇博客在已经通过mapping获得SAM/BAM文件，且进行深度统计获得深度分布文件的基础上进行分析，

1. 关于mapping获得SAM/BAM文件的操作可以参考博客：

[基因组质量评估：（五）mapping法：1. 简介](https://yanzhongsino.github.io/2022/07/23/omics_genome_quality.assessment_mapping_intro/)

1. 关于通过SAM/BAM文件统计深度分布可以参考博客：

[基因组质量评估：（五）mapping法：3. 统计mapped reads的深度分布](https://yanzhongsino.github.io/2022/07/27/omics_genome_quality.assessment_mapping_depth_samtools/)

# 2. 深度分布
- 在测序是随机分布的情况下，期望在基因组的所有染色体上，mapped reads的depth是均匀分布的。
- 把reads回mapping到组装好的基因组，通过计算depth，观察depth在基因组上的分布来判断组装的质量。

# 3. 评估对象
用clean reads映射（mapping）回组装好的初始基因组，然后查看mapping的效果。

- 如果是长度不长的细胞器基因组，或者核基因组的特定位置，可以在IGV直接查看mapping情况。
- 如果是大的核基因组，则考虑过滤mapping得到的sam/bam文件，然后计算深度和其他统计值来评估质量。

# 4. 查看深度
## 4.1. 背景
1. 由于**叶绿体基因组**结构稳定，较少需要用这种方式评估组装质量；**核基因组**较大，肉眼观察不可实现，所以此方法多用于**线粒体基因组**组装质量和核基因组部分区域的评估。
2. 在核与细胞器同时测序情况下，由于数量分布差异，期望测序深度是叶绿体>线粒体>核基因，且三者差异是数量级级别的。
3. 评估线粒体基因组时，可以考虑是否有叶绿体/核的reads映射到基因组上，并同时考虑水平基因转移的可能性。

## 4.2. 深度分布的查看
查看mapped reads的深度，在全基因组范围是否保持相当。如果有极端高/低的深度，则怀疑组装错误，或者映射了错误的reads，反映了错误或者特定特征的mapped reads。

1. 重复序列
- 如果基因组上存在重复序列，组装时只得到其中一个拷贝，则在这个拷贝处的mapped reads深度会是附近序列的两倍（三个拷贝就三倍）。
- 重复序列一般只分析超过100bp(或者>50bp)的情况，这可以和核基因转移到线粒体的情况区别开。
- 重复序列之间非常接近，但不一定是完美匹配。

2. 水平基因转移【线粒体】
- 显著高的深度可能是叶绿体的reads。
- 显著低的深度可能是核的reads。
- 轻微高的深度可能是核转移到线粒体的情况（一般这种情况映射到的序列不长，<100bp）。可以调整参数，只保留映射超过100bp的reads即可排除这种情况。

3. 异质性【线粒体】
由于一个细胞中有多个细胞器，细胞器之间还可能存在不同构象或不同碱基的位点。这种情况的存在称为**细胞器的异质性**（不常见）。

具有异质位点的细胞器基因组的证据：
- 排除重复序列映射的可能性。
- 异质位点映射的reads包含两种（也可能多种，以下同）碱基，一般少数种的占比超过5%就可能算异质性了。
- 异质位点映射的reads的两种碱基的深度加起来应该与周围位点的深度相当。
- 异质位点映射的reads不全都很短（>100bp），双端测序的最好是成对映射。
- 异质位点周围的位点映射了同一read，且周围位点的mapping效果很好（单一碱基，均匀深度）。


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>