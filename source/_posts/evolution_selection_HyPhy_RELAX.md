---
title: 用HyPhy分析自然选择：（二）用RELAX模型检测指定分支的选择是加强或放松
date: 2023-10-20
categories: 
- bio
- evolution
- selection
tags:
- bioinfo
- biosoft
- HyPhy
- RELAX
- selection
description: 介绍HyPhy软件，以及用HyPhy的RELAX模型检测自然选择在系统发育树上指定分支的变化趋势，通常用来检测负选择的放松或加强。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1934103823&auto=1&height=32"></iframe></div>

# 1. HyPhy的简介
HyPhy的介绍可以参考博客**用HyPhy分析自然选择：（一）介绍HyPhy**：https://yanzhongsino.github.io/2023/10/20/evolution_selection_HyPhy_intro

- HyPhy的全称是Hypothesis Testing using Phylogenies，是一款近些年受到广泛使用和关注的一款用于选择压力分析的软件。
- 相较经典的选择分析软件PAML的codeML，HyPhy操作更简单一点，并且支持多线程运行，并使用了更新的模型，模型数量也更多。

# 2. HyPhy的RELAX模型
## RELAX
1. HyPhy的RELAX模型是用来检测系统发育树上的指定分支是否经历了负选择的放松或加强的模型。
2. RELAX检测选择压力与一组指定的 "前景(foreground)/测试 （TEST）"分支的放松（正选择和负选择都变弱）或加强（正选择和负选择都变强）。RELAX方法不适用于检测正选择。
## RELAX原理
1. RELAX对放松选择的检验原理是基于放松选择对小于 1 的 ω 值（代表纯化选择）和大于 1 的 ω 值（代表正向选择）的不同影响。 当放松选择时，小于 1 的 ω 值会向 1 增大，而大于 1 的 ω 值会向 1 减小； ω 值的分布会向 1 集中。相反，当选择压力增强时， ω 值的分布会更加分散。
2. 在**分支位点模型**中（不同分支不同的位点有不同的 ω 值），放松选择的这种趋势可能有两种的影响： （1）推断出的选择类别的 ω 值可能趋向于 1，和/或 （2）属于不同类别的位点比例可能发生变化，从而使更多的位点被分配到 ω 值更接近于 1 的类别。这与PAML检测放松选择不同，PAML使用**分支模型**。

<img src="https://oup.silverchair-cdn.com/oup/backfile/Content_public/Journal/mbe/32/3/10.1093_molbev_msu400/2/msu400f1p.jpeg?Expires=1736163661&Signature=ohsqtggkLnzfBg93gTMDMkQyXGTWmqvmEQs-4cgfnzLpZbw862pt2c-fij-Y3F~VvPhfQBxxZI8DnHFGB66fe5EMkrGjNmkMYzuucTws5TwD1qRRWfKOTFkh~N7QAI6Z9RhCLxgKIkxDT2EKw2deJBIGBVZKYAe~6GtP8J5hFozscOWYRtGmJQsEa~Or21Q~2wfjwqtYMb61Rrl3QXADahYKQ0yz9bSayXd0Guh1bo0Cpei3cTN-szB4RzWVh~WlqKZAjjoU2lmHTrUumPWpc7N2JRQwnK-7PWUh0Y-zUVq1ZEfozlCv-8K3mlJUYjqe88jsFoUTTf7CB~6lxicD6Q__&Key-Pair-Id=APKAIE5G5CRDK6RD3PGA" title="强化选择和放松选择的模型效应" width="60%"/>

