---
title: 基因组质量评估：（五）mapping法：5. 用软件QualiMap统计BAM文件
date: 2022-07-31
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- mapping
- sam
- bam
- depth
- biosoft
- QualiMap

description: mapping法评估基因组组装质量。这篇文章主要介绍了软件QualiMap的安装和使用。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=18511124&auto=1&height=32"></iframe></div>

# 1. QualiMap简介
- QualiMap是用于统计bam文件的基于java的软件，结果包含详细的统计信息和可视化图形，以及每条contig的mapping情况。
- 官网（http://qualimap.conesalab.org/）有软件的详细介绍。
- 每个模块运行的输入输出文件的案例都可以在这里查看：http://qualimap.conesalab.org/doc_html/samples.html#bam-samples

# 2. QualiMap的模块
1. **bamqc模块**（BAM QC）：用于单个NGS样本bam文件的QC统计。
2. **rnaseq模块**（RNA-seq QC）：用于转录组RNA-seq样本bam文件的QC统计。
3. **multi-bamqc模块**（Multi-sample BAM QC）：用于多样本NGS的bam文件的分组QC统计，即包含个体数据，又包含分组比较。
4. **counts模块**（Counts QC）：可用于转录组数据计数的统计，用于量化表达水平。
5. **clustering模块**：用于表观基因组（例如甲基化）特征的聚类。
6. **comp-counts模块**：输入bam文件和注释文件，计算映射到每个区域的reads数量。

# 3. 下载安装
下载解压即可使用

```
wget https://bitbucket.org/kokonech/qualimap/downloads/qualimap_v2.2.1.zip
unzip qualimap_v2.2.1.zip
cd qualimap_v2.2.1
./qualimap -h
```

# 4. 使用
## 4.1. bamqc模块
bamqc模块用于单个NGS样本bam文件的统计。

1. 运行

`qualimap bamqc -bam sample.bam -outformat PDF:HTML -outdir out -nt 12 --java-mem-size=10G`

2. 参数
- -bam sample.bam：指定bam文件。
- -outformat PDF:HTML：输出文件格式PDF和HTML，默认是HTML。
- -outdir out：输出文件的目录，不指定则生成sample_stats目录。
- -nt 12：线程12，默认是144。
- --java-mem-size=10G：设置最大内存为10G，建议每个模块都设置。

3. 结果

所有结果下载后，可以用浏览器打开qualimapReport.html或者打开report.pdf，可以参考给出的结果例子：https://rawgit.com/kokonech/kokonech.github.io/master/qualimap/HG00096.chr20_bamqc/qualimapReport.html。

包括以下几部分内容：

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

QualiMap的所有coverage都是深度，而不是覆盖度。

## 4.2. rnaseq模块
与bamqc模块相似，用于RNA-seq数据的bam文件的统计。

1. 运行

`qualimap rnaseq -bam rnaseq_sample.bam -outdir rnaseq_out -outformat PDF:HTML --java-mem-size=10G`

2. 参数
- -bam rnaseq_sample.bam：输入的bam文件。
- -outdir rnaseq_out：输出文件的目录。
- -outformat PDF:HTML：输出文件格式PDF和HTML，默认是HTML。
- --java-mem-size=10G：设置最大内存为10G。

## 4.3. multi-bamqc模块
多样本NGS的bam文件的统计和比较。

1. 运行

`qualimap multi-bamqc -r -d qualimap.list -outdir out -outformat PDF:HTML --java-mem-size=10G`

2. 参数
- -r：`multi-bamqc`模块可以输入bam文件或者`bamqc`模块的结果，如果输入bam文件则需加-r参数。
- -d qualimap.list：输入文件列表，qualimap有三列，每行一个样本，第一列样品名称，第二列包含路径的bam文件/bamqc结果目录，第三列组名。
- 如果用-r，qualimap.list的第二列则应为bam文件，此时multi-bamqc模块会先对每个样本运行bamqc，bamqc的结果存放在bam文件所在目录下，再进行multi-bamqc的统计。默认是用4个线程，一个样本一个样本单独跑bamqc。
- -outdir out：结果文件输出目录。
- -outformat PDF:HTML：结果文件格式，pdf和html都要。
- --java-mem-size=10G：设置最大内存为10G。

## 4.4. counts,clustering,comp-counts模块
此外，还有counts,clustering,comp-counts模块。

... ...

# 5. tips
如果遇到报错RAM不足，可以加上参数`--java-mem-size=10G`指定内存上限。

# 6. references
1. http://qualimap.conesalab.org/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>