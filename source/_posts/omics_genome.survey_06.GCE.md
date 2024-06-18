---
title: 用k-mer分析进行基因组调查：（六）用GCE分步实现
date: 2022-06-07
categories:
- omics
- genome
- genome survey
tags:
- genome
- genome survey
- k-mer
- GCE
- kmerfreq

description: 介绍GCE，用GCE的kmerfreq做基因组调查(genome survey)的k-mer频数统计，GCE的gce做基因组特征评估。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283100&auto=1&height=32"></iframe></div>

**用k-mer分析进行基因组调查系列：**
- （一）基本原理：https://yanzhongsino.github.io/2022/05/25/omics_genome.survey_01.intro/
- （二）用Smudgeplot估计倍性：https://yanzhongsino.github.io/2022/12/31/omics_genome.survey_02.Smudgeplot/
- （三）用jellyfish进行k-mer频数统计：https://yanzhongsino.github.io/2022/05/27/omics_genome.survey_03.jellyfish/
- （四）用KMC进行k-mer频数统计：https://yanzhongsino.github.io/2022/06/05/omics_genome.survey_04.KMC/
- （五）用GenomeScope评估基因组特征：https://yanzhongsino.github.io/2022/06/05/omics_genome.survey_05.GenomeScope/
- （六）用GCE分步实现：https://yanzhongsino.github.io/2022/06/07/omics_genome.survey_06.GCE/
- （七）用KmerGenie一步实现：https://yanzhongsino.github.io/2022/06/19/omics_genome.survey_07.KmerGenie/

**【推荐】用Smudgeplot评估物种倍性后，用组合jellyfish+GenomeScope1.0做二倍体物种的基因组调查，用组合KMC+GenomeScope2.0做多倍体物种的基因组调查。**

# 1. k-mer进行基因组调查的软件
k-mer进行基因组调查分为**k-mer频数统计**和**基因组特征评估**两步。
- GCE可以分步实现两步。第一步k-mer频数统计和第二步基因组特征评估。
- GCE第一步的结果sample.histo可以用在GenomeScope和其他基因组特征评估的软件上，实现第二步。

# 2. GCE 简介
GCE (genomic charactor estimator)是华大基因在2013年开发的一款基于贝叶斯模型的用于基因组调查的软件，在2020年发布了版本v2。

# 3. GCE 安装
## 3.1. GCE 下载
- GCE主要托管在BGI的ftp站点：ftp://ftp.genomics.org.cn/pub/gce。
- 也可以在GCE github：https://github.com/fanagislab/GCE上找到。

## 3.2. GCE 安装
1. 已编译【推荐】
```
git clone git@github.com:fanagislab/GCE.git
```

2. 未编译
```
wget ftp://ftp.genomics.org.cn/pub/gce/gce-1.0.2.tar.gz
tar -xzvf gce-1.0.2.tar.gz #解压缩和解包
make #编译
```

文件夹下有`kmerfreq`和`gce`两个命令。

老版本是`kmer_freq_hash`代替`kmerfreq`命令。

# 4. GCE 运行
GCE软件里包含两个主要的命令，`kmerfreq`用来完成第一步k-mer频数统计，`gce`用来完成第二步基因组特征评估。
## 4.1. k-mer频数统计
### 4.1.1. 运行
1. 命令

`/path/gce-1.0.2/kmerfreq -k 17 -t 24 -p sample input.path &> kmer_freq.log`

2. 参数
- -k 17：k-mer size
- -t 24：线程
- -p prefix：输出文件的前缀
- input.path：输入数据的路径保存在input.path文本文件

### 4.1.2. 输出文件
1. **sample.kmer.freq.stat**：结果文件
- 结果文件sample.kmer.freq.stat，记录了每个k-mer频数的统计信息，用于生成GCE的输入文件。
- 之前的版本，该文件只统计到第255行，第255行之后的数据合并至第255行，表示k-mer出现频数>=255的片段总数。
- 现在这个版本(gce-1.0.2)上限变成了65534。

2. **kmer_freq.log**：运行日志文件
- 老版本的日志文件还对测序数据进行了简要统计。
- 在该文件的最下方，统计了k-mer片段总数、k-mer种类数、k-mer平均频数、碱基总数、reads平均长度、基因组大小的粗略估计等信息。

## 4.2. 基因组特征评估
### 4.2.1. 获取参数
1. 获取k-mer总数

