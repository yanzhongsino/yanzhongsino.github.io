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



### KmerGenie
2. [KmerGenie](http://blog.sciencenet.cn/blog-3406804-1159967.html)
```
$cat fastq_list.txt
path_to_sample_1.fastuniq.clean.fq
path_to_sample_2.fastuniq.clean.fq
$kmergenie fastq_list.txt -o ./sample -l 17 -k 121 -s 10 -t 4 > sample.log1.txt 2> sample.log2.txt
# 默认单倍体模式，以K-mer长度17为起始，121为终止，10为间隔逐一测试；程序运行线程数4。结果输出在当前路径下，以sample为结果文件前缀名。“sample.log1.txt”和“sample.log2.txt”分别为程序运行时的正确/错误输出日志。
```
生成结果报告文件*_report.html，报告开头以折线图的形式展示出在每种长度K-mer下，估算的基因组大小，同时给出了“最佳K-mer”选择数值（其实就是将评估基因组总大小最高的那个K-mer值判定为“最佳K-mer”，为基因组组装时K-mer的选择提供参考）。

生成各K-mer取值下的频数分布表*.histo和对应的频数分布图*.histo.pdf，以及所有K-mer取值的总计*.dat和*.dat.pdf

2. KmerGenie

生成结果报告文件*_report.html展示了在每种长度K-mer下，估算的基因组大小，同时给出了“最佳K-mer”选择数值，给组装基因组提供参考。

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