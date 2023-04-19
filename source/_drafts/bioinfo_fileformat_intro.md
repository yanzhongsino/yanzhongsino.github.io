---
title: 生物信息学常用文件格式
date: 2021-11-12
categories: 
- bio
- bioinfo
tags: 
- gene set enrichment analysis
- GSEA
- topGO
description: 介绍生物信息学常用文件格式。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=399341112&auto=1&height=32"></iframe></div>


各种生物学使用文件格式的解释

1. seqkit
2. bedtools
3. vcftools/bcftools
4. gff3read
5. gff3toolkit: https://github.com/NAL-i5K/GFF3toolkit


# fasta&fastq
seqkit


fastq

fasta

## sample.fa.fai
`samtools faidx sample.fa`会生成sample.fa的索引文件sample.fa.fai

该命令对输入的fasta序列有一定要求：对于每条序列，除了最后一行外， 其他行的长度必须相同。

sample.fa.fai共5列，tab分隔

第一列 NAME：序列的名称，只保留“>”后，第一个空白之前的内容；

第二列 LENGTH：序列的长度， 单位为bp；

第三列 OFFSET： 第一个碱基的偏移量， 从0开始计数，换行符也统计进行；

第四列 LINEBASES：除了最后一行外， 其他代表序列的行的碱基数， 单位为bp；

第五列 LINEWIDTH：行宽， 除了最后一行外， 其他代表序列的行的长度， 包括换行符， 在windows系统中换行符为\r\n, 要在序列长度的基础上加2；




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



-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>


