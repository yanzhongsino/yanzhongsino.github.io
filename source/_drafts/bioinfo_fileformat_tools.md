---
title: 生物信息学常用文件格式
date: 2021-11-12 16:00:00
categories: 
- bio
- bioinfo
tags: 
- gene set enrichment analysis
- GSEA
- topGO
description: 介绍生物信息学常用文件格式。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>


各种生物学使用文件格式的解释

1. seqkit
2. bedtools
3. vcftools/bcftools
4. gff3read
5. gff3toolkit: https://github.com/NAL-i5K/GFF3toolkit
install：`pip install gff3tool`
包含许多命令
gff3_QC：检测gff3格式错误
gff3_fix：修正gff3格式错误
gff3_merge：合并两个gff3文件
gff3_sort：根据scaffold，coordinates坐标来排序gff3文件
gff3_to_fasta：根据基因组fasta和注释gff生成gene/cds/protein/exon


gff3_to_fasta:
用来操作gff3格式文件，包括提取gene,cds,exon,pep等序列。

# fasta&fastq
seqkit


fastq

fasta

# sam&bam
sam文件是tab分隔的文本文件，bam文件是sam文件的二进制格式，以节约储存空间。


samtools

bamtools

# vcf&gvcf
vcftools

bcftools

# bed
bedtools

tab分隔，0开始

# gff&gtf






