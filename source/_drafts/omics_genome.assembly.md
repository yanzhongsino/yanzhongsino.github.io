---
title: genome assembly
date: 2024-06-19
categories: 
- omics
- genome
- genome assembly
tags:
- genome assembly

description: 基于三代和二代测序数据对非模式生物进行基因组的从头组装
---

<div align="middle"><music URL></div>

# 基因组组装


# pacbio
三代下机数据是subreads.bam,subreads.bam.pbi,subreads.xml三个文件。

subreads.bam与reads比对到参考基因组上生成的bam文件格式一致，但内容有差异。

1. bam2fasta
- conda安装
```
conda install -c hcc smrtlink-tools #安装PacBio官方软件SMRTlink
bam2fasta subreads.bam -o sample #bam2fasta转换成sample.fasta
```

- 下载安装
https://www.pacb.com/support/software-downloads/ 下载最新版的SMRT LINK，目前是V11.0，压缩包有点大(1.8GB)。

```
wget https://downloads.pacbcloud.com/public/software/installers/smrtlink_11.0.0.146107.zip
unzip smrtlink_11.0.0.146107.zip #生成smrtlink_11.0.0.146107.run文件和它的md5文件
./smrtlink_11.0.0.146107.run --smrttools-only #安装smrttools，会在当前目录下生成smrtlink目录
```

`./smrtlink/smrtcmds/bin/`下会有很多命令，包括bam2fasta，samtools，minimap2，falconc等。
`./smrtlink/admin/bin/`下也有一些可用的命令。



# Hifi组装基因组的软件选择

对11种针对HiFi测序技术的组装工具的评估结果显示，**hifiasm**和**hifiasm-meta**分别成为组装真核基因组和宏基因组的优选工具。

在真核生物基因组组装中，hifiasm在不同方法比较的组装基因组均具有更高的连续性、完整性和准确性；HiCanu、Verkko与LJA次之，但Verkko与LJA具有组装的contig较短等缺陷；NextDenovo仅对单倍体基因组具有更好的性能。宏基因组组装评估中，hifiasm-meta以及metaflye的组装错误最少，但是在面对复杂宏基因组时hifiasm-meta的完整性及连续性明显优于metaflye，但同时也会保留部分冗余的序列。


## hifiasm组装

`nohup hifiasm -o sample_prefix -t 48 --h1 sample_HiC_1.fq.gz --h2 sample_HiC_2.fq.gz Hifi.fastq.gz 2>&1 > hifiasm.log &`

一定要用命令保存日志log文件，好查看hifiasm组装的一些参数，帮助判断是否需要调整参数。

1. 时效：300Mb基因组，48线程，组装耗时3.5小时。


## Hifiasm作者建议
- Hifiasm默认运行清除单倍型重复（purges haplotig duplications）的步骤。对于纯合基因组（inbred or homozygous genomes），需要用-l0参数禁止运行清除步骤。
- 旧的HiFi reads可能在两端包含短的接头（adapter）序列。可以用-z20参数修剪掉reads两端的20bp序列。
- 对于小的基因组，可以使用-f0参数禁止初始过滤步骤（initial bloom filter）来节约内存，这个步骤在程序早期消耗16GB内存。
- 对于比人基因组（3Gb）大得多的基因组，使用-f38或-f39参数来节约k-mer计算的内存。


## log文件解释（debug）
Hifiasm在log文件（前面生成的hifiasm.log）中打印的几个信息可用于debug，主要是判断Hifiasm是否根据k-mer频数分布图正确判断了纯合子覆盖度值。

1. k-mer频数分布图（k-mer plot）（这与做genome survey时原理一致）
- log文件会先打印一个k-mer plot，如果指定了Hi-C数据，还会再接着打印几轮校正（round 1，2，3，finally）的k-mer plot。我们做debug关注第一个总的k-mer plot即可。
- 对于纯合子样本（homozygous samples），k-mers 频数分布图应该有一个峰（代表纯合reads的频数分布中心），峰值位置k-mer频数用Ho_peak来代表。
- 对于杂合子样本（heterozygous samples），k-mers 频数分布图应该有两个峰。 频数大一点的峰代表纯合reads覆盖和分布中心（峰值位置k-mer频数用Ho_peak代表）；频数小一点的峰代表杂合reads覆盖和分布中心（峰值位置k-mer频数用He_peak代表）。
- 如果出现非典型的单峰或双峰的k-mer频数分布图，那么可能是样本污染导致。
2. 纯合子覆盖度（homozygous coverage）
- log文件会在打印完k-mer plot后，打印一行：`[M::purge_dups] homozygous read coverage threshold: 90.` 
- 这个90是hifiasm确定的纯合子覆盖度值（用Ho_coverage代表）。要检查Ho_coverage值是否接近k-mer plot中确定的纯和峰值Ho_peak值，如果不接近（比如更接近He_peak值）那表明hifiasm错误地确定了纯合子覆盖度值，此时组装的基因组要么太大，要么太小。要用--homo-cov参数设置纯合子覆盖度值为Ho_peak值。
3. 纯合/杂合碱基数量（number of het/hom bases）
- log文件会在打印一行：`[M::stat] # heterozygous bases: 437440353; # homozygous bases: 93698357`
- 分别代表在Hi-C定向组装（Hi-C phased assembly）时，unitig graph中多少碱基是纯合的（homozygous bases），多少碱基是杂合的（heterozygous bases）。
- 对于杂合子样本，通常杂合碱基比纯合碱基数量多。如果log文件中显示纯合碱基数量比杂合碱基数量还多，代表hifiasm错误地确定了纯合子覆盖度值（Ho_coverage），需要用--homo-cov参数设置纯合子覆盖度值为Ho_peak值。


