---
title: 多序列比对(Multiple Sequence Alignment,MSA)
date: 2021-09-06 18:00:00
categories: 
- bioinfo
- MSA
tags:
- Multiple Sequence Alignment
- MSA
- sequence alignment
- trimming
- Clustal
- MUSCLE
- MAFFT
- PRANK
- Gblocks
- trimAl

description: 记录了多序列比对(Multiple Sequence Alignment,MSA)及比对后过滤(trimming)的软件。

---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

**写在前面**：

- 比对软件的选择：
	要是是基因组大数据的序列做比对，优先选择**MAFFT**（快速且兼顾准确）；要是少量序列比对，优先选择**PRANK**（准确）；要是极少数序列比对，哪个方便选哪个，一般**Clustal和MUSCLE**（方便）会整合到许多软件里，会更方便使用。
- 过滤软件的选择：
	目前暂时优先选**trimAl**。

update-20211006:今天发现PRANK这个软件比对过后序列的顺序会被打乱，后续处理要小心。

# 1. 序列比对(sequence alignment)
## 1.1. 概念
在生物信息学中，序列比对(sequence alignment)是指根据相似性排列DNA,RNA或者protein序列，通过在序列中插入空格(gap)来使得相似的碱基/氨基酸在相同列对齐的一种方法。

## 1.2. 方法
比对方法通常包括全局比对(global alignments)和局部比对(local alignments)。
1. 全局比对(global alignments)
全局比对强制要求比对跨越所有序列的整个长度，尽量对齐每条序列的每个碱基，当序列碱基和长度大致相似时最有用。通常全局比对用Needleman-Wunsch算法，是一种动态规划算法。
2. 局部比对(local alignments)
局部比对则确认通常有较大分歧的长序列内的相似性，有更多的计算相似性区域的额外的挑战。通常在较长序列包含相似序列基序的差异较大序列中更有用，常见的是Smith-Waterman算法，也是一种动态规划算法。
3. 混合比对(Hybrid methods)
全局和局部比对都使用的方法。

## 1.3. 类型
按照序列的数量分为成对比对(pairwise alignment)和多序列比对(multiple sequence alignment)。
1. 成对比对(pairwise alignment)
- 两条序列的比对。因为计算效率很高，通常不需要高精度方法。
- 常见的三种方法是点阵法(dot-matrix methods)，动态规划法(dynamic programming)和字节法(word methods)。
2. 多序列比对(multiple sequence alignment)
- 三条及以上序列的比对，是成对比对的扩展，算法上有些差异，以及更复杂。是这篇博客主要讲的内容。

# 2. 多序列比对(Multiple Sequence Alignment,MSA)
## 2.1. 概念
多序列比对(Multiple Sequence Alignment,MSA)是对三个以上的生物学序列（biological sequence），如蛋白质序列、DNA序列或RNA序列所作的序列比对。
目的是在不改变序列顺序的前提下，尽可能地把不同序列的相同碱基或者氨基酸排在同一列，并认为同一列的序列在进化上是同源的，有共同的祖先。

## 2.2. 用途
比对之后的序列，可用于进化学研究。比如，比较序列差异来推断序列进化历史（eg. 分化时间）和进化方式(突变/插入缺失/倒置等)。

常见的应用：
- 构建系统发育树：亲缘关系的分析。
- 比较基因功能：比较不同物种间的基因功能。
- 预测蛋白质结构，分类和注释蛋白和ncRNA：通过与已知功能的基因序列的相似性比对，进行基因功能的预测和注释。
- 序列拼接：通过比对，组装基因组。
- 突变分析：不同个体基因组间的差异；不同物种基因组间的共线性和结构变异。
- 检测适应性进化

## 2.3. 重要性
多序列比对的准确性与进化分析的准确性息息相关，不同比对软件和过滤软件的结果不同。

Blackburner和Whelan在2013年发表在MBE上的文章《Class of Multiple Sequence Alignment Algorithm Affects Genomic Analysis》对不同的比对算法和软件进行了比较分析，提出多序列比对MSA方法对下游分析至关重要，系统发育树和分支长度的推断高度依赖使用的多序列比对MSA类别，基于相似性的比对软件倾向于识别适应性进化（正选择）。

