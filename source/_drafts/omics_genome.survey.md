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

# 基因组调查(genome survey)
## 基因组调查
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
判断基因组复杂程度可以参考以下经验性标准：
- 简单基因组: 杂合度低于0.5%, GC含量在35%~65%, 重复序列低于50%。
- 二倍体普通基因组: 杂合度在0.5%~1.2%中间，重复序列低于50%；或杂合度低于0.5%，重复序列低于65%。
- 高复杂基因组: 杂合度>1.2% 或 重复序列占比大于65%。

# K-mer分析
## K-mer相关概念
1. monomeric unit (mer): 单体单元，单位是nt或者bp。通常用于双链核酸中的单位，100 mer DNA相当于每一条链有100nt，那么整条链就是100bp。
2. K-mer概念
在生物信息学中，K-mer是指包含在一段序列中的长度为k的子串。一段长度为L的核酸序列，以一个碱基为步长滑动，一共可以生成(L-K+1)个K-mers，另外还可以用这段核酸的反向互补序列再生成一次K-mer。

<img src="https://ask.qcloudimg.com/http-save/yehe-6581713/k82jlzu9rb.png?imageView2/2/w/1620" title="K-mer示例" width="90%"/>

**<p align="center">Figure 1. K-mer示例**
from [博客：k-mer与基因组组装](https://cloud.tencent.com/developer/article/1613847)</p>

## K-mer原理
### K-mer的优势
二代测序的准确率已达到99.9%，但测序量非常大时，错误碱基的绝对数量(比如10亿碱基里错误碱基数量会达到1000万个)还是会对分析有很大的影响。

由于测序错误具有随机性，通过将reads切割产生的K-mer中，测序错误生成的K-mer绝大多数都是测序物种中不存在的K-mer，因此都只出现1次(或很少的几次)，要是将这些低频的K-mer去掉，就有较大可能去除测序错误，从而使得分析(基因组调查，组装基因组)结果更可靠。

### K-mer用途
许多分析都会用到K-mer的处理方法，把测序得到的reads通过取K-mer后用于分析。比如评估基因组特征，组装基因组，物种样品污染评估等。评估基因组特征(genome survey)包括评估基因组大小(size)，杂合度，重复序列比例等。

1. 组装基因组
组装基因组使用K-mer的目的主要是去除低频率的K-mer以提高组装结果准确性。

2. 评估基因组大小(size)


3. 评估基因组杂合度

4. 评估基因组重复序列比例

5. 物种样品污染评估

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
1. 安装
- `conda install -c bioconda jellyfish` #安装的是v2.2.10
- 在[github：jellyfish](https://github.com/gmarcais/Jellyfish)上通过源码安装。

2. K-mer计数
`jellyfish count -m 17 -s 10G -t 12 -o sample -C sample_1.clean.fq sample_2.clean.fq`

参数：
- sample_1.clean.fq sample_2.clean.fq：使用的PE reads，不支持压缩格式*.fq.gz输入文件，如果不解压缩，也可以用`<(zcat sample_1.fq.gz) <(zcat sample_2.fq.gz)`代替。
- -m 17: K-mer长度设置为17bp
- -s 10G：存储用的hash表大小为10G
- -t 12：线程12
- -C：对DNA正负链都进行统计，表示考虑DNA正义与反义链，遇到反义kmer时，计入正义kmer频数中。如果是双端测序reads，需要这个参数。
- -o sample：结果文件前缀名为sample，会生成K-mer计数文件sample.jf，是hash的二进制文件。
- 不推荐用-Q，会将低质量的碱基替换成N。

3. K-mer频率
`jellyfish histo -t 12 sample.jf > sample.histo`

统计K-mer计数(sample.jf)得到K-mer频数分布直方表(sample.histo)，包含空格分隔的两列数据，第一列代表k值出现的次数x(x=1,2,3...)，第二列是出现了x次的kmer的种类的数量y。sample.histo的两列即是kmer分布频率直方图的x和y轴的值。

参数：
- -t 12：线程12。
- -l 1：x的最小值，默认是1。结果会将小于此值的所有的k-mer的数目作为(x‐1)的值总结到一行。
- -h 10000：x的最大值，默认是10000。结果会将大于此值的所有的k-mer的数目作为(x+1)的值总结到一行。
- -i 1：x轴取值间隔，每隔该数值取值，默认为1。

4. 统计【可选】
`jellyfish stats mer_counts.jf -o mer_counts_stats.txt`

可以用stats模块来统计出k-mer总数（Total），特异的k-mer数目（Distinct），只出现过一次的k-mer数（Unique），频数最高的k-mer数目（Max_count）等信息。

5. 画图
获得K-mer频数分布表sample.histo后，推荐用[GenomeScope2.0](http://qb.cshl.edu/genomescope/genomescope2.0/)或者[GenomeScope1.0](http://qb.cshl.edu/genomescope)或者GenomeScope的R脚本来做基因组特征评估和画图。也可直接用sample.histo绘制频率分布直方图/频率分布曲线。

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
### genomescope

使用[genomescope网页版](http://qb.cshl.edu/genomescope/)上传第一步获得的kmer频数分布表histo文件，设置参数Kmer length为第一步选择的K-mer长度值，这里是17；参数Read length为序列读长，一般为150；最后一个参数Max kmer coverage建议修改成10000，以统计更准确。

结果显示预估的基因组大小，杂合度，重复率等信息。

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