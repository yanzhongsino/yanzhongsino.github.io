---
title: genome survey
date: 2022-05-15
categories:
- omics
- genome
tags:
- genome survey
- K-mer

description: 记录基因组调查(genome survey)的方法，利用K-mer分析来估计基因组大小，杂合度等基本信息。
---

<div align="middle"><music URL></div>

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

# GenomeScope
GenomeScope有网页版和Linux本地版，功能一样。
通过jellyfish或其他软件分析得到K-mer频数分布表sample.histo后，可使用GenomeScope进行基因组特征评估。GenomeScope可获取的基因组特征包括基因组大小(genome size)，杂合度(heterozygosity)，重复序列比例，GC含量等。

## GenomeScope 网页版
### GenomeScope1.0 网页版
[GenomeScope1.0 网页版](http://qb.cshl.edu/genomescope/)


上传第一步获得的kmer频数分布表histo文件，设置参数Kmer length为第一步选择的K-mer长度值，这里是17；参数Read length为序列读长，一般为150；最后一个参数Max kmer coverage建议修改成10000，以统计更准确。

结果显示预估的基因组大小，杂合度，重复率等信息。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/omics_genome.survey_GenomeScope1.0.png?raw=true" width=70% title="GenomeScope1.0结果示例" align=center/>

**<p align="center">Figure 1. GenomeScope1.0结果示例</p>**

### GenomeScope2.0 网页版
<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/omics_genome.survey_GenomeScope2.0.png?raw=true" width=70% title="GenomeScope2.0结果示例" align=center/>

**<p align="center">Figure 2. GenomeScope2.0结果示例</p>**

[GenomeScope2.0 网页版](http://qb.cshl.edu/genomescope/genomescope2.0)

实际使用中发现，GenomeScope2.0常常估算不准。建议还是使用GenomeScope1.0。

比如在估算一个约300Mb的二倍体基因组时，GenomeScope1.0估算出来267Mb，GenomeScope2.0估算出来149Mb。
在估算一个约6Gb的四倍体基因组(6Gb)时发现，GenomeScope1.0估算出来5.5Gb，GenomeScope2.0估算出来2.7Gb。

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
1. [GenomeScope github](https://github.com/schatzlab/genomescope)
2. [GenomeScope 2.0 github](https://github.com/tbenavi1/genomescope2.0)