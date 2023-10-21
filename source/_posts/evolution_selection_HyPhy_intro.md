---
title: 用HyPhy分析自然选择：（一）介绍HyPhy
date: 2023-10-20
categories: 
- bio
- evolution
- selection
tags:
- bioinfo
- biosoft
- selection
- HyPhy
- FEL
- SLAC
- FUBAR
- MEME
- FADE
- aBSREL
- BUSTED
- RELAX
- PAML
- codeML
description: 介绍自然选择分析软件HyPhy，它的版本，模型和使用。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=486310963&auto=1&height=32"></iframe></div>

# 1. HyPhy的简介
HyPhy的全称是Hypothesis Testing using Phylogenies，是一款近些年受到广泛使用和关注的一款用于选择压力分析的软件。

相较经典的选择分析软件PAML的codeML，HyPhy操作更简单一点，并且支持多线程运行，并使用了更新的模型，模型数量也更多。尤其是HyPhy的BUSTED模型，官方说比PAML的分支-位点模型（branch-site models）更强大和准确。

# 2. HyPhy的版本
HyPhy官网是https://www.hyphy.org/，有很多版本可以使用，包括：
1. 网页版（datamonkey）：https://www.datamonkey.org/，在网页(https://www.datamonkey.org/relax)中上传序列比对文件进行分析。最简单。
2. 命令行版本
- 标准命令行：https://www.hyphy.org/tutorials/CLI-tutorial/；这篇博客主要使用标准命令行版本。
- 交互式命令行：https://www.hyphy.org/tutorials/CL-prompt-tutorial/
3. 本地运行图形界面版本（GUI）：与网页版（datamonkey）操作类似。
4. 在Galaxy服务器上也可以使用HyPhy的许多方法：galaxy.hyphy.org
5. MEGA软件包正在整合HyPhy进界面，已可使用HyPhy部分功能
6. 还可以通过Python，R和其他程序语言编译HyPhy，并作为可访问的库使用。

# 3. HyPhy的分析库和常用检测
HyPhy的标准分析库共约有100种不同的方法，最常用于测试以下四种进化过程的假说：
1. 检测选择信号
2. 评估进化率
3. 比较不同的进化模型
4. 将自定义模型拟合到比对序列

# 4. HyPhy的模型
Hyphy的各种模型，基本上都可以分为不指定foreground和指定foreground运行的两种方式，前者对应的是检测pervasive (across the whole phylogeny) positive or purifying selection，即整个系统发育中的普遍的正选择/纯化选择，而后者对应的是检测episodic (at a subset of branches) positive or purifying selection，即检测一部分branches的独立正选择/纯化选择。

