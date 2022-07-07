---
title: 基因组质量评估：（）软件qualimap
date: 2022-07-06
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- coverage
- depth
- mapping
- BAM
- biosoft
- qualimap

description: 记录软件qualimap
---

<div align="middle"><music URL></div>


# qualimap简介
qualimap（http://qualimap.conesalab.org/）用于分析bam文件，更详细的统计信息，包括每条contig的mapping情况



bamqc模块用于单个NGS样本bam文件的统计
rnaseq模块用于RNA-seq样本bam文件的统计
multi-bamqc模块用于多样本NGS的bam文件的分组统计，即包含个体数据，又分组比较
comp-counts模块用于比较multi-bamqc生成的count文件中的一些特性，并作图输出

# 下载安装


# 使用
## bamqc模块
1. 运行
`qualimap bamqc -bam sample.bam -outformat PDF:HTML -outdir out -nt 12 --java-mem-size=8G`

参数：
- -bam sample.bam：指定bam文件。
- -outformat PDF:HTML：输出文件格式PDF和HTML，默认是HTML。
- -outdir out：输出文件的目录，不指定则生成sample_stats目录。
- -nt 12：线程12，默认是144。
- --java-mem-size=8G：如果运行提示out of memory，则需要参数限制内存上限。

2. 结果

所有结果下载后，可以用浏览器打开qualimapReport.html或者打开report.pdf，包括以下几部分内容：

- globals：reads的mapping情况
- ACGT content：四种碱基和N的含量
- Coverage：深度
- Mapping Quality：平均值
- Insert Size：平均值和标准差
- Mismatches and indels：统计值
- Chromosome stats：每条染色体的长度，mapped bases，mean coverage，standard deviation。
- Coverage across reference：贯穿整个基因组的深度（coverage）和GC含量
- Coverage Histogram：深度分布
- genome fraction coverage
- duplication rate histogram
- mapped reads nucleotide content
- mapped reads GC-content distribution
- mapped reads clipping profile
- homopolymer indels
- mapping quality across reference
- mapping quality histogram
- insert size across reference
- insert size histogram

qualimap的所有coverage都是深度，而不是覆盖度。

## rnaseq模块


## multi-bamqc模块

多样本NGS的bam文件统计


`nohup qualimap --java-mem-size=10G multi-bamqc -r -d qualimap.list -outdir qualimap -outformat PDF:HTML &`

参数：
- --java-mem-size=10G设置最大内存为10G，建议每个模块都设置
- -r multi-bamqc可以输入bam文件或者bamqc的结果，如果输入bam则需加-r参数
- -outdir 结果文件输出目录
- -outformat 结果文件格式，pdf和html都要
- -d qualimap.list 输入文件列表，qualimap有三列，每行一个样本，第一列样品名称，第二列包含路径的bam文件/bamqc结果目录，第三列组名
- 如果用-r，qualimap.list的第二列则应为bam文件，此时multi-bamqc模块会先对每个样本运行bamqc，bamqc的结果存放在bam文件所在目录下，再进行multi-bamqc的统计。
- 默认是用4个线程，一个样本一个样本单独跑bamqc。

## comp-counts模块


# references
1. http://qualimap.conesalab.org/