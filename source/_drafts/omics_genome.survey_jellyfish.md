---
title: 用K-mer分析进行基因组调查(genome survey) —— jellyfish
date: 2022-05-25
categories:
- omics
- genome
- genome survey
tags:
- genome
- genome survey
- K-mer
- jellyfish

description: 用jellyfish做基因组调查(genome survey)。
---

<div align="middle"><music URL></div>

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

# jellyfish
## jellyfish 简介
jellyfish是Center for Bioinformatics and Computational Biology在2011年研发的一款对DNA的K-mers计数的软件，用Hash表储存数据，能多线程运行。

## jellyfish 安装
1. conda安装
- `conda install -c bioconda jellyfish` #安装的是v2.2.10
2. github安装
- 在[github：jellyfish](https://github.com/gmarcais/Jellyfish)上通过源码安装。

## jellyfish 运行
一般先用`jellyfish count`进行K-mer计数，然后用`jellyfish histo`对结果进行统计，获得K-mer的频数分布直方表sample.histo。

### count —— K-mer计数
1. 命令
`jellyfish count -m 17 -s 10G -t 12 -C -o sample.jf <(zcat sample_1.fq.gz) <(zcat sample_2.fq.gz)`

2. 参数
- sample_1.clean.fq sample_2.clean.fq：使用的PE reads，不支持压缩格式*.fq.gz输入文件，如果不解压缩，也可以用`<(zcat sample_1.fq.gz) <(zcat sample_2.fq.gz)`代替`sample_1.fq sample_2.fq`; 或者使用这种形式`zcat *fq.gz | jellyfish count /dev/fd/0`，其中`/dev/fd/0`是进程输入标志，代表管道前结果传递。
- -m 17: K-mer长度设置为17bp。如果基因组大小为G(单位是bp)，K-mer长度推荐设置成log(200*G)/log(4)。500Mbp的基因组对应约为17，1Gbp的19，10Gbp的21。
- -s 1000M：存储用的hash表大小为1000M，这个参数识别单位M(Mbp)和G(Gbp)。若该值不够大，则会生成多个hash文件，以数字区分文件名。最好设置的值大于总的独特的(distinct)k-mer数，这样生成的文件只有一个。如果基因组大小为G，每个reads有一个错误，总共有n条reads，则该值可以设置为[(G + n)/0.8]。
- -t 12：线程12
- -C：对DNA正负链都进行统计，表示考虑DNA正义与反义链，遇到反义kmer时，计入正义kmer频数中。如果是双端测序reads，需要这个参数。
- -o sample.jf：结果文件名为sample.jf，会生成K-mer计数文件sample.jf，是hash的二进制文件。
- c 7：K-mer的计数结果所占的最大比特数，默认支持的最大数字是2^7=128。该值最大，消耗内存越大。
- -out-counter-len=4：输出的二进制hash文件中的计数结果所占的字节数，一个字节是8比特，则默认支持的最大数字是2^32=4.3G。
- 不推荐用-Q，会将低质量的碱基替换成N。
- -L：不输出低于此值的K-mer
- -U：不输出高于此值的K-mer

3. 输出
- sample.jf：hash格式储存的K-mer频数文件

### histo —— 统计K-mer频率
`jellyfish histo -t 12 sample.jf > sample.histo`

统计K-mer计数(sample.jf)得到K-mer频数分布直方表(sample.histo)，包含空格分隔的两列数据，第一列代表k值出现的次数x(x=1,2,3...)，第二列是出现了x次的kmer的种类的数量y。sample.histo的两列即是kmer分布频率直方图的x和y轴的值。

参数：
- -t 12：线程12。
- -l 1：x的最小值，默认是1。结果会将小于此值的所有的k-mer的数目作为(x‐1)的值总结到一行。
- -h 10000：x的最大值，默认是10000。结果会将大于此值的所有的k-mer的数目作为(x+1)的值总结到一行。
- -i 1：x轴取值间隔，每隔该数值取值，默认为1。

### merge 合并【按需选择】
如果jellyfish count模块输出结果的二进制hash文件有多个，需要将多个hash文件合并，合并到merge.jf。

`jellyfish merge sample_hash1.jf sample_hash2.jf sample_hash3.jf -o merge.jf`

### stats 统计【可选】
`jellyfish stats sample.jf -o counts_stats.txt`

可以用stats模块来统计出k-mer总数（Total），特异的k-mer数目（Distinct），只出现过一次的k-mer数量（Unique），频数最高的k-mer数量（Max_count）等信息。

# 基因组特征评估
获得K-mer频数分布表sample.histo后，推荐用[GenomeScope1.0](http://qb.cshl.edu/genomescope)或者[GenomeScope2.0](http://qb.cshl.edu/genomescope/genomescope2.0/)或者GenomeScope的R脚本来做基因组特征评估和画图。

也可直接用R绘制sample.histo的频率分布直方图/频率分布曲线。

## [GenomeScope1.0 网页版](http://qb.cshl.edu/genomescope/)
1. 在[GenomeScope1.0 网页版](http://qb.cshl.edu/genomescope/)上传前一步获得的K-mer频数分布表sample.histo文件。
2. 设置参数K-mer length为第一步选择的K-mer长度值，这里是17；参数Read length为序列读长，一般为150；最后一个参数Max kmer coverage建议修改成更大的10000，以统计更多的K-mers。
3. 结果显示预估的基因组大小，杂合度，重复率等信息。

[GenomeScope2.0 网页版](http://qb.cshl.edu/genomescope/genomescope2.0)也是类似的步骤，多一个参数选择物种的倍型。实践经验发现GenomeScope1.0比起2.0估计的结果更接近实际C值。

## R绘制
R绘制K-mer频数分布曲线初步查看基因组特征

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

获得kmer_plot.png为频数分布曲线，可根据曲线峰值对基因组大小进行计算和预估。

# references
1. [jellyfish paper](https://academic.oup.com/bioinformatics/article/27/6/764/234905?login=true)
2. [jellyfish github](https://github.com/gmarcais/Jellyfish)
3. [GenomeScope github](https://github.com/schatzlab/genomescope)
4. [jellyfish参数推荐](https://www.bilibili.com/read/cv16360242)
5. [chenlianfu blog: jellyfish参数推荐](http://www.chenlianfu.com/?p=806)