简单的对齐后过滤不能解决不同MSA方法间的差异。

# 3. 多序列比对(Multiple Sequence Alignment,MSA)的算法
多序列比对是双序列比对的扩展。
## 3.1. 动态规划算法(dynamic programming) —— 主要用于双序列比对
算法特点：

- 可以给出最优解。
- 将一个二维的动态规划矩阵扩展到三维或多维，即可用于多序列比对。
- 比对序列超过3个，需要的存储空间和运算时间都使得比对应用无法实现。
### 3.1.1. 基于全局比对的算法：Needleman-Wunsch算法
1. **全局比对**
	全局比对是指将参与比对的两条序列里面的所有字符进行比对。全局比对在全局范围内对两条序列进行比对打分，找出最佳比对，主要被用来寻找关系密切的序列。其可以用来鉴别或证明新序列与已知序列家族的同源性，是进行分子进化分析的重要前提。其代表是Needleman-Wunsch算法。

2. 全局比对算法特点：
- 打分矩阵：罚分分值不同，比对结果不同。
- 计算比对最高得分的算法：常用Needleman-Wunsch算法。
- 打印出最高得分相应的序列比对结果：根据得分矩阵回溯，找到最高得分结果。

### 3.1.2. 基于局部比对的算法：Smith-Waterman算法
1. **局部比对**
	与全局比对不同，局部比对不必对两个完整的序列进行比对，而是在每个序列中使用某些局部区域片段进行比对。其产生的需求在于、人们发现有的蛋白序列虽然在序列整体上表现出较大的差异性，但是在某些局部区域能独立的发挥相同的功能，序列相当保守。这时候依靠全局比对明显不能得到这些局部相似序列的。其次，在真核生物的基因中，内含子片段表现出了极大变异性，外显子区域却较为保守，这时候全局比对表现出了其局限性，无法找出这些局部相似性序列。其代表是Smith-Waterman局部比对算法。BLAST是局部比对的一种推广。

2. Smith-Waterman算法特点
- 是一种用来寻找并比较具有局部相似性区域的动态规划算法，适用于亲缘关系较远，整体上不具有相似性而在一些较小区域存在局部相似性的两条序列。
- 使用迭代方法计算出每个序列的相似分值，保存在一个得分矩阵M中，然后根据这个得分矩阵，通过动态规划的方法回溯找到最优的比对序列。

## 3.2. 启发式搜索算法(heuristic algorithm)：BWT算法和BLAST算法
一个基于直观或经验构造的算法，在可接受的花费下给出待解决组合优化问题每一个实例的一个可行解，该可行解与最优解的偏离程度一般不能被预计；目前绝大多数应用的比对软件都是启发式算法。

- 渐进式算法(progressive methods): Clustal W,MUSCLE,T-Coffee,DiAlign
- 迭代算法(iterative methods): MUSCLE,PRRP算法
- 星对比算法

## 3.3. 随机算法
对构造好的目标函数进行最优解搜索。
- 遗传算法：借鉴生物界进化规律演化来的全局意义上的自适应随机搜索方法。
- 模拟退火算法：用一物质系统的退火过程来模拟优化问题的寻优方法，当物质系统达到最小能量状态时，优化问题的目标函数也相应地达到了全局最优解。
- 粒子群算法

## 3.4. 其他算法
还有一些其他不常见的算法
1. 分治算法
2. 蚁群算法

# 4. 序列选择和比对前处理
一般来说，同源序列、同一家族的序列才会进行比对。

**通用的序列选择标准**：
- 如果是编码区序列，优先选蛋白序列。因为蛋白序列短且含有20种氨基酸信息，比DNA的4种核苷酸信息更多。
- 如果是数据库里选序列，尽量选择有详细注释的，可以在后续分析中提供更多信息。
- 多序列比对选用10-15条开始比对。如果比对结果不错，再加其他序列比对；如果比对结果不好，对现有序列处理（比如删除，剪辑等）。比对序列数量不是越多越好，多了可能增加比对软件出错概率。
- 如果有一条序列与半数以上其他序列一致性低于30%，比对会有问题。（一般氨基酸序列一致性在30-70%，E-value在10^-40-10^-5，仅供参考）。
- 如果有序列间一致性太高的，多序列比对也没什么价值，结果一直不错。
- 很多比对软件擅长比对总长度类似的序列，对长短不一的序列可以尝试比对前剪辑。
- 一般的比对软件对有重复片段的多序列比对时会有问题，尤其是序列间重复次数不同时问题更大，需要人工提取这部分手工比对。