`less sample.kmer.freq.stat | grep "#Kmer indivdual number"` 

用于gce的-g参数

2. 获取k-mer深度分布表

`less sample.kmer.freq.stat | perl -ne 'next if(/^#/ || /^\s/); print; ' | awk '{print $1"\t"$2}' > sample.kmer.freq.stat.2colum`

- sample.kmer.freq.stat.2colum文件包含两列数据，第一列是k-mer频数，第二列是频数对应的k-mer的种类数量。
- 预期第二列数据是泊松分布，有明显峰值；频数最小的(比如小于5)的那几列对应的k-mer数量高是测序错误造成的，频数最大的那列对应的k-mer数量高是把所有大于该列的频数进行合计数量造成的，两端都可以忽略。

### 4.2.2. 运行gce
1. 纯合模式

`gce -f sample.kmer.freq.stat.2colum -g 173854609857 > gce.table 2> gce.log`

2. 杂合模式

`gce -f sample.kmer.freq.stat.2colum -g 173854609857 -H 1 -c 75 > gce2.table 2> gce2.log`

3. 开发者建议

对于不能判断纯合还是杂合的数据，可以先运行纯合模式，获得初始峰值(raw_peak)，即k-mer的期望深度，用作`-c`参数。

然后再用`-c`参数和`-H 1`参数运行杂合模式。最后比较两种模式的结果，从而判断哪种更适用当前数据。

### 4.2.3. gce的参数
-f和-g是必需参数，其他都为可选参数。
- -f sample.freq.stat.2colum：k-mer频数分布表。
- -g 173854609857：k-mer片段总数，通过上面命令获取，或者查看kmer_freq.log获取。
- -H 1：默认是0。homozygous mode 纯合模式(0)，heterozgyous mode 杂合模式(1)。
- -c 75：独特的k-mer的期望深度，通过肉眼检查sample.gce.table的峰值获取。
- -b 1：有bias(1)，无(0)bias，默认是0。
- -m 1：评估模型，离散模型(0)，连续模型(1)，默认是0。
- -M 1500：设置最大深度，默认1500，大于1500的会被忽略。
- -D 1：期望值的精度，默认是1。

### 4.2.4. 结果
生成两个文件：**gce.table**和**gce.log**。gce.table保存了用于作图的数据，gce.log日志文件最后记录了基因组特征评估结果的统计。

1. gce.log文件最后记录了如下内容：

```
raw_peak        effective_kmer_species  effective_kmer_individuals      coverage_depth  genome_size     a[1]    b[1]
75      742400596       168346645871    75.8021 2.22087e+09     0.663012        0.271515
```

含义：
- raw_peak： 覆盖度为 75 的 kmer 的种类数最多，为主峰。
- effective_kmer_species：真实的k-mer种类的总数(去除测序错误造成的低频k-mers)
- effective_kmer_individuals：真实的k-mer个体的总数(去除测序错误造成的低频k-mers)
- coverage_depth：估算出的真实k-mers的覆盖深度
- genome_size：基因组大小。
**genome_size = effective_kmer_individuals / coverage_depth**
- a[1]： uniqe kmers (在基因组上仅出现 1 次的 kmer ) 的种类数占总种类数的比例。
- b[1]： uniqe kmers (在基因组上仅出现 1 次的 kmer ) 的个体数占总个体数的比例。该值代表着基因组上拷贝数为 1 的序列比例。


2. 如果使用杂合模式`-H 1`，则会在gce.log文件最后额外得到下面信息：
- a[1/2]：a[1/2]=0.223671表示在所有的 uniqe kmers 种类中，有 0.223671 比例的 kmer 属于杂合 kmer 。
- b[1/2]：a[1/2]=0.326934 表示在所有的 uniqe kmers 个体中，有 0.326934 比例的 kmer 属于杂合 kmer 。
通过计算，还可以获得的信息：
- k-mer种类的杂合率 kmer-species heterozygous ratio = 0.125918，0.125918 是由 a[1/2] 计算出来的。

$$0.125918 = a[1/2] / （ 2- a[1/2] )$$ 

- 重复序列的含量 = $$1 - b[1/2] - b[1]$$

# 5. references
1. GCE github：https://github.com/fanagislab/GCE
2. kmerfreq：https://github.com/fanagislab/kmerfreq
3. GCE paper：https://arxiv.org/abs/1308.2012
4. GCE blog：http://www.chenlianfu.com/?p=2335

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>