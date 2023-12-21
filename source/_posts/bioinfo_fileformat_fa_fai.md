---
title: samtools faidx创建fasta/fastq格式文件的.fai格式索引文件
date: 2023-11-24
categories: 
- bioinfo
- fileformat
- fasta
tags: 
- samtools
- faidx
- fasta
- fas
- fa
- fastq
- fq
- fai
description: 介绍用samtools faidx为fasta/fastq格式文件创建.fai格式索引文件，并详细介绍.fai格式索引文件内容。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1394643615&auto=1&height=32"></iframe></div>

# 1. 为fasta/fastq格式文件创建索引
- fasta/fastq文件是用于存储生物序列的常用格式，后缀为fasta/fas/fa/fastq/fq。
- 为fasta/fastq创建索引文件，可以快速查找和提取任意位置序列。如使用GATK,IGV等软件时用到。
- `samtools faidx sample.fa`命令即可创建`sample.fa.fai`索引文件。
- `samtools faidx`命令对fasta文件的格式有要求。即对每条序列内，除最后一行外，其他行长度必须相同，行末尾的换行符也必须一致（Unix-style或Windows-style）。同一个fasta文件内的不同序列间的长度和换行符可以不一致，但同一条序列内部必须一致。
- `samtools faidx`命令对fastq文件的格式的要求是fastq的碱基质量行必须与序列行有着一样的长度。

# 2. 索引文件含义
.fai文件是文本文件，列间用TAB(\t)分隔，fa.fai共五列，fq.fai共六列（多一列QUALOFFSET）。

## 2.1. fai文件的六列

```
NAME	Name of this reference sequence
LENGTH	Total length of this reference sequence, in bases
OFFSET	Offset in the FASTA/FASTQ file of this sequence's first base
LINEBASES	The number of bases on each line
LINEWIDTH	The number of bytes in each line, including the newline
QUALOFFSET	Offset of sequence's first quality within the FASTQ file
```

## 2.2. 每列的含义
1. NAME：序列名称，保留>符号后到第一个空格前的字符串。与SAM文件的@SO header的SN值一致。
2. LENGTH：序列长度，即碱基的数量。与SAM文件的@SO header的SL值一致。
3. OFFSET：OFFSET（偏移）列包含该序列第一个碱基在 FASTA/FASTQ 文件中的偏移量（以字节为单位，从零开始计数，换行符也统计在内），即从标题行（FASTA 中为">"行，FASTQ 中为"@"行）的第一个字符到此条序列的第一个碱基（包括换行符）的字符数量。通常情况下，fai 索引文件的各行是按照序列在 FASTA/FASTQ 文件中出现的顺序排列的，因此 .fai 文件通常是按照这一列排序的。
4. LINEBASES：除了较短的最后一行外， 其他代表序列的行的碱基数， 单位为bp；
5. LINEWIDTH：除了较短的最后一行外， 其他代表序列的行的长度。与LINEBASES不同的是，LINEWIDTH 包括换行符， unix系统中换行符为\n，在序列长度基础上加1；Windows系统中换行符为\r\n, 要在序列长度的基础上加2。
6. QUALOFFSET：与 OFFSET 相同，但针对的是序列的第一个质量分数。这将是 "+"行末尾换行后的第一个字符。仅适用于 FASTQ 文件。

## 2.3. 例子
https://www.htslib.org/doc/faidx.html 中有一些fasta和fastq文件的索引文件的例子。

# 3. 应用
1. 提取指定位置序列：`samtools faidx sample.fa chr1 >chr1.fa`;`samtools faidx sample.fa chr1:100-235 >chr1_partial.fa`。
2. 统计fasta序列长度，保存到bed格式文件：`samtools faidx sample.fa && awk '{print $1"\t1\t"$2}' sample.fa.fai > sample.fa.bed`

# 4. references
1. https://www.htslib.org/doc/faidx.html

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>