**HyPhy的模型和匹配的选择分析：**
## 4.1. 检测Pervasive selection的位点模型
检测个别位点是否受到普遍（整个系统发育树）的正选择或负选择，可选用以下模块：
1. FEL（Fixed Effects Likelihood，固定效应似然法）
- 适用于中小型数据集。
- 使用最大似然(ML)方法来推断每个位点上的非同义(dN)和同义(dS)替换率，用于给定的编码比对和相应的系统发育。
- 该方法假设在整个系统发育过程中，每个位点的选择压力是恒定的。
2. SLAC（Single-Likelihood Ancestor Counting，单似然祖先计数法）
- 是一种近似方法，准确度与 FEL 相似，但适用于较大的数据集。不过，SLAC 不适合高度分歧的序列。
- 对于给定的编码比对和相应的系统发育使用最大似然(ML)和计数方法的结合来推断每个位点上的非同义(dN)和同义(dS)替换率。
- 像FEL一样，该方法假设在整个系统发育过程中，每个位点的选择压力是恒定的。
3. FUBAR（Fast, Unconstrained Bayesian AppRoximation，快速无约束贝叶斯估计法）
- 适用于中型到大型数据集，与 FEL 相比，FUBAR 在检测位点的普遍选择方面具有效。FUBAR 是推断pervasive selection的首选方法。
- 使用贝叶斯方法来推断给定编码比对和相应系统发育的每个位点上的非同义(dN)和同义(dS)替换率。该方法假设在整个系统发育过程中，每个位点的选择压力是恒定的。
## 4.2. 检测episodic selection的位点模型
检测个别位点是否受到偶发性（在分支的子集）的正选择或负选择，可选用以下模块：
1. MEME（Mixed Effects Model of Evolution，进化混合效应模型）
- 采用混合效应最大似然方法来检验个别位点是否受到episodic positive或多样化选择的影响的假设。换句话说，MEME的目的是检测在一定比例的分支下正选择下进化的位点。
- 对于每个位点，MEME推测两种ω值，以及在给定的分支下，以此ω进化的概率。为了推断ω，MEME会推断α(dS) 和两个不同的β(dN),β−和β+。在空模型和备择模型中，MEME强制β−≤α。因此β+是空模型和备择模型不同的关键：在空模型中，β+被限制为≤α，但在备择模型中不受限制。最终，当β+>α时，位点被推断为正选择，并使用似然比检验显示显著。
- 需要注意的是，MEME 不接受先验分支规范（v2.3-dev 及更高版本将引入此功能）。
- MEME 是检测单个位点正选择的首选方法。
## 4.3. 检测蛋白序列比对的正选择/定向选择的位点模型
1. FADE(FUBAR Aproach to Directional Evolution)
- FADE是一种基于FUBAR引入的贝叶斯框架(Bayesian framework)的方法，用来测试蛋白质比对中的位点是否受定向选择的影响。
- 具体地说，FADE将系统地测试，对于比对中的每个位点，与背景分支相比，一组指定的前景分支是否显示对特定氨基酸的替代偏向。
- 该偏差参数的高值表明该位点对特定氨基酸的取代作用大大超过预期。使用贝叶斯因子(BF)评估FADE的统计显著性，其中BF>=100提供了强有力的证据，表明该位点正在定向选择下进化。
- 重要的是，与HyPhy中的大多数方法不同，FADE不使用可逆的马尔可夫模型，因为它的目标是检测定向选择。因此，FADE分析需要一个有根的系统发育。在使用FADE进行分析之前，可以使用基于浏览器的交互工具Phylotree.js（http://phylotree.hyphy.org/）来帮助建立树的根。
## 4.4. 检测独立分支的episodic selection(branch-site model,at a subset of sites)
检测个别分支是否受到偶发性（在部分位点）的正选择或负选择，可选用以下模块：
1. aBSREL（adaptive Branch-Site Random Effects Likelihood，自适应分支位点随机效应似然法）
- aBSREL是常见的“Branch-Site”模型的改进版。aBSREL 既可以先验地指定分支（即指定foreground branches）来检验选择，也可以探索性地检验每个世系的选择。（p-value将自动进行BH校正）
- aBSREL 是检测各个分支正选择的首选方法。
- 需要注意一点的是，aBSREL是多次独立对指定的每一支进行检验的，也就是说，你指定了许多的branches，实质上和多次指定不同一个branch来多次运行，效果是一样的，而并非将这些branches视为一个整体去做检测
## 4.5. 检测一个基因是否在某一分支或一组分支上的任何位点经历了正选择
检测一个基因是否在特定分支或部分分支的任何位点经历了正选择？
1. BUSTED（Branch-Site Unrestricted Statistical Test for Episodic Diversification，分支位点无限制统计检验）
- 通过在预先指定的系统树上测试一个基因是否在至少一个分支的至少一个位点上经历了正选择，BUSTED提供了一个全基因(非位点特异性)正选择的测试。当运行BUSTED时，用户可以指定一组前景支来测试正选择(其余分支被指定为“背景”)，或者用户可以测试整个系统发生的正选择。在后一种情况下，整个树被有效地视为前景，正选择的检验考虑整个系统发育。
- 这种方法对于相对较小的数据集(少于10个分类单元)特别有用，在这些数据集中，其他方法可能没有足够的功效来检测选择。
- 这种方法不适用于确定受正向选择影响的特定位点。
- 对于每个系统发育分区（前景和背景分支位点），BUSTED拟合了一个具有三个速率类的密码子模型，约束为ω1≤ω2≤1≤ω3。与其他方法一样，BUSTED同时估计每个分区属于每个ω类的位点的比例。这种模型作为选择检验中的替代模型，被称为无约束模型。然后，BUSTED通过比较这个模型与前景分支上ω3=1（即不允许正选择）的空模型的拟合度来测试正选择。这个零模型也被称为约束模型。如果零假设被拒绝，那么就有证据表明，至少有一个位点在前景枝上至少有一部分时间经历了正选择。重要的是，一个显著的结果并不意味着该基因是在整个前景的正选择下进化的。
## 4.6. 检测选择压力的放松/加强
全基因组的选择压力是否随着分支的某个子集而放松或加强？
1. RELAX
- RELAX检测选择压力与一组指定的 "测试 （TEST）"分支的放松（如净化选择变得不那么严格）或加强（如净化选择变得更强）。这种方法不适用于检测正选择。
- RELAX是一种假设检验框架，它检测自然选择的强度是否沿着一组指定的测试分支被放松或加强。因此，RELAX不是明确检测正选择的合适方法。相反，RELAX在识别特定基因上自然选择严格程度的趋势和/或变化方面最有用。
- RELAX需要一组指定的 "测试 "分支与第二组 "参考 "分支进行比较（注意，不必分配所有的分支，但测试集和参考集各需要一个分支）。RELAX首先对整个系统发育过程拟合一个具有三个ω类的密码子模型（空模型）。然后，RELAX通过引入作为选择强度参数的参数k（其中k≥0）作为推断ω值的指数：ωk来测试放松/强化选择。具体来说，RELAX固定推断的ω值（都是ωk<1,2,3>），并对测试分支推断出一个将比率修改为ωk<1,2,3>的k值（替代模型）。然后，RELAX进行似然比检验，比较替代模型和空模型。
- K>1表示选择强度加强，K<1表示选择压力放松。

