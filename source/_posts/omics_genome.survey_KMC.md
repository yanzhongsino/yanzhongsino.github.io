---
title: 用k-mer分析进行基因组调查(genome survey)：（三）用KMC进行k-mer频数统计
date: 2022-06-05
categories:
- omics
- genome
- genome survey
tags:
- genome
- genome survey
- k-mer
- KMC
- GenomeScope

description: 介绍KMC，用KMC做基因组调查(genome survey)的k-mer频数统计。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283101&auto=1&height=32"></iframe></div>

**【推荐】用Smudgeplot评估物种倍性后，用组合jellyfish+GenomeScope1.0做二倍体物种的基因组调查，用组合KMC+GenomeScope2.0做多倍体物种的基因组调查。**

# 1. k-mer进行基因组调查的软件概况
k-mer进行基因组调查分为**k-mer频数统计**和**基因组特征评估**两步。
- KMC可以实现第一步k-mer频数统计。
- KMC的结果sample.histo可以用在GenomeScope上，实现第二步基因组特征评估。

# 2. KMC 简介
- KMC是一个用来从FASTQ/FASTA文件中计算k-mers的基于KMC二进制数据库的程序。
- KMC是波兰的Silesian University of Technology的算法和软件学院的[REFRESH Bioinformatics Group](https://refresh-bio.github.io/)开发的工具。
- 2017年发布了第三个版本，KMC3。
- KMC是主要基于C语言的程序。

# 3. KMC 安装
1. 版本
有两个版本的KMC，一般使用第一个版本，Smudgeplot评估物种倍性时用到了第二个版本。
- 一个是[REFRESH Bioinformatics Group](https://refresh-bio.github.io/)的[refresh-bio/KMC](https://github.com/refresh-bio/KMC)。
- 一个是GenomeScope2.0的开发团队tbenavi1修改的[tbenavi1/KMC](https://github.com/tbenavi1/KMC)

2. 下载
在[KMC download](https://github.com/refresh-bio/KMC/releases)找对应系统的最新版本KMC软件，下载解压缩即可使用。

```
mkdir KMC && cd KMC
wget https://github.com/refresh-bio/KMC/releases/download/v3.2.1/KMC3.2.1.linux.tar.gz #下载最新版本的KMC
tar -xzf KMC3.2.1.linux.tar.gz #解压缩和解包，生成bin文件夹和include文件夹
```

3. 使用
解压缩后bin目录下会包含可执行文件，可直接使用，建议加入环境变量，包括：
- bin/kmc：计算k-mer频数的主程序
- bin/kmc_dump：在kmc生成数据库中列出k-mers的程序
- bin/kmc_tools：允许操作kmc数据库的程序

# 4. KMC 运行
用KMC计算k-mer频率，生成k-mer频数直方表和k-mer直方图。

1. 运行
```
mkdir tmp #创建临时文件夹
ls *.fastq.gz > FILES #用于分析的clean reads路径保存到文件FILES中
kmc -k21 -t16 -m64 -ci1 -cs10000 @FILES kmcdb tmp #计算k-mer频率
kmc_tools transform kmcdb histogram sample.histo -cx10000 #生成k-mer频数直方表sample.histo和k-mer直方图
```

2. kmc命令参数
- -k21：k-mer长度设置为21
- -t16：线程16
- -m64：内存64G，设置使用RAM的大致数量，范围1-1024。
- -ci1 -cs10000：统计k-mer coverages覆盖度范围在[1-10000]的。
- @FILES：保存了输入文件列表的文件名为FILES
- kmcdb：KMC数据库的输出文件名前缀
- tmp：临时目录

3. kmc_tools命令参数
- -cx10000：储存在直方图文件中counter的最大值。

4. 结果
生成的sample.histo可用于第二步GenomeScope的分析。

# 5. 基因组特征评估
获得k-mer频数分布表sample.histo后
- 推荐用[GenomeScope1.0](http://qb.cshl.edu/genomescope)或者[GenomeScope2.0](http://qb.cshl.edu/genomescope/genomescope2.0/)或者GenomeScope的R脚本来做基因组特征评估和画图。
- 也可直接用R绘制sample.histo的频率分布直方图/频率分布曲线。

## 5.1. GenomeScope 网页版
### 5.1.1. GenomeScope1.0 网页版 —— 适用于二倍体物种
1. 在[GenomeScope1.0 网页版](http://qb.cshl.edu/genomescope/)上传前一步获得的k-mer频数分布表sample.histo文件。
2. 设置参数k-mer length为第一步选择的k-mer长度值，这里是17；参数Read length为序列读长，一般为150；最后一个参数Max kmer coverage建议修改成更大的10000，以统计更多的k-mers。
3. 结果显示预估的基因组大小，杂合度，重复率等信息。

### 5.1.2. GenomeScope2.0 网页版 —— 适用于多倍体物种
[GenomeScope2.0 网页版](http://qb.cshl.edu/genomescope/genomescope2.0)也是类似的步骤。

## 5.2. R绘制
- R绘制k-mer频数分布曲线初步查看基因组特征。
- 获得kmer_plot.png为频数分布曲线，可根据曲线峰值对基因组大小进行计算和预估。

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

# 6. references
1. KMC3 paper：https://academic.oup.com/bioinformatics/article/33/17/2759/3796399
2. refresh-bio/KMC：https://github.com/refresh-bio/KMC
3. tbenavi1/KMC github：https://github.com/tbenavi1/KMC