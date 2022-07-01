---
title: 软件mitogenomics
date: 2022-06-30
categories: 
- bio
- biosoft
tags: 
- mitogenomics
- aln2tbl.py
- aln2tbl-legacy.py
- mitos2fasta.py
- tbl
- organelle
- mitogenome
description: 介绍软件mitogenomics，用于线粒体基因组相关的格式转换，貌似是常用于动物线粒体数据的软件，常用于软件MITOS2的后续分析。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=27901965&auto=1&height=32"></iframe></div>

# mitogenomics简介
- 无意中发现的一个软件。
- 基于已有线粒体基因组序列和基因序列，通过比对获取注释tbl格式文件，所以只适用于线粒体的基因。
- 包括两个转化格式的脚本mitos2fasta.py，aln2tbl.py（和python2版本）。

# 软件安装
1. git下载
`git clone https://github.com/IMEDEA/mitogenomics`

2. 脚本
- 因为软件是三个脚本组成，可以直接使用脚本。

3. 依赖
- python3的Biopython和argparse模块。
- 可以用pip安装`pip install biopython argparse`。

# 软件mitogenomics
## 脚本mitos2fasta.py
python 3 版本，用于转化基因序列为比对到线粒体基因组序列的格式。

1. 命令
`mitos2fasta.py -m mito.fa -g genes.fa -c Y > assembly.fa`

2. 输入输出
- -m mito.fa：线粒体基因组序列，fasta格式。
- -g genes.fa：基因序列，fasta格式。可以是软件MITOS2的输出。
- -c Y：是否简化基因名字（genes.fa文件的序列ID）并适应aln2tbl.py，Y/N。
- assembly.fa：输出保存到assembly.fa，即将线粒体基因组序列和基因序列比对好的序列格式，基因没比对的位置用-代替。

## aln2tbl.py
python 3 版本，用于转化比对到线粒体基因组序列的基因序列（即mitos2fasta.py的输出）为tbl格式。

aln2tbl-legacy.py是aln2tbl.py的python2版本，功能一样。

1. 命令
`aln2tbl.py -f assembly.fa -g genes.txt -c 1 > sample.tbl`

2. 输入输出
- -f assembly.fa：输入文件是线粒体基因组和基因序列的比对文件，mitos2fasta.py的输出。
- -g genes.txt：保存了基因名称的文本文件，单行，多个基因名称间逗号分隔。
- -c number_genetic_code：用数字指定线粒体编码方式，植物线粒体是一般的编码方式 (1)。此外还有脊椎动物vertebrate (2), 酵母菌yeast (3), 霉菌mold, 原生动物protozoan and 腔肠动物coelenterate (4), 无脊椎动物invertebrate (5), 棘皮动物echinoderm and 扁形虫flatworm (9), 海鞘类ascidian (13)。
- > sample.tbl：输出到tbl格式文件。

# references
1. https://github.com/IMEDEA/mitogenomics