---
title: 用K-mer分析进行基因组调查(genome survey) —— GenomeScope
date: 2022-05-27
categories:
- omics
- genome
- genome survey
tags:
- genome
- genome survey
- K-mer
- GenomeScope

description: 介绍GenomeScope，用GenomeScope做基因组调查(genome survey)的基因组特征评估，基因组特征包括基因组大小、杂合度、重复率等信息。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283098&auto=1&height=32"></iframe></div>

# K-mer分析软件概况
K-mer分析分为**K-mer频数统计**和**基因组特征评估**两步。
1. jellyfish
jellyfish可以实现第一步K-mer频数统计。特点是使用Hash表存储数据，能多线程运行；速度快，内存消耗小。
2. GenomeScope
软件GenomeScope可以利用K-mer频数统计结果进行基因组特征评估。
3. KAT(The K-mer Analysis Toolkit)
软件KAT(The K-mer Analysis Toolkit)可以实现两步。包含多个工具来帮助用户通过使用k-mer对测序数据进行简单分析，如组装完整性、测序错误、是否有污染等。
4. gce
可以分别实现两步骤。
5. KmerGenie
软件KmerGenie可以同时实现两步。最大优点在于可以实现在多个预设K-mer下的自动分析，除了进行常规的k-mer频数统计之外，还能够基于不同k-mer自动计算基因组大小，并为基因组组装评估一个最佳组装k-mer数值作为备选。

# GenomeScope
通过jellyfish或其他软件分析得到K-mer频数分布表sample.histo后，可使用GenomeScope进行基因组特征评估。GenomeScope可获取的基因组特征包括基因组大小(genome size)，杂合度(heterozygosity)，重复序列比例，GC含量等。

GenomeScope有网页版和Linux本地版，功能一样。

## GenomeScope 网页版
### GenomeScope1.0 网页版【推荐】
[GenomeScope1.0 网页版](http://qb.cshl.edu/genomescope)：http://qb.cshl.edu/genomescope

1. 上传第一步获得的K-mer频数分布表histo文件；
2. 设置参数Kmer length为第一步选择的K-mer长度值，这里是17；
3. 参数Read length为序列读长，一般为150；
4. 最后一个参数Max kmer coverage建议修改成更大，比如10000，以统计更准确；或者设置成-1，代表不限制最大K-mer深度；
5. 提交后几分钟就可以得到结果，保存结果图片可用于发表。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/omics_genome.survey_GenomeScope1.0.png?raw=true" width=70% title="GenomeScope1.0结果示例" align=center/>

**<p align="center">Figure 1. GenomeScope1.0结果示例</p>**

### GenomeScope2.0 网页版
[GenomeScope2.0 网页版](http://qb.cshl.edu/genomescope/genomescope2.0)：http://qb.cshl.edu/genomescope/genomescope2.0

GenomeScope2.0与1.0的使用步骤很像，添加了参数倍性**Ploidy**，意味着可以评估多倍体的基因组特征。

1. 上传第一步获得的K-mer频数分布表histo文件；
2. 设置参数Kmer length为第一步选择的K-mer长度值，这里是17；
3. 参数倍性Ploidy根据物种的倍性设定，默认是二倍体，设置成2；
4. 参数Max k-mer coverage默认是-1，即不限制最大K-mer深度，可以根据物种情况调整；
5. 参数Average k-mer coverage for polyploid genome默认是-1，即不进行筛选，可以根据情况调整。
6. 提交后几分钟就可以得到结果，保存结果图片可用于发表。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/omics_genome.survey_GenomeScope2.0.png?raw=true" width=70% title="GenomeScope2.0结果示例" align=center/>

**<p align="center">Figure 2. GenomeScope2.0结果示例</p>**

## GenomeScope在线版


## GenomeScope实践经验
实际使用中发现，GenomeScope2.0常常估算不准。建议还是使用GenomeScope1.0。
- 在估算一个约300Mb的二倍体基因组时，GenomeScope1.0估算出来267Mb，GenomeScope2.0估算出来149Mb。
- 在估算一个约6Gb的四倍体基因组(6Gb)时发现，GenomeScope1.0估算出来5.5Gb，GenomeScope2.0估算出来2.7Gb。

# references
1. [GenomeScope github](https://github.com/schatzlab/genomescope)
2. [GenomeScope 2.0 github](https://github.com/tbenavi1/genomescope2.0)