---
title: 蛋白质比对转换成CDS比对 —— ParaAT,PAL2NAL
date: 2021-10-29 17:50:00
categories:
- bioinfo
- align
tags:
- tutorial
- sequence alignment
- protein
- CDS
- biosoft
- ParaAT
- PAL2NAL
description: 记录了两种先进行蛋白质比对，再根据蛋白质比对转换成CDS比对的软件和使用方法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=17059176&auto=1&height=32"></iframe></div>

# 1. 序列比对(sequence alignment)
序列比对的知识可以参考另一篇博文的内容。
[Multiple Sequence Alignment](https://yanzhongsino.github.io/2021/09/06/bioinfo_MSA/)

如果序列间差异较大，为了获得更精准的比对，有时我们会先做蛋白质的比对，然后根据蛋白质比对，转化成CDS比对。这篇博客就是记录把蛋白质比对转换成CDS比对的软件和使用。

# 2. 蛋白比对转CDS比对
## 2.1. 成对比对(pairwise alignment)
### 2.1.1. 应用场景
如果是两条序列比对的转化，批量处理，可以用ParaAT脚本。

比如用MCScanX软件做了共线性分析找到物种间的homologs基因对，或者WGD分析找到物种内的paralogs基因对，想要根据蛋白比对做对应的CDS比对。比对之后用于计算Ka和Ks。

### 2.1.2. 成对比对软件
#### 2.1.2.1. ParaAT
ParaAT是中科院基因组所张章课题组在2012年开发，2014年更新了2.0版本，是一个perl脚本。

[ParaAT download](https://ngdc.cncb.ac.cn/tools/paraat)
[ParaAT paper](https://www.sciencedirect.com/science/article/pii/S0006291X12003518)

#### 2.1.2.2. ParaAT下载
```
wget ftp://download.big.ac.cn/bigd/tools/ParaAT2.0.tar.gz
tar -zxf ParaAT2.0.tar.gz
cd ParaAT2.0
ParaAT.pl -h
```

ParaAT2.0目录下有两个脚本，ParaAT.pl用于成对比对的转换（可以批量处理），目录下还有另一个多序列转换脚本Epal2nal.pl（好像是pal2nal.pl的V13版本）。

#### 2.1.2.3. ParaAT使用
1. 输入文件
三个输入文件,sample.id,cds.fa,pep.fa，三个文件的序列id要一致。
- sample.id文件
两列，每行对应两条要做成对比对的序列ID，任意行，ParaAT可以批量处理多个成对比对。

`cat sample.collinearity |grep "species_prefix"|cut -f2,3 >sample.id` 用MCScanX的结果文件提取blocks的同源gene对，获得sample.id文件。

- cds.fa
包括所有需要比对的cds序列的文件
- pep.fa
包括所有需要比对的蛋白序列的文件

2. 运行
- `echo "12" >proc`
指定ParaAT.pl使用线程，也可以不指定，默认是6个线程

- `ParaAT.pl -g -t -h sample.id -n cds.fa -a pep.fa -m mafft -p proc -f axt -o paraat 2>paraat.log`
先根据sample.id指定的id做蛋白的成对比对，然后根据蛋白的比对结果转换成对应的CDS的比对结果。

参数：
- -h sample.id：指定基因对文件
- -n cds.fa：指定CDS序列文件，包含所有sample.id涉及的基因的CDS序列；
- -a pep.fa：指定氨基酸序列文件，包含所有sample.id涉及的基因的氨基酸序列；
- -m mafft：指定比对软件
- -p proc：指定线程
- -f axt：指定输出比对的CDS序列的格式（WGD计算KaKs一般用axt格式比较多）
- -o paraat：指定输出文件目录(如果不存在会新建目录)。

其他参数：
- -g：移除比对后包含gap的密码子；建议加上-g和-t，免得后面计算Ks时报错Error. The size of two sequences in 'ctg00816-ctg08844' is not equal。
- -t：移除mismatched codons；建议加上-g和-t，免得后面计算Ks时报错Error. The size of two sequences in 'ctg00816-ctg08844' is not equal。
- -k：用KaKs_Calculator计算(需要输出axt格式)Ka和Ks，获得axt文件后自动计算kaks值，使用MA模型，比YN模型慢，推荐输出axt后自己用KaKs_Calculator计算并用YN模型。
- -c 1：指定编码方式，不同生物的编码方式不同，默认是标准型1。

3. 报错
ERROR: inconsistency between the following pep and nuc seqs

## 2.2. 多序列比对(multiple sequence alignment)
### 2.2.1. 应用场景
如果是多序列的转化，可以使用PAL2NAL脚本。

比如做了orthofinder2找到多个物种的orthogroups，要对每一组orthogroups进行蛋白比对转换成CDS比对。

### 2.2.2. 多序列比对软件
#### 2.2.2.1. PAL2NAL
[PAL2NAL介绍](http://www.bork.embl.de/pal2nal/)

- PAL2NAL可以将蛋白的多序列比对转换成CDS比对，如果输入的是一对序列，还会通过paml的codeml程序自动计算dn和ds。
- 如果数据量少，可以通过上面的网页进行转换。

#### 2.2.2.2. PAL2NAL下载
```shell
wget http://www.bork.embl.de/pal2nal/distribution/pal2nal.v14.tar.gz
tar -zxvf pal2nal.v14.tar.gz
pal2nal.pl -h
```

#### 2.2.2.3. PAL2NAL使用
`pal2nal.pl -nogap -nomismatch pep.aln nuc.fa -output fasta >nuc.aln`

需要已经做好比对的蛋白质序列pep.aln和ID一致的cds序列nuc.fa

参数：
- -output (clustal|paml|fasta|codon) # Output format; default = clustal
- -nogap  # remove columns with gaps and inframe stop codons
- -nomismatch # remove mismatched codons (mismatch between pep and cDNA) from the output

# 3. references
1. https://en.wikipedia.org/wiki/Sequence_alignment


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>