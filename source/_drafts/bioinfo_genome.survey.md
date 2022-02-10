---
title: genome survey
date: 2022-01-30 14:30:00
categories:
- bio
- bioinfo
tags:
-

description: 记录基因组调查的方法，利用k-mer分析来调查基因组大小，杂合度。
---

<div align="middle"><music URL></div>

## 基因组survey
在基因组大规模测序或者正式组装之前，首先构建DNA小片段文库进行中低深度的二代测序，使用PE文库测序所得的reads信息进行基因组Survey分析以初步评估基因组特征,包括基因组大小genome size，杂合度heterozygosity，重复序列比例，GC含量等。
* 基因组大小：基因组越大，测序花的钱越多
* 简单基因组: 杂合度低于0.5%, GC含量在35%~65%, 重复序列低于50%
* 二倍体普通基因组: 杂合度在0.5%~1.2%中间，重复序列低于50%；或杂合度低于0.5%，重复序列低于65%
* 高复杂基因组: 杂合度>1.2% 或 重复率大于65%

基因组survey分析核心是**k-mer分析。**

### k-mer分析前数据处理
#### 质控和过滤
对下机的低深度二代raw reads进行质控和过滤，去除质量低的序列和接头adapters序列，获得clean reads
```
$fastp -c -l 50 -w 8 -i sample_1.raw.fq.gz -I sample_2.raw.fq.gz -o sample_1.clean.fq.gz -O sample_2.clean.fq.gz -h sample_fastp.html -j sample_fastp.json
# 生成*_1.clean.fq.gz、*_2.clean.fq.gz结果文件，和*.html可视化QC报告文件, *.json JSON版报告文件
```
#### 去重
去除测序过程中（二代测序主要是PCR环节引入）带来的重复片段Duplication，通常重复片段占比<15%。
[Duplication占比问题的解释](http://blog.sciencenet.cn/blog-3406804-1215719.html)

*p.s. 去重一般操作比对到基因组上并排序完成的bam文件，利用基因组的位置信息进行去重，效率较高。若没有参考基因组的情况，比如这里的genome survey分析前去重，可以直接比对fq文件实现。*
```
$cat input_list.txt
path_to_sample_1.clean.fq
path_to_sample_2.clean.fq
$fastuniq -i input_list.txt -o sample_1.fastuniq.clean.fq -p sample_2.fastuniq.clean.fq
# 去重后生成sample_*.fastuniq.clean.fq两个文件
# fastuniq不支持压缩格式*.fq.gz输入文件
```
### k-mer分析
分为k-mer频数统计和基因组特征评估两步，软件KmerGenie一行命令同时实现两步，软件gce两行命令分别实现两步，jellyfish+genomescope两个软件分别实现两步。KmerGenie，gce和jellyfish软件第一步获取的频数分布表，都可用于genomescope和gce软件第二步骤的分析。

由于gce第一步骤支持的最大k-mer频数为255，大于255的数据被合并；而jellyfish统计到10000行，预估结果会更为准确。Genomescope对于高重复序列的基因组统计的基因组大小会偏小，建议max kmer coverage设置成10000。

建议使用jellyfish+genomescope/gce或者KmerGenie进行k-mer分析。

[k-mer分析介绍](http://blog.sciencenet.cn/blog-3406804-1162384.html)
#### k-mer频数统计
多个软件可以实现，目的是获取k-mer频数分布表。

k-mer长度一般选择17/21，另一个参数需注意和设定，单倍体模式还是杂合模式，可以两种模式都分析，查看差别。
1. [jellyfish](http://blog.sciencenet.cn/blog-3406804-1161522.html)
```
$jellyfish count -m 17 -s 100M -t 4 -o sample -C sample_1.fastuniq.clean.fq sample_2.fastuniq.clean.fq
# k-mer计数，当前工作路径下生成结果文件sample.jf，k-mer长度17，存储用的hash表大小为100M，线程4,-C对DNA正负链都进行统计，表示考虑DNA正义与反义链，遇到反义kmer时，计入正义kmer频数中。结果文件前缀名为sample。jellyfish不支持压缩格式*.fq.gz输入文件
$jellyfish histo sample.jf > sample.histo
# 统计 k-mer 计数得到 k-mer 频数分布表
```
值得注意的是，当输入PE双端测序数据时，需要-C参数，这样jellyfish只会统计一半数据量。

2. [KmerGenie](http://blog.sciencenet.cn/blog-3406804-1159967.html)
```
$cat fastq_list.txt
path_to_sample_1.fastuniq.clean.fq
path_to_sample_2.fastuniq.clean.fq
$kmergenie fastq_list.txt -o ./sample -l 17 -k 121 -s 10 -t 4 > sample.log1.txt 2> sample.log2.txt
# 默认单倍体模式，以k-mer长度17为起始，121为终止，10为间隔逐一测试；程序运行线程数4。结果输出在当前路径下，以sample为结果文件前缀名。“sample.log1.txt”和“sample.log2.txt”分别为程序运行时的正确/错误输出日志。
```
生成结果报告文件*_report.html，报告开头以折线图的形式展示出在每种长度k-mer下，估算的基因组大小，同时给出了“最佳k-mer”选择数值（其实就是将评估基因组总大小最高的那个k-mer值判定为“最佳k-mer”，为基因组组装时k-mer的选择提供参考）。

生成各k-mer取值下的频数分布表*.histo和对应的频数分布图*.histo.pdf，以及所有k-mer取值的总计*.dat和*.dat.pdf

3. [gce](http://blog.sciencenet.cn/blog-3406804-1161524.html)
```
$kmer_freq_hash -k 17 -L 150 -l fastq_list.txt -t 4 -o 0 -p sample &> sample.kmer.log
# k-mer长度为17，设定统计用的reads最大长度150bp，线程4，为节省时间不输出k-mer序列，结果文件前缀名sample
```
结果文件k-mer频数分布表“sample.freq.stat”和运行log文件“sample.kmer.log”
sample.freq.stat文件只统计到第255行，第255行之后的数据合并至第255行，表示k-mer出现频数>=255的片段总数。

sample.kmer.log为程序运行的日志文件，同时对测序数据进行了简要统计。该文件的最下方，统计了k-mer片段总数、k-mer种类数、k-mer平均频数、碱基总数、reads平均长度、基因组大小的粗略估计等信息。

频数分布表sample.histo/sample.kmer.freq.stat文件有两列，第一列是k-mer频数，第二列是频数对应的k-mer数量，预期是泊松分布，在k-mer频数中间有k-mer数量的峰值；频数小于5的那几列对应的k-mer数量高是测序错误造成的，频数最大的那列对应的k-mer数量高是把所有大于该列的频数进行合计数量造成的，两端都可以忽略。

#### 基因组特征评估
根据获取的频数分布表，进行基因组特征的评估，多个软件可以实现。

R脚本绘制k-mer频数分布曲线初步查看基因组特征
```R
$R
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

1. genomescope

使用[genomescope网页版](http://qb.cshl.edu/genomescope/)上传第一步获得的kmer频数分布表histo文件，设置参数Kmer length为第一步选择的k-mer长度值，这里是17；参数Read length为序列读长，一般为150；最后一个参数Max kmer coverage建议修改成10000，以统计更准确。

结果显示预估的基因组大小，杂合度，重复率等信息。

2. KmerGenie

生成结果报告文件*_report.html展示了在每种长度k-mer下，估算的基因组大小，同时给出了“最佳k-mer”选择数值，给组装基因组提供参考。

3. gce

峰值对应的k-mer频数(假设为n)。
```
gce -f sample.freq.stat -c n -b 1 -H 0 -m 1 -D 8 -M 256 > sample.gce.stat 2> sample.gce.log
# -c n频数峰值为n；-b 1数据有bias；-H 0单倍体模式；-m 1估算模型使用连续型；-D 8期望值精度；-M 256支持的最大k-mer频数，若输入jellyfish获得的sample.histo，则设置-M 10001。
```
结果文件sample.gce.stat和sample.gce.log，在log文件最后记录了基因组特征评估结果，包括估算的kmer平均深度cvg，基因组大小genome_size。