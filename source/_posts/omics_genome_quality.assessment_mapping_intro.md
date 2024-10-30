---
title: 基因组质量评估：（五）mapping法：1. 简介
date: 2022-07-23
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- organelle
- plastome
- mitogenome
- transcriptome
- mapping
- BWA
- HiSat2
- minimap2
- sam
- bam

description: mapping法评估基因组组装质量。mapping法是指把测序的reads（包括Pacbio，Illumina，RNA-seq 等reads）映射回组装好的基因组，评估mapping rate，genome coverage，depth分布等指标，用这些指标评估基因组组装质量。这篇文章简单介绍了mapping法的评估工具和评估指标。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2192010&auto=1&height=32"></iframe></div>

# 1. 基因组评估的方法——mapping法
把测序reads与组装好的基因组做alignment，这个操作常被称为mapping。mapping之后生成SAM/BAM格式文件，通过分析SAM/BAM格式文件，获取reads mapping回参考基因组的信息（比如mapping rate，coverage，depth），从而评估基因组组装的质量。

## 1.1. mapping工具
不同的reads可以用不同的软件进行mapping

|reads|mapping tools|
|---|---|
|Illumina DNA-seq reads|BWA|
|Pacbio reads/ONT reads|minimap2|
|Illumina RNA-seq|HiSat2|

## 1.2. 评估指标
主要是通过以下三个量化指标来评估组装质量：
1. mapping rate
- reads的mapping rate：$mapped reads number/total reads number$
- HiSat2对RNA-seq进行mapping时把mapping rate统计在log文件中
- `samtools flagstat`，bamdst等软件也可以统计mapping rate
2. genome coverage
- genome coverage：$mapped genome length/total genome length$
- `samtools depth`，bedtools，bamdst等软件也可以统计genome coverage
3. depth
- 平均depth：计算基因组的平均深度作为参考指标
- depth的分布：基因组上每个碱基mapped碱基的数量称为单碱基的深度（depth），或者通过滑窗统计基因组上每个固定大小（比如1000bp）的窗口的mapped碱基的平均数量作为窗口深度，分析深度在基因组上的分布可以判断基因组组装的质量。
- 此外，通过可视化软件直观地查看reads在基因组上具体的mapping情况，也可以判断基因组组装是否存在错误碱基、组装结构问题。
- `samtools mpileup`,`samtools depth`，qualimap，bamdst，mosdepth等软件可以计算平均深度和深度分布信息。

# 2. mapping实操
用特定工具对各种reads进行mapping，生成SAM/BAM文件。
## 2.1. Illumina reads：BWA
用BWA-MEM+samtools对Illumina reads进行mapping

1. 建索引
- `bwa index ref.fa`
2. bwa mapping
- `bwa mem -t 4 ref.fa R1.clean.fq r2.clean.fq | samtools sort -@ 4 -m 4G > illumina.bam &`
3. 建bam的索引文件
- `samtools index sample.bam` # 为sample.bam建立索引，生成索引文件sample.bam.bai。在IGV等软件查看必须要有索引文件。

## 2.2. PacBio/Nanopore reads：minimap2
用minimap2对三代reads进行mapping
1. 建索引
- `minimap2 -x map-ont -d ref.mmi ref.fa`
- 为参考序列ref.fa建索引，生成索引文件ref.mmi
- 建索引时也要根据reads的不同设置-x参数。map-pb是PacBio的CLR数据，map-ont是nanopore数据，map-hifi/asm20用于PacBio的HiFi数据。

2. mapping
- `minimap2 -ax map-pb ref.fa pacbio.fq.gz -t 8 > aln.sam`      # PacBio CLR genomic reads
- `minimap2 -ax map-ont ref.fa ont.fq.gz -t 8 > aln.sam`         # Oxford Nanopore genomic reads
- `minimap2 -ax map-hifi ref.fa pacbio-ccs.fq.gz -t 8 > aln.sam` # PacBio HiFi/CCS genomic reads (v2.19 or later)
- `minimap2 -ax asm20 ref.fa pacbio-ccs.fq.gz -t 8 > aln.sam`    # PacBio HiFi/CCS genomic reads (v2.18 or earlier)

3. 参数
- -a：输出sam格式，默认是PAF格式
- -x: 选择数据类型，map-pb是PacBio的CLR数据，map-ont是nanopore数据，map-hifi/asm20用于PacBio的HiFi数据。
- -t 8：线程

## 2.3. RNA-seq reads：HiSat2
### 2.3.1. mapping
对于RNA-seq数据，用HiSat2进行reads的mapping。

1. 建索引
- `hisat2-build ref.fa ref.hisat`

2. mapping
- `hisat2 --dta -p 8 -x ref.hisat -1 rna1_1.fa -2 rna1_2.fa 2>rna1_hisat.log |samtools sort -O BAM -@ 12 > rna1_hisat.bam &` #样品1，保存rna1_hisat.log文件，里面有包括mapping rate的统计信息。
- `hisat2 --dta -p 8 -x ref.hisat -1 rna2_1.fa -2 rna2_2.fa 2>rna2_hisat.log |samtools sort -O BAM -@ 12 > rna2_hisat.bam &` #样品2，保存rna2_hisat.log文件，，里面有包括mapping rate的统计信息。

3. merge
- `samtools merge -@ 8 merged_hisat.bam rna1_hisat.bam rna2_hisat.bam`  #合并多个bam文件到一个bam文件

# 3. 评估指标
## 3.1. mapping rate
1. mapping rate的计算公式
- reads的mapping rate：$mapped reads number/total reads number$
2. mapping rate的计算工具
- HiSat2对RNA-seq进行mapping时把mapping rate统计在log文件中
- `samtools flagstat`可用于统计mapping rate
- bamdst等软件也可以统计mapping rate

## 3.2. genome coverage
1. genome coverage的计算公式
- genome coverage：$mapped genome length/total genome length$

2. `samtools depth`统计genome coverage

```
samtools depth -aa sample.bam >depth.out # 计算所有位点的深度
u = $(cat depth.out |awk '$3 == 0 {print $0}'|wc -l) # 统计没有mapped碱基的长度，并赋值给u
t = $(cat depth.out |wc -l) # 统计所有位点的长度，并赋值给t。这个值与与基因组大小一致。
echo "scale=5; 1-$u/$t" | bc #计算基因组覆盖度
```

3. bedtools
- `bedtools genomecov`可以统计coverage，具体参数和结果还没看，留个坑。
- `bedtools genomecov -ibam sample.bam -d >sample.depth`

## 3.3. depth
### 3.3.1. depth分布的统计工具
- `samtools mpileup`,`samtools depth`，qualimap，bamdst，mosdepth等软件可以计算平均深度和深度分布信息。

### 3.3.2. depth的具体指标
1. 平均depth
- 计算基因组的平均深度作为参考指标。
2. depth分布
- 基因组上每个碱基mapped碱基的数量称为单碱基的深度（depth），或者通过滑窗统计基因组上每个固定大小（比如1000bp）的窗口的mapped碱基的平均数量作为窗口深度，分析深度在基因组上的分布可以判断基因组组装的质量。
3. 直接观察depth
- 此外，通过可视化软件直观地查看reads在基因组上具体的mapping情况，也可以判断基因组组装是否存在错误碱基、组装结构问题。


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>