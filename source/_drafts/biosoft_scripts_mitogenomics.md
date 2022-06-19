---
title: 软件mitogenomics
date: 2022-06-19
categories: 
- bio
- bioinfo
tags: 
- aln2tbl.py
- aln2tbl-legacy.py
- mitos2fasta.py
description: 介绍生物信息学常用文件格式。
---

<div align="middle"></div>


无意中发现的一个软件，包括两个转化格式的脚本mitos2fasta.py，aln2tbl.py。


# 软件下载
`git clone https://github.com/IMEDEA/mitogenomics`

因为软件是三个脚本组成，可以直接使用脚本。

依赖：python3的Biopython和argparse模块，可以用pip安装`pip install biopython argparse`。

# mitos2fasta.py
python 3 版本，用于转化基因序列为比对到线粒体基因组序列的格式。

1. 命令
`mitos2fasta.py -m mito.fa -g genes.fa -c Y > assembly.fa`

2. 参数
- -m mito.fa：线粒体基因组序列，fasta格式。
- -g genes.fa：基因序列，fasta格式。MITOS2的输出。
- -c Y：是否简化基因名字（genes.fa文件的序列ID）并适应aln2tbl.py，Y/N。
- assembly.fa：输出保存到assembly.fa，即比对好的基因序列。

# aln2tbl.py
python 3 版本，转化比对到线粒体基因组序列的基因序列为tbl格式文件。

1. 命令
`aln2tbl.py -f assembly.fa -g forward_genes_file.txt -c number_genetic_code > sample.tbl`

2. 参数


aln2tbl-legacy.py
aln2tbl.py的python2版本。






# references
1. https://github.com/IMEDEA/mitogenomics