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



### gce


```
kmer_freq_hash -k 17 -L 150 -l fastq_list.txt -t 4 -o 0 -p sample &> sample.kmer.log

kmerfreq -k 17 -t 24 -p k17 path.txt
# K-mer长度为17，设定统计用的reads最大长度150bp，线程4，为节省时间不输出K-mer序列，结果文件前缀名sample
```

结果文件K-mer频数分布表“sample.freq.stat”和运行log文件“sample.kmer.log”
sample.freq.stat文件只统计到第255行，第255行之后的数据合并至第255行，表示K-mer出现频数>=255的片段总数。

sample.kmer.log为程序运行的日志文件，同时对测序数据进行了简要统计。该文件的最下方，统计了K-mer片段总数、K-mer种类数、K-mer平均频数、碱基总数、reads平均长度、基因组大小的粗略估计等信息。

频数分布表sample.histo/sample.kmer.freq.stat文件有两列，第一列是K-mer频数，第二列是频数对应的K-mer数量，预期是泊松分布，在K-mer频数中间有K-mer数量的峰值；频数小于5的那几列对应的K-mer数量高是测序错误造成的，频数最大的那列对应的K-mer数量高是把所有大于该列的频数进行合计数量造成的，两端都可以忽略。

3. gce

峰值对应的K-mer频数(假设为n)。
```
gce -f sample.freq.stat -c n -b 1 -H 0 -m 1 -D 8 -M 256 > sample.gce.stat 2> sample.gce.log
# -c n频数峰值为n；-b 1数据有bias；-H 0单倍体模式；-m 1估算模型使用连续型；-D 8期望值精度；-M 256支持的最大K-mer频数，若输入jellyfish获得的sample.histo，则设置-M 10001。
```
结果文件sample.gce.stat和sample.gce.log，在log文件最后记录了基因组特征评估结果，包括估算的kmer平均深度cvg，基因组大小genome_size。




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