**<p align="center">Figure 1. 强化选择和放松选择的模型效应。强化选择使所有类别的 ω 远离中性（ω = 1），而放松选择使所有类别的 ω 向中性（ω = 1）靠拢。净化选择下的位点用蓝色表示，正选择下的位点用红色表示。 图源：[RELAX paper](https://academic.oup.com/mbe/article/32/3/820/981440)</p>**

3. RELAX需要一组指定的 "前景/测试"分支与第二组 "背景/参考"分支进行比较。RELAX首先对整个系统发育过程拟合一个具有三类 ω 的密码子模型（空模型）。然后，RELAX通过引入作为选择强度参数的参数 k（其中k ≥ 0）作为推断 ω 值的指数：ωk来测试放松/强化选择。具体来说，RELAX固定推断的ω值（都是ωk <1,2,3>），并对测试分支推断出一个将比率调整为ωk<1,2,3>的k值（替代模型）。然后，RELAX进行似然比检验（LRT），比较替代模型和空模型来检测结果的显著性。

# 3. HyPhy的RELAX模型检测选择放松或加强
## 3.1. 安装HyPhy
`conda install -c bioconda hyphy`
## 3.2. 输入文件
1. samples.aln：fasta或phylip格式的序列比对文件
2. tree.nw：newick格式的树文件
- RELAX模块的分析需要指定前景支（foreground）/测试支（TEST），其余支是参考支(reference，默认，无需标记)。
- 标记方法：前景支(类比codeml的branch model的前景支)的分支名和支长（如果存在）之间标注{FG}；如果要标记某一支及其包含的所有支，则需要标注多个{FG}。
- 可以用 HyPhy官网的phylotree在线工具（http://phylotree.hyphy.org/）对系统树进行标注。
- A支做前景支的标记实例：`((((A{FG}:0.14,B:0.15):0.23,C:0.69):0.47,D:0.26):0.11,E:0.20);`。
- A+B+C所在的所有支做前景支的标记实例：`((((A{FG}:0.14,B{FG}:0.15){FG}:0.23,C{FG}:0.69){FG}:0.47,D:0.26):0.11,E:0.20);`

## 3.3. 运行
`nohup HYPHYMPI relax --alignment aln.fas --tree tree.nw --test FG --output relax.json > relax.log 2>&1 &`

## 3.4. 参数
1. --alignment：比对好的codon文件，可以是fas格式，也可以是phylip格式。
2. --tree：newick格式的树文件，如有test支标记在树文件里（标记方法见输入文件）
3. --test：RELAX模块指定树文件中标记的分支为测试支
4. --output：指定输出json文件名称，默认是输入比对文件加json后缀，aln.fas.json
5. --rates 3： ω 值(dN/dS)的类别，可设置2-10类 ω 值，默认是3类。数字越大，运行越耗时。
6. 屏幕输出的是markdown格式，建议保存为relax.log文件。

## 3.5. 结果文件
1. relax.json：json格式，可以在网站http://vision.hyphy.org/RELAX 上传这个结果文件进行可视化，直观展示。结果中site的数量代表的是密码子的数量，是碱基数量的1/3。
2. relax.log：屏幕输出的markdown格式，直接查看方便。

## 3.6. 结果展示
1. 结果解释
- 主要看**K值**和LRT检验的**P值**。
- K>1表示选择强度加强（intensification），K<1表示选择放松（relaxation）。
- P值小于0.001表示结果显著（significant），否则不显著。
2. 结果可视化可以在网站 http://vision.hyphy.org/ 上上传json格式文件，然后交互式的展示结果。
- 结果中会根据K值显示放松或加强，并根据p值标注是否显著。
- json格式的结果具体的介绍可以参考https://www.hyphy.org/resources/json-fields.pdf。
3. 也可以在relax.log文件中搜索K值：**Relaxation/intensification parameter (K) = 1.05**和LRT检验的P值：**Likelihood ratio test p =   0.4564**。

ps：有些物种，不能简单的根据ω值或K值判断选择的放松或加强，有时ω值或K值的结果不一致，还要进一步看dN和dS来判断。

# 4. reference
1. HyPhy官网：https://www.hyphy.org/
2. HyPhy的github：https://github.com/veg/hyphy-analyses/tree/master
3. 介绍HyPhy的博客：https://www.jianshu.com/p/2e8f7f7d545a

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>