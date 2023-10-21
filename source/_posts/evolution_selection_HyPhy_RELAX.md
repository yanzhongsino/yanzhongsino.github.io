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
- HyPhy的RELAX模型是用来检测系统发育树上的指定分支是否经历了负选择的放松或加强的模型。
- RELAX检测选择压力与一组指定的 "测试 （TEST）"分支的放松（负选择变得不那么严格）或加强（负选择变得更强）。这种方法不适用于检测正选择。
- RELAX是一种假设检验框架，它检测自然选择的强度是否沿着一组指定的测试分支被放松或加强。因此，RELAX不是明确检测正选择的合适方法。相反，RELAX在识别特定基因上自然选择严格程度的趋势和/或变化方面最有用。
- RELAX需要一组指定的 "测试 "分支与第二组 "参考 "分支进行比较（注意，不必分配所有的分支，但测试集和参考集各需要一个分支）。RELAX首先对整个系统发育过程拟合一个具有三个ω类的密码子模型（空模型）。然后，RELAX通过引入作为选择强度参数的参数k（其中k≥0）作为推断ω值的指数：ωk来测试放松/强化选择。具体来说，RELAX固定推断的ω值（都是ωk<1,2,3>），并对测试分支推断出一个将比率修改为ωk<1,2,3>的k值（替代模型）。然后，RELAX进行似然比检验，比较替代模型和空模型。

# 3. HyPhy的RELAX模型检测选择放松或加强
## 3.1. 安装HyPhy
`conda install -c bioconda hyphy`
## 3.2. 输入文件
1. samples.aln：fasta或phylip格式的序列比对文件
2. tree.nw：newick格式的树文件
- RELAX模块的分析需要指定前景支/测试支（TEST），其余支是参考支(reference，默认，无需标记)。
- 标记方法：测试支(类比codeml的branch model的前景支)的分支名和支长（如果存在）之间标注{foreground}。
- 另外，HyPhy官网的phylotree工具（http://phylotree.hyphy.org/）可以在线对系统树进行标注。
- Pig支做前景支的标记实例：

```
((((Pig{FG}:0.147969,Cow:0.213430):0.085099,Horse:0.165787,Cat:0.264806):0.058611,((RhMonkey:0.002015,Baboon:0.003108):0.022733,(Human:0.004349,Chimp:0.000799):0.011873):0.101856):0.340802,Rat:0.050958,Mouse:0.097950);
```

## 3.3. 运行
`nohup HYPHYMPI relax --alignment aln.fas --tree tree.nw --test FG --output relax.json > relax.log 2>&1 &`

## 3.4. 参数
1. --alignment：比对好的codon文件，可以是fas格式，也可以是phylip格式。
2. --tree：newick格式的树文件，如有test支标记在树文件里（标记方法见输入文件）
3. --test：RELAX模块指定树文件中标记的分支为测试支
4. --output：指定输出json文件名称，默认是输入比对文件加json后缀，aln.fas.json
5. 屏幕输出的是markdown格式，建议保存为relax.log文件

## 3.5. 结果文件
1. relax.json：json格式，可以在网站http://vision.hyphy.org/RELAX 上传这个结果文件进行可视化，直观展示。结果中site的数量代表的是密码子的数量，是碱基数量的1/3。
2. relax.log：markdown格式，直接查看方便。

## 3.6. 结果展示
1. json格式的结果具体的介绍可以参考https://www.hyphy.org/resources/json-fields.pdf。
2. 结果的可视化可以在网站 http://vision.hyphy.org/ 上上传json格式文件，然后交互式的展示结果。
- 结果中会根据K值显示放松（relaxation）或加强（intensification），并根据p值标注是否显著（significant）
- K>1表示选择强度加强，K<1表示选择压力放松。

ps：有些物种，不能简单的根据ω值或K值判断选择的放松或加强，还要进一步看dN和dS来判断。

# 4. reference
1. HyPhy官网：https://www.hyphy.org/
2. HyPhy的github：https://github.com/veg/hyphy-analyses/tree/master
3. 介绍HyPhy的博客：https://www.jianshu.com/p/2e8f7f7d545a

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>