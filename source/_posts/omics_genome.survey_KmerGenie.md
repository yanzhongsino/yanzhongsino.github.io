---
title: 用k-mer分析进行基因组调查：（六）用KmerGenie一步实现
date: 2022-06-19
categories:
- omics
- genome
- genome survey
tags:
- genome
- genome survey
- k-mer
- KmerGenie

description: 介绍KmerGenie，用KmerGenie做基因组调查(genome survey)的k-mer频数统计和基因组特征评估。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=27538254&auto=1&height=32"></iframe></div>

**【推荐】用Smudgeplot评估物种倍性后，用组合jellyfish+GenomeScope1.0做二倍体物种的基因组调查，用组合KMC+GenomeScope2.0做多倍体物种的基因组调查。**

# 1. k-mer进行基因组调查的软件
k-mer进行基因组调查分为**k-mer频数统计**和**基因组特征评估**两步。
- KmerGenie可以同时实现两步。第一步k-mer频数统计和第二步基因组特征评估。
- KmerGenie第一步的结果可用于其他软件第二步基因组特征评估。
- KmerGenie可以同时分析多个预设的k-mers，并选出一个最佳基因组组装k-mer值。

# 2. KmerGenie 简介
- KmerGenie在2014年第一次发表，2018年最近一次更新。开发用于基因组组装的参数k的最佳值的选择。
- KmerGenie官网：http://kmergenie.bx.psu.edu/。
- 官网包含软件的下载地址，示例报告，和版本更新记录。

# 3. KmerGenie 安装
在KmerGenie官网：http://kmergenie.bx.psu.edu/ 下载，目前最新版是18年更新的1.7051。

安装前需要python（>=2.7）和R支持，我用的anaconda的python，安装运行完成后自动把kmergenie命令添加到了`/anaconda3/bin/`下面，所以不用再次把kmergenie命令添加到环境变量了。

```
wget http://kmergenie.bx.psu.edu/kmergenie-1.7051.tar.gz
tar -xzvf kmergenie-1.7051.tar.gz
python setup.py install
kmergenie -h
```

# 4. KmerGenie 运行
1. 命令

`kmergenie fastq_list.txt -o ./sample -l 17 -k 121 -s 10 -t 4 > sample.log1.txt 2> sample.log2.txt`

- fastq_list.txt文件保存着fastq文件的位置和文件名，每个文件一行。
- 默认单倍体模式，以k-mer长度17为起始，121为终止，10为间隔逐一测试；程序运行线程数4。
- 结果输出在当前路径下，以sample为结果文件前缀名。
- “sample.log1.txt”和“sample.log2.txt”分别为程序运行时的正确/错误输出日志。

2. 参数
- --diploid：使用二倍体模式，默认是单倍体模式（haploid）。
- --one-pass：默认是两次评估（two passes），这个参数设置用来跳过在2bp分辨率上评估k的第二次评估。
- -k 121：最大的k-mer值，默认是121。
- -l 15：最小的k-mer值，默认是15。
- -s 10：在最小和最大的k-mer值间的间隔，默认是10。意味着会进行k=15,25,35...115,121的分析。
- -e 200：程序运行内存，默认是每个线程200MB。
- -t 8：线程数。
- -o histograms：输出文件的前缀，默认是histograms。
- --debug：开发者使用，输出R脚本。
- --orig-hist：老程序的评估方法（更慢且准确性更低）。

# 5. KmerGenie 结果
1. 结果报告文件sample_report.html

下载所有结果文件，打开sample_report.html，报告内容包括：

- 开头以折线图的形式展示出在每种长度k-mer下，估算的基因组大小。
- 同时给出了**最佳k-mer**选择数值。其实就是将评估基因组总大小**最高**的那个k-mer值判定为**最佳k-mer**，为基因组组装时k-mer的选择提供参考。
- 折线图的详细说明，包括最佳k-mer的评估规则，以及当测序深度足够高时的k-mer选择等。
- 每种k-mer的频数分布图，在基因组的k-mer中可根据该图判定基因组杂合度或重复序列比例。

2. 频数分布表sample.histo
- 包括各k-mer取值下的频数分布表sample.histo和对应的频数分布图sample.histo.pdf。
- 如果想用某个k-mer的频数分布表做**基因组特征评估**，自己绘制频数分布图，可以使用sample.histo文件。

3. 所有k-mer取值评估的基因组大小记录在sample.dat
- 包括sample.dat和sample.dat.pdf。

# 6. notes
1. 二倍体模式
- 如果待测物种是低杂合低重复的简单基因组，则使用**单倍体模式**。
- 如果是复杂基因组，使用**二倍体模式**。
- 如果不确定基因组简单还是复杂，可以先用单倍体模式运行，根据结果中是否有**明显杂合峰**判断，再运行二倍体模式。

2. KmerGenie软件默认将k-mer频数曲线的纵坐标进行了log10转化
可以通过修改脚本来更改展示效果：
- 在脚本`kmergenie-1.7051/scripts/plot_histogram.r`中第110行，`suppressWarnings`函数的参数`log='y'`设置的log10转化，可以通过去除`log='y'`参数来展示未log10转化的原始坐标。
- 在脚本`kmergenie-1.7051/scripts/plot_histogram.r`中第110行，`suppressWarnings`函数的参数`covNormalized`改为`covNormalized[-c(1:5)]`来过滤掉Abundance<5的区域。

3. k-mer取值
- KmerGenie软件是用于二代数据组装基因组推荐k-mer参数的。推荐的**最佳k-mer**是评估基因组最大的对应的k-mer。
- 在基因组组装时，**k-mer的取值**受测序深度的影响，若测序深度越高，可选择更高的k-mer进行尝试组装，以得到更长更完整的contigs序列。
- 但若在低深度测序模式下使用较高的k-mer进行组装时，就会引入较高的**错误率**。表现为k-mer频数分布曲线（纵坐标未进行log10转化的）的左侧由于测序错误导致的低频k-mer数量未随着k-mer频数升高下降至最低即产生了上升趋势。
- 用KmerGenie软件做基因组调查时，可以根据每个k-mer值的频数分布图结果选择更为合适的k-mer值做**基因组特征评估**。

# 7. references
1. KmerGenie website：http://kmergenie.bx.psu.edu/
2. KmerGenie paper：https://academic.oup.com/bioinformatics/article/30/1/31/235479
3. http://blog.sciencenet.cn/blog-3406804-1159967.html
4. https://www.jianshu.com/p/0251b55977c0
