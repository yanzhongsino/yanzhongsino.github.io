---
title: 基因组质量评估：（四）用LAI评估基因组的连贯性
date: 2022-07-12
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- organelle
- transcriptome

description: 用LTR Assembly Index (LAI)这个参数来评估基因组组装的连贯性。
---

<div align="middle"><music URL></div>

# LAI简介
## LAI是什么
LTR组装指数(LTR Assembly Index, LAI)是

## LAI评估基因组组装质量

# 计算LAI指数
## 注释LTR时，顺带就计算LAI【推荐】


## 单独计算LAI
在LTR预测的结果上，计算LAI。

运行EDTA之后，可以利用EDTA的LTR预测结果计算LAI值，单独计算LAI耗时不长（<10min）。

1. 输入文件
- genome.fa.mod.pass.list：如果用EDTA预测了LTR，则在此路径下可以找到这个文件：/path/to/edta/genome.fa.mod.EDTA.raw/LTR/genome.fa.mod.pass.list
- genome.fa.mod.out：如果用EDTA预测了LTR，则在此路径下可以找到这个文件：/path/to/edta/genome.fa.mod.EDTA.final/genome.fa.mod.out

2. 运行LAI

`nohup LAI -t 24 -genome genome.fa -intact /path/to/edta/genome.fa.mod.EDTA.raw/LTR/genome.fa.mod.pass.list -all /path/to/edta/genome.fa.mod.EDTA.final/genome.fa.mod.out &> lai.out &`

3. 参数
- -t 24：blast使用的线程数量
- -genome genome.fa：指定基因组
- -intact genome.fa.mod.pass.list：指定LTR_retriever生成的非冗余LTR-RT文库列表
- -all genome.fa.mod.out：指定RepeatMasker注释的所有LTR序列
- -window 3000000：指定计算LAI的window，默认是3Mb
- -step 300000：指定计算LAI的step，默认是300Kb
- -q：快速评估LTR identity的模式，建议大基因组用，牺牲0.5%的精度。本人试过300Mb基因组非快速模式2分钟就出结果了，所以除非特别大的基因组，否则不建议用。
- -qq：超快速模式。不评估LTR identity，只输出raw_LAT值（用于种间比较）。
- -mono [file]：用文件提供序列名称，LAI只计算指定的序列。主要用于多倍体基因组，在文件中提供代表单倍体的序列。

4. 输出
- sample.fa.mod.out.LAI：保存了raw_LAI值和LAI值，包括全基因组的和滑窗范围的。
- sample.fa.mod.out.LAI.LTR.ava.age
- sample.fa.mod.out.LAI.LTR.ava.out

## intact LTR reconstruction

references
1. paper：https://academic.oup.com/nar/article/46/21/e126/5068908
2. github：https://github.com/oushujun/LTR_retriever


- 欢迎关注微信公众号：生信技工。
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>