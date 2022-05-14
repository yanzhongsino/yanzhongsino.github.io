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


# K-mer分析软件
## K-mer分析软件概况
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

## K-mer频数统计
### jellyfish
jellyfish是Center for Bioinformatics and Computational Biology在2011年研发的一款对DNA的K-mers计数的软件，用Hash表储存数据，能多线程运行。
1. 安装
- `conda install -c bioconda jellyfish` #安装的是v2.2.10
- 在[github：jellyfish](https://github.com/gmarcais/Jellyfish)上通过源码安装。

2. K-mer计数
`jellyfish count -m 17 -s 10G -t 12 -o sample.jf -C <(zcat sample_1.fq.gz) <(zcat sample_2.fq.gz)`

参数：
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

3. 合并【按需选择】
`jellyfish merge sample_hash1 sample_hash2 sample_hash3 -o merge.jf`

如果jellyfish count模块输出结果的二进制hash文件有多个，需要将多个hash文件合并，合并到merge.jf。

4. 统计【可选】
`jellyfish stats sample.jf -o counts_stats.txt`

可以用stats模块来统计出k-mer总数（Total），特异的k-mer数目（Distinct），只出现过一次的k-mer数量（Unique），频数最高的k-mer数量（Max_count）等信息。

5. K-mer频率
`jellyfish histo -t 12 sample.jf > sample.histo`

统计K-mer计数(sample.jf)得到K-mer频数分布直方表(sample.histo)，包含空格分隔的两列数据，第一列代表k值出现的次数x(x=1,2,3...)，第二列是出现了x次的kmer的种类的数量y。sample.histo的两列即是kmer分布频率直方图的x和y轴的值。

参数：
- -t 12：线程12。
- -l 1：x的最小值，默认是1。结果会将小于此值的所有的k-mer的数目作为(x‐1)的值总结到一行。
- -h 10000：x的最大值，默认是10000。结果会将大于此值的所有的k-mer的数目作为(x+1)的值总结到一行。
- -i 1：x轴取值间隔，每隔该数值取值，默认为1。

6. 画图
获得K-mer频数分布表sample.histo后，推荐用[GenomeScope2.0](http://qb.cshl.edu/genomescope/genomescope2.0/)或者[GenomeScope1.0](http://qb.cshl.edu/genomescope)或者GenomeScope的R脚本来做基因组特征评估和画图。也可直接用sample.histo绘制频率分布直方图/频率分布曲线。






## 基因组特征评估
### GenomeScope
GenomeScope有网页版和Linux本地版。网页版又分为1.0和2.0两个版本。

#### GenomeScope1.0 网页版
使用[GenomeScope1.0 网页版](http://qb.cshl.edu/genomescope/)上传第一步获得的kmer频数分布表histo文件，设置参数Kmer length为第一步选择的K-mer长度值，这里是17；参数Read length为序列读长，一般为150；最后一个参数Max kmer coverage建议修改成10000，以统计更准确。

结果显示预估的基因组大小，杂合度，重复率等信息。

[GenomeScope2.0 网页版](http://qb.cshl.edu/genomescope/genomescope2.0)


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