# 5. 多序列比对软件
## 5.1. 比对软件概况
1. 基于相似性的比对(similarity-MSAMs)
- 经典渐进式比对(progressive alignment)软件：ClustalW
- 现代渐进式比对软件：DIALIGN-TX, MUSCLE
- 一致性比对软件(Consistency methods)：MUMMALS,ProbAlign,ProbCons,MAFFT,T-Coffee。
2. 基于进化关系的比对
- PRANK
- BAli-Phy
- StatAlign
3. 其他
- MACSE
- 基于隐马尔可夫模型(Profile HMM Methods)的比对：SEPP, TIPP, UPP, HIPPI

## 5.2. 基于相似性的比对(similarity-MSAMs)
### 5.2.1. Clustal(1994)
Clustal版本有X、W系列，目前最新的Omega表现最好。

基于渐进式比对(progressive alignment)，由引导树(guide tree)确定将序列添加到不断增长的MSA中的顺序，此算法通过成对的序列(sequence)-序列，序列-剖面(profile)，剖面-剖面的比对实现。

1. 算法流程
- 先进行序列两两比对，构建距离矩阵；
- 基于两两比对距离矩阵，通过计算构建系统进化引导树(guide tree)，对关系密切的序列进行加权；
- 从关系最近的两条序列开始，逐渐加入关系远的序列并不断重新构建多序列比对，直到所有序列都被加入为止。

2. 算法特点
- 是一种试探算法，渐进式比对算法都不能保证得到最优的比对。
- 比对准确性高度依赖开始的两两比对，适用于**亲缘关系较近**的序列。

### 5.2.2. MUSCLE(2004)
特点：
- 采用迭代方法进行比对运算，每一次最优化过程就是迭代过程，通过不断使用动态规划法重排来纠正错误，同时对这些亚类群进行比对以获得所有序列的全局比对。
- 相对Clustal，在比对速度和精度上都更优。
- 相对Clustal，添加了额外的步骤，包括改进起始树(starting tree)，重新选择树的分支和重新对齐任一剖面的改进步骤。

### 5.2.3. MAFFT(2002)
特点：
- 基于渐进式比对的算法，用快速傅里叶变换(fast Fourier transform)对序列进行聚类，再实现成对比对(pairwise alignments)；
- 之后的版本增加了其他算法和操作模式，包括快速、精确模式，非编码RNA序列比对及在现有比对中添加新的序列。
- 实现了两种算法，累进方法（FFT‐NS‐2）和迭代优化方法（FFT‐NS‐i）
- 比对精度比MUSCLE高，速度也快。

### 5.2.4. T-Coffee (2000)
与MAFFT功能类似，但包括更复杂的成对比对评分函数，MSA必须描述一组一致的成对比对。

## 5.3. 基于进化关系的比对 —— 统计比对(statistical alignment)
### 5.3.1. 统计比对的特点
- 统计比对(statistical alignment)是在进化学语境下描述序列的变化，能够确定各序列在与其共同祖先分化后发生的插入、替换和删除的过程，在系统发育树上明确建模。
- 可以根据序列的差异性灵活地使用不同地比对标准（评分矩阵与罚分），基于渐进式的比对无法做到
- 基于进化关系，准确性优于渐进式比对，但由于计算量大，很少使用在基因组数据上。

### 5.3.2. BAli-Phy(2005)
- 贝叶斯后验比对
- 利用马尔科夫链蒙特卡罗来探索给定分子序列数据的比对和系统发育的联合空间，同时估计消除了对不准确的比对引导树的偏差，在比对过程中采用了更复杂的替换模型，并自动利用共享插入/删除中的信息来帮助推断系统发育关系
- 准确度和PRANK差不多？

### 5.3.3. PRANK(2008)
特点：
- 针对DNA,密码子(codon)和氨基酸序列的概率多重比对软件（有密码子和氨基酸模式）；
- 可以重新构建祖先序列，有DNA翻译(DNA translation)和回译(back-translation)选项；
- 对完全统计对齐的启发式(heuristic)方法；
- 准确度高，但非常耗时，不适合基因组数据；

p.s.：PRANK比对过后序列顺序会被打乱，与比对前不一致，做后续处理时要小心。

### 5.3.4. StatAlign(2008)

## 5.4. MACSE
了解得不多，先摘录在这里。
- 第一个可以用于自动调整含有移码变异以及假基因的蛋白编码序列，而不破坏潜在密码子结构。
- 在核苷酸水平上对DNA序列进行比对，但有可能包括不是三个碱基倍数的间隙长度（即产生移码），同时基于其氨基酸翻译对产生的核苷酸比对进行评分。
- 这可以产生保留潜在密码子结构的核苷酸比对，同时受益于氨基酸序列的更高相似性。
- 已被用于基因组分析中
- OrthoMAM v8/v10数据库采用这个软件的分析流程

## 5.5. 基于隐马尔可夫模型(Profile HMM Methods)的比对
### 5.5.1. SEPP(SATe-enabled Phylogenetic Placement)
解决将short reads放进参考序列和树的系统发育问题
### 5.5.2. TIPP(Taxonomic Identification and Phylogenetic Profiling)
解决元组数据的分类识别和丰度分析问题
### 5.5.3. UPP(Ultra-large alignments using Phylogeny-aware Profiles)
解决非常大的数据集对齐的问题，这些数据集可能包括零碎数据，可以将数据集多达1,000,000条序列对齐；
### 5.5.4. HIPPI(Highly Accurate Protein Family Classificatio with Ensembles of HMMs)
解决蛋白质家族分类问题

# 6. Trimming - 质量过滤软件
在比对完成后，使用质量过滤软件过滤一些低质量以及高变异度的序列区域，仅保留进化保守的区域用于后续分析。

## 6.1. 产生低质量区域的原因
- 序列特征：远缘物种的序列在一些位置（例如第一个或最后一个外显子）有很大差异，这种真实存在的差异由于受到选择的影响，不适用于推断系统发育树（希望用保守区域推断系统发育关系）。
- 各种错误：包括测序错误、基因组组装错误、基因预测错误和多序列比对错误等。

## 6.2. 多序列比对后过滤软件
### 6.2.1. 常见过滤软件
1. **block-filtering**: trimAl,Gblocks
2. **segment filtering**: HmmCleaner,PREQUAL（基于隐马尔可夫模型的算法）
3. **sliding window analysis**: FasParser2

### 6.2.2. Gblocks(2000)
- 删除大片段非保守性或非同源性片段（6-10bp的非同源片段识别得不好），还对block(即一段连续不含Gap的列)的长度进行了限制。
- 不足：武断地规定了某个具体阈值来判断比对片段的保留或删除，对所有基因和所有片段一刀切式的处理，没有考虑不同片段的不同进化速率。

在线版本：http://www.phylogeny.fr/one_task.cgi?task_type=gblocks

### 6.2.3. trimAl(2009)
- 优点：速度快，准确度高。
- 比起Gblocks，可自动选择每个特定比对中的参数，包括Gap比例和氨基酸相似性水平，从而优化signal-to-noise ratio。

### 6.2.4. HmmCleaner
- 基于隐马尔可夫模型的新过滤算法。
- 只能过滤氨基酸序列。

### 6.2.5. PREQUAL
- 基于隐马尔可夫模型的新过滤算法。
- 针对非比对的蛋白编码序列及氨基酸序列。

### 6.2.6. FasParser2(2018)
- 有图形界面。
- 包括有效过滤序列比对质量模块（滑窗分析），可实现大规模检测序列中经历正选择的位点并清除。

# 7. references
1. https://en.wikipedia.org/wiki/Sequence_alignment
2. https://zh.wikipedia.org/zh-hans/%E5%A4%9A%E9%87%8D%E5%BA%8F%E5%88%97%E6%AF%94%E5%B0%8D
3. https://academic.oup.com/mbe/article/30/3/642/1038709?login=true
4. https://www.jianshu.com/p/430f21cce4f5
5. https://www.jianshu.com/p/31fb919f1c91
6. https://zhuanlan.zhihu.com/p/150579075

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>