## 4.7. 模型的选择
1. 基本选择
- 如果检测类似paml中的M8位点模型，最好用FUBAR，如果是小数据，则用FEL，大数据并且分歧度不是很高用SLAC。
- 如果检测某个前景支当中正选择位点，最好用MEME。
- 如果检测单独的某个branch是否存在正选择，最好用aBSREL。
- 如果检测一系列的branches的正选择，即检验你的这个基因，在指定的branches的任意一个位点是否在某段时间经历过正选择，则用BUSTED，BUSTED是不适合检测单独位点的正选择的。
- 如果检测选择压力的放松/加强，用RELAX。
- 如果用蛋白序列来检测氨基酸位点正选择/定向选择，用FADE。
2. 模型选择的其他考虑因素
- 几乎所有模型(还有一些没常用的模型没有提到)都可以分为指定前景和不指定前景的模式运行，但根据分析目的不同，会有最优选择。
- 当然也可以把某种模型都跑一遍，比如各种位点模型都走个流程。
- 还也可以结合paml的模型比较，例如，对于检测Pervasive selection的位点模型，你可以结合paml的M8、M2a来分析。对于检测episodic selection的branch-site，你可以结合paml的branch-site modelA和BUSTED/aBSREL来比较分析。

# 5. 标准命令行版本HyPhy的使用
## 5.1. 安装
`conda install -c bioconda hyphy`
## 5.2. 输入文件
HyPhy的各个模型的输入文件都是下面两个：
1. samples.aln：fasta或phylip格式的序列比对文件
2. tree.nw：newick格式的树文件
- 有些模块的分析可以指定前景支/测试支（Foreground/TEST），其余支是参考支(reference，默认，无需标记)。
- 标记方法：前景支/测试支(类比codeml的branch model的前景支)的分支名和支长（如果存在）之间标注{foreground}。
- 另外，HyPhy官网的phylotree工具（http://phylotree.hyphy.org/）可以在线对系统树进行标注。
- Pig支做前景支的标记实例：

```
((((Pig{FG}:0.147969,Cow:0.213430):0.085099,Horse:0.165787,Cat:0.264806):0.058611,((RhMonkey:0.002015,Baboon:0.003108):0.022733,(Human:0.004349,Chimp:0.000799):0.011873):0.101856):0.340802,Rat:0.050958,Mouse:0.097950);
```

## 5.3. 运行
HyPhy各个模型的运行都类似，修改busted为其他模型即可：

`nohup HYPHYMPI busted --alignment aln.fas --tree tree.nw --test FG --output busted.json > busted.log 2>&1 &`

## 5.4. 参数
1. --alignment：比对好的codon文件，可以是fas格式，也可以是phylip格式。
2. --tree：newick格式的树文件，如有test支标记在树文件里（标记方法见输入文件）
3. --branches或--test：指定树文件中标记的分支为前景支，RELAX模块用--test指定，其他模块用--branches指定。
4. --output：指定输出json文件名称，默认是输入比对文件加json后缀，如aln.fas.json。
5. 屏幕输出的是markdown格式，建议保存为relax.log文件

## 5.5. 多线程实现
1. 方法一(新版本hyphy可以自动根据需求采用多线程，所以线程也可以不设置。)
- `hyphy slac --alignment alignment.fasta --tree tree.newick --output slac.json`
2. 方法二(需要openmp支持)
`HYPHYMPI slac --alignment alignment.fasta --tree tree.newick CPU=interger`
3. 方法三
`mpiexec --oversubscribe -n 10（线程数) HYPHYMPI absrel --alignment alignment.fasta --tree tree.newick --branches Foreground`

## 5.6. 结果文件
1. busted.json：json格式，可以在网站http://vision.hyphy.org 上传这个结果文件进行可视化，直观展示。结果中site的数量代表的是密码子的数量，是碱基数量的1/3。
2. busted.log：markdown格式，直接查看方便。

## 5.7. 结果展示
1. json格式的结果具体的介绍可以参考https://www.hyphy.org/resources/json-fields.pdf。
2. 结果的可视化可以在网站 http://vision.hyphy.org/ 上上传json格式文件，然后交互式的展示结果。

# 6. reference
1. HyPhy官网：https://www.hyphy.org/
2. HyPhy的github：https://github.com/veg/hyphy-analyses/tree/master
3. 介绍HyPhy的博客：https://www.jianshu.com/p/2e8f7f7d545a

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>