## 建议这样跑Hifiasm组装二倍体物种

1. 先用默认参数跑一遍

`nohup hifiasm -o sample_prefix -t 48 --h1 sample_HiC_1.fq.gz --h2 sample_HiC_2.fq.gz Hifi.fastq.gz 2>&1 > hifiasm.log &`

2. 查看hifiasm.log文件，如果k-mer plot只有一个峰代表是纯合子样本，则加-l0参数关闭purge duplication步骤，再跑一遍。

`nohup hifiasm -o sample_prefix -t 48 -l0 --h1 sample_HiC_1.fq.gz --h2 sample_HiC_2.fq.gz Hifi.fastq.gz 2>&1 > hifiasm.log &`

3. 查看hifiasm.log文件，如果k-mer plot有两个峰代表是杂合子样本。
- 首先，确定纯合子覆盖值（Ho_coverage值，即homozygous read coverage threshold）是否判断正确，接近k-mer图中的较大的峰值Ho_peak值即为判断正确。
- 然后，检查碱基数量是否异常。Hi-C模式下，纯合碱基比杂合碱基要多即为异常，则会错误判断纯合子覆盖度值。
- 如果有问题，需要使用参数--hom-cov指定纯合子覆盖度为纯合峰值（Ho_peak值，比如90）重跑一遍。

`nohup hifiasm -o sample_prefix -t 48 --hom-cov 90 --h1 sample_HiC_1.fq.gz --h2 sample_HiC_2.fq.gz Hifi.fastq.gz 2>&1 > hifiasm.log &`

4. 检查组装结果是否异常，调整参数

- 组装的基因组大小是否符合预期（genome survey估计的或者流式细胞仪测定的基因组大小）：组装的基因组过大或过小通常是由于hifiasm判断错了纯合子覆盖值。检查日志文件，如果判断错了，用参数--hom-cov指定，重跑一遍。
- 分型的两套基因组大小是否相差不大：如果两套基因组差别过大，可能是做purge的程度不够，可以调低-s值（默认0.55）来改善。
- 组装的基因组序列是否足够长：如果序列不够长，片段化明显，则可以尝试增加 -D 和 -N, 会提高重复区域的分辨率，同时也会增加运行时间。
- 是否存在错误组装：如果后续的Hi-C，或者BioNano发现hifiasm组装结果有比较多错误组装，则可以适当降低 --purge-max, -s和 -O。或者设置 -u 关闭post-join 步骤，hifiasm通过该步骤提高组装的连续性。

5. 还有几个Hi-C辅助组装（Hi-C integrated assembly）的参数。调高以下参数可能改善结果，但增加耗时。
- --n-weight：rounds of reweighting Hi-C links。默认是3轮。
- --n-perturb：rounds of perturbation。默认是10000。
- --f-perturb：fraction to flip for perturbation。默认是0.1。
- --l-msjoin：detect misjoined unitigs of >=INT in size。默认是500000，这个参数非常棘手，尽量不调整。


## 组装后
1. 格式转换
- `awk '/^S/{print ">"$2;print $3}' test.bp.p_ctg.gfa > test.p_ctg.fa  # get primary contigs in FASTA`
- 用awk从gfa文件提取主要的contigs（第一列为S的contigs），生成fasta格式的基因组文件。




# references
1. Hifi组装基因组的软件的比较：https://genome.cshlp.org/content/34/2/326
2. hifiasm manual：https://hifiasm.readthedocs.io/_/downloads/en/latest/pdf/
3. http://xuzhougeng.com/archives/assemble-hifi-pacbio-with-hifiasm

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>