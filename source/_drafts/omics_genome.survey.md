---
title: 用K-mer分析进行基因组调查(genome survey)
date: 2022-05-15
categories:
- omics
- genome
- genome survey
tags:
- genome
- genome survey
- K-mer

description: 记录基因组调查(genome survey)的概念和方法，K-mer分析的原理，利用K-mer分析来进行基因组调查（估计基因组大小，杂合度，重复序列占比等基本信息）。
---

<div align="middle"><music URL></div>

# 基因组调查(genome survey)
## 基因组调查
基因组调查(genome survey)指基因组特征评估，一般指通过K-mer分析二代测序数据，获得基因组大小(genome size)，杂合度(heterozygosity)，重复序列比例，GC含量等基因组信息的手段。
1. 基因组调查介绍
对于高等真核生物(特别是高等植物)来说，在进行基因组denovo测序和正式组装之前，首先构建DNA小片段文库进行中低深度的二代测序，使用测序所得的reads(通常是illumina的PE reads)进行基因组调查(genome survey)，来初步评估基因组特征，包括基因组大小(genome size)，杂合度(heterozygosity)，重复序列比例，GC含量等，从而为后续的基因组测序、组装和结构注释方案提供参考依据。

2. 基因组调查的目的
基因组调查主要目的是获取两个方面的信息，一个是基因组的大小，一个是基因组复杂程度。
- 基因组大小(genome size)
因为测序费用是以测序量为单价计算，所以基因组越大，测序费用越高。
- 基因组复杂程度
基因组越复杂(杂合度越高，重复序列占比越高)，意味着测序难度和组装难度越大。

3. 基因组调查实践
- 为了准确估计基因组信息，建议测序深度为50X，即预估基因组大小的50倍，最小也不低于25X。
- 预估基因组大小可以通过已有研究粗略判断，包括流式细胞研究，近缘种研究，也推荐植物在[C值数据库网站](https://cvalues.science.kew.org/)里查询。
- 基因组调查(genome survey)常常使用**K-mer分析**来实现。

## 基因组复杂程度
基因组复杂程序的判断标准包括：基因组大小，倍性，杂合度，重复序列比例，GC含量等。

一般而言，基因组越大，重复序列比例越高; GC含量异常低或异常高，重复序列比例也会很高；多倍体基因组的杂合度高于二倍体。

判断基因组复杂程度可以参考以下经验性标准：
- 简单基因组: 单倍体；或纯合二倍体；或杂合度低于0.5%, 且重复序列低于50%, 且GC含量在35%-65%的二倍体。
- 复杂基因组: 杂合度在0.5%~1.2%之间，或重复序列高于50%，或GC含量异常(<35%或>65%)的二倍体，或者多倍体。
- 高复杂基因组: 杂合度>1.2%；或重复序列占比大于65%。

# K-mer分析
K-mer分析可以用在生物信息学许多方面，这篇博客的K-mer分析特指用于基因组调查的K-mer分析方法。

## K-mer相关概念
1. monomeric unit (mer): 单体单元，单位是nt或者bp。通常用于双链核酸中的单位，100 mer DNA相当于每一条链有100nt，那么整条链就是100bp。
2. K-mer概念
在生物信息学中，K-mer是指包含在一段序列中的长度为k的子串。一段长度为L的核酸序列，以一个碱基为步长滑动，一共可以生成(L-K+1)个K-mers，另外还可以用这段核酸的反向互补序列再生成一次K-mer。

<img src="https://ask.qcloudimg.com/http-save/yehe-6581713/k82jlzu9rb.png?imageView2/2/w/1620" title="K-mer示例" width="90%"/>

**<p align="center">Figure 1. K-mer示例**
from [博客：k-mer与基因组组装](https://cloud.tencent.com/developer/article/1613847)</p>

## K-mer分析步骤

1. 通过切割二代测序的reads为K-mers
2. 统计K-mer的总数和每一种K-mer的频数
3. 统计K-mer的频数分布
4. 根据K-mer的频数分布的主峰峰值判定K-mer的期望深度(即主峰对应的K-mer频数)

## K-mer原理
K-mer的前提假设是测序是随机分布在基因组上的。

首先定义几个变量，方便原理解释：
- 基因组大小：G
- read读长：L
- 总reads条数：n
- K-mer长度：K

### 碱基深度分布
1. 单条read测序覆盖到某一个碱基的概率：L/G。
2. 因为L/G很小，n很大，每个碱基覆盖深度服从泊松分布。
3. 则每个碱基的覆盖深度的期望为：d=(L/G)*n。

### K-mer深度分布
1. 一个大小为G的基因组可以产生的K-mer种类约为G
- 假设一个基因组产生的K都是unique的，从一个大小为G的基因组可以得到(G-K+1)种不同的K-mer，
- 一般而言，基因组大小G在几百Mb或者Gb为单位，远大于K和1，所以K和1可以忽略不计，约等于G种K-mer。
2. 单条read测序完全覆盖某种K-mer的概率：(L-K+1)/G
- K-mer种类的总数约为G
- 单条read能产生的K-mer种类数量为(L-K+1)
- 假设read随机分布在基因组上，单条read测序完全覆盖某种K-mer的概率就为：(L-K+1)/G
3. 同样因为(L-K+1)/G很小，n很大，每种K-mer的覆盖深度服从泊松分布。
4. 每种K-mer的覆盖深度的期望为：D=((L-K+1)/G)*n
5. 由此可以得到，基因组大小为：G=(L-K+1)*n/D

### 通过K-mer分布估计基因组大小
1. 通过K-mer分析估计基因组大小时，计算分析可以得到总K-mer个数N和K-mer期望深度D。
- 对测序reads进行K-mer分割，获得的总K-mer个数N。
- 统计所有分割的K-mer，绘制频数分布图，期望是泊松分布，且分布峰值的K-mer频数即为K-mer的期望的覆盖深度D。

2. 通过下面公式可知，基因组大小：G=N/D。
- N=(L-K+1)*n
- D=((L-K+1)/G)*n

notes：
- K-mer的大小选择：K应该足够大到K-mer可以映射到基因组的唯一位置，但太大的K-mer会导致计算资源的过度使用。一般选21，17比较常见。
- K-mer分析适用于分析唯一主峰区域所占比例较大的基因组，当基因组杂合非常高或者重复序列比例非常大时，其影响可能导致无法通过K-mer分析正确估计基因组大小。
- 将K-mer深度等于1的情况认为是错误情况，计算错误率，并用于修正基因组大小。

## K-mer的优势
1. 增加准确率
- 二代测序的准确率已达到99.9%，但测序量非常大时，错误碱基的绝对数量(比如10亿碱基里错误碱基数量会达到1000万个)还是会对分析有很大的影响。
- 由于测序错误具有随机性，通过将reads切割产生的K-mer中，测序错误生成的K-mer绝大多数都是测序物种中不存在的K-mer，因此都只出现1次(或很少的几次)，要是将这些低频的K-mer去掉，就有较大可能去除测序错误，从而使得分析(基因组调查，组装基因组)结果更可靠。

## K-mer用途
许多分析都会用到K-mer的处理方法，把测序得到的reads通过取K-mer后用于分析。比如评估基因组特征，组装基因组，物种样品污染评估等。评估基因组特征(genome survey)包括评估基因组大小(size)，杂合度，重复序列比例等。

1. 组装基因组
组装基因组使用K-mer的目的主要是去除低频率的K-mer以提高组装结果准确性。

2. 评估基因组大小(size)
如果测序得到的reads能够覆盖整个基因组，那么从reads取出来的k-mer就能遍历整个基因组，因此我们就可以通过k-mer粗略计算该物种的基因组大小。

基因组大小G=全部的碱基数/平均每个碱基频数=T/μ

根据Lander Waterman的算法，基因组大小（G）= 总reads数 ÷ 期望测序深度（在不考虑测序错误、序列重复性的条件下，k-mer的深度分布遵循泊松分布，可以将深度分布曲线的峰值作为期望测序深度）

3. 评估基因组杂合度

4. 评估基因组重复序列比例
理论上单拷贝序列的k-mer，出现在主峰以外的概率非常低，所以我们取峰值的1.6倍后的k-mer为重复k-mer，得到重复k-mer的总数，从而算出重复序列的长度R

基因组中的单拷贝序列长度U=G-R

5. 物种样品污染评估

# K-mer分析软件概况
K-mer分析分为**K-mer频数统计**和**基因组特征评估**两步。
- 软件KmerGenie可以同时实现两步。
- 软件gce可以分别实现两步。
- 软件jellyfish可以实现第一步K-mer频数统计。
- 软件genomescope可以利用K-mer频数统计结果进行基因组特征评估。
- 软件KmerGenie，gce和jellyfish获取的频数分布表，都可用于软件genomescope和gce第二步骤的分析。

notes：
- 推荐使用**jellyfish+genomescope**进行K-mer分析。
- 由于gce第一步骤支持的最大K-mer频数为255，大于255的数据被合并；而jellyfish统计到10000行，预估结果会更为准确。
- Genomescope对于高重复序列的基因组统计的基因组大小会偏小，建议max kmer coverage设置成10000。
- K-mer长度常用17/21/25。

另一个参数需注意和设定，单倍体模式还是杂合模式，可以两种模式都分析，查看差别。

实践经验发现，K-mer值设置得越高，估计出来的基因组size会越大；

另外，在jellyfish里的jellyfish histo统计频数分布时，用参数-h 10000把统计上限调高，以及在GenomeScope阶段，Max kmer coverage设置的大一些(即统计进的kmer数量越多)，估算出来的基因组大小也会略大一些。

### R绘制
3. ggplot绘制

R脚本绘制K-mer频数分布曲线初步查看基因组特征
```R
#R 脚本示例
kmer <- read.table('sample.histo')
kmer <- subset(kmer, V1 >=5 & V1 <=500) #对频数范围5-500的数据进行绘制 
Frequency <- kmer$V1
Number <- kmer$V2
png('kmer_plot.png')
plot(Frequency, Number, type = 'l', col = 'blue')
dev.off()
```
获得kmer_plot.png为频数分布曲线，可查看曲线峰值对基因组大小进行计算和预估。


# references
1. https://en.wikipedia.org/wiki/K-mer
2. https://xuzhougeng.top/archives/genome-survey-using-kmers
3. [xuzhougeng's blog](https://www.jianshu.com/p/85de8f025899)
4. [jellyfish paper](https://academic.oup.com/bioinformatics/article/27/6/764/234905?login=true)
5. [jellyfish github](https://github.com/gmarcais/Jellyfish)
6. [GenomeScope github](https://github.com/schatzlab/genomescope)
7. [博客：k-mer与基因组组装](https://cloud.tencent.com/developer/article/1613847)
8. [K-mer分析和原理](https://www.bbsmax.com/A/lk5aQMxP51/)
9. [jellyfish参数推荐](https://www.bilibili.com/read/cv16360242)
10. [chenlianfu blog: jellyfish参数推荐](http://www.chenlianfu.com/?p=806)