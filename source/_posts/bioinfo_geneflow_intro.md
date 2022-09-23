---
title: 基因流及其推断
date: 2022-04-10
categories: 
- bio
- bioinfo
- gene flow
tags: 
- gene flow
- hybridization
- introgressive
- population networks
- Dsuite
- PhyloNetworks
- TreeMix
- 3s
description: 介绍了基因流相关概念，推断基因流常用软件Dsuite，PhyloNetworks，TreeMix，3s等。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=210766&auto=1&height=32"></iframe></div>

# 1. 基因流(gene flow)/杂交(hybridization)/渐渗(introgressive)
基因流，杂交和渐渗通常一起讨论，有些情况下甚至三者在说同一件事。从定义判断，基因流通常发生在种内群体间，杂交则是发生在种间，渐渗是指杂交加回交产生的一种现象。
可以这样理解，杂交和渐渗都是基因流的具体结果。
1. 基因流(gene flow)
- 基因流是指遗传物质在不同群体间的流动。
- 造成基因流动的原因可能是个体或配子(例如花粉)在群体间的迁徙，或者不同群体间个体的交配等。
- 基因流可能发生在同一物种的不同群体间，也可能发生在不同物种间。
2. 杂交(hybridization)
- 杂交指不同物种间通过有性生殖实现配子融合形成下一代的过程。
3. 渐渗(introgressive)
- 渐渗是指通过种间杂种与亲本物种之一的反复回交，将遗传物质从一个物种转移到另一个物种的基因库中，是一个长期的过程。

# 2. 推断基因流/杂交
通常在物种内检测不同地区的群体间是否存在基因流，也可以在物种间检测基因流来判断杂交/渐渗，物种间的基因流会导致系统发育树的不稳定或核质冲突等问题，所以可以推断系统发育网络来检测所有物种对的基因流。

## 2.1. 推断基因流的软件
1. 通过计算Patterson's D值(ABBA-BABA值)和相关统计量来判断基因流：Dsuite(2020),ADMIXTOOLS(2012),HyDe(2018),ANGSD(2011,2018),POPGENOME(2014,2019),COMP-D(2020)。
2. 推断系统发育网络：PhyloNetworks(2017),PhyloNet(2008,2018),TreeMix(2012),BEAST2(2017)。
3. 基于最大似然法：3s(2017)
4. 基于MCMC算法的：IM, IMA

有几个软件单独写了博客：

### 2.1.1. Dsuite【推荐】
[Dsuite blog](https://yanzhongsino.github.io/2022/04/10/bioinfo_geneflow_Dsuite/)

1. Dsuite简介
   - Dsuite是通过计算Patterson's D统计量(即ABBA统计量)和f4等统计量来评估种群间或近缘种间基因流的基于C语言的软件。
2. Dsuite 原理
   - D值（即ABBA统计量）和f4-ratio统计可以表示为适用于四个分类群的双等位基因SNP：P1,P2,P3,O，拓扑是 (((P1,P2),P3),O)。
   - 其中外类群O携带祖先等位基因A，衍生等位基因用B表示。BBAA,ABBA,BABA分别代表四个分类群携带等位的三种模式。
   - 在没有基因流的零假设下，由于具有相同频率的不完全谱系分类，预计P3与P1或P2共享衍生等位基因B的两种模式ABBA和BABA的频率相等，如果ABBA和BABA的频率有显著差异则代表在P3和P1或P2间存在基因渐渗。
   - D=(nABBA-nBABA)/(nABBA+nBABA)；在外群对于祖先等位基因A是固定的（外群中B的频率为0）假设下，D统计量是等位基因模式计数的归一化差异。
   - 如果外群中衍生等位基因B不为0，则Dsuite的D值是Patterson's D，适用于无根的四分类群树。
3. Dsuite输入输出
   - 输入：基因组snp的vcf格式文件，居群树文件(可选optional)
   - 输出：D值统计，f4-ratio统计，f-branch统计，f-branch树矩阵热图
4. Dsuite优势和不足
   - Dsuite的优势是运行非常快(时间以小时计算)
   - 不足是Dsuite分析结果不包含基因流的方向
5. Dsuite适用范围
   - Dsuite适用于基因组学大数据和多样本(超过十个)数据
   - 适用于居群间或物种间的基因流推测
   - 即使每个群体只有一个个体也可以推测基因流
   - 还可以计算pool-seq数据的基因流
   - 相较其他计算D值软件，Dsuite还同时可以计算f4-ratio和f-branch，以及滑窗统计f相关值。

### 2.1.2. PhyloNetworks
[PhyloNetworks blog](https://yanzhongsino.github.io/2022/04/14/bioinfo_geneflow_PhyloNetworks/)
1. PhyloNetworks简介
   - PhyloNetworks是通过基因树或多位点序列(SNaQ)的最大伪似然进行推断系统发育网络的一个Julia包。
2. PhyloNetworks原理
   - 原理：通过SNaQ来实现网络推断，SNaQ通过估计4分类群子集的最大伪似然来加速运算，估计的网络不受根的影响。
3. PhyloNetworks输入输出
   - 输入：newick格式基因树(多个基因树组成的文件)
   - 输出：系统发育网络，基因流方向和杂交节点贡献比例
4. PhyloNetworks优势和不足
   - 推断系统发育网络，包括基因流的方向和强度。
   - 相较于其他推断系统发育网络的软件，PhyloNetworks集成了上游分析，网络估计，引导分析，下游特征进化分析，绘图等功能。
   - 不足是运行多样本(超过十个个体)和数据量大(超过1000个)会非常耗时(常常以星期/月计时)。
5. PhyloNetworks适用范围
   - PhyloNetworks适用于基因树数据
   - 适用于居群间或物种间的基因流推测
   - 适用于推断基因流方向和强度

### 2.1.3. TreeMix
[TreeMix blog](https://yanzhongsino.github.io/2022/03/20/bioinfo_geneflow_treemix/)

1. TreeMix简介
   - TreeMix利用等位基因频率来推断群体间分化和杂合（基因流动或基因渗入）
2. TreeMix输入输出
   - 输入：基因组snp的vcf文件，和居群系统树(可选optional)
   - 输出：最佳杂交次数和系统发育网络(包含杂交方向和强度)
3. TreeMix优势和不足
   - TreeMix和PhyloNetworks一样，也是推断系统发育网络。
   - 我自己用时，有些PhyloNetworks报错无法定根和边缘错误的情况TreeMix可以找到最佳杂交次数。
   - 不足是比PhyloNetworks更耗时，超级耗时。

### 2.1.4. 3s
[3s blog](https://yanzhongsino.github.io/2022/09/22/bioinfo_geneflow_3s/)

1. 3s简介
- 3s利用似然率来推断两个物种/群体间的基因流方向和强度
2. 3s输入
- 输入：基因组或其他测序序列phylip文件
- 输出：基因流方向和强度
3. 3s优势和不足
- 随着数据量线性增加运算时间，运算快，适合基因组数据。
- 一次只能检测三个物种/群体，无法建立系统发育网。

# 3. reference
1. wiki: gene flow：https://en.wikipedia.org/wiki/Gene_flow
2. wiki: introgression：https://en.wikipedia.org/wiki/Introgression
3. Dsuite paper：https://onlinelibrary.wiley.com/doi/10.1111/1755-0998.13265
4. PhyloNetworks paper：https://academic.oup.com/mbe/article/34/12/3292/4103410
5. TreeMix paper：https://www.nature.com/articles/npre.2012.6956.1

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>