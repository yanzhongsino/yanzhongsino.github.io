---
title: 基因组质量评估：（一）概述
date: 2022-06-30
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- organelle
- transcriptome

description: 基因组的组装和注释的质量评估，包括基因组组装的完整性、准确性和连续性等质量评估，以及对基因组注释的质量进行评估。这里讨论的质量评估主要用于核基因组，但许多质量评估方法也适用于细胞器基因组（包括线粒体和叶绿体）和转录组的质量评估。
---

<div align="middle"><music URL></div>


## intact LTR reconstruction


1. 运行LAI
运行EDTA之后，可以直接计算LAI值。

`nohup LAI -t 24 -genome genome.fa -intact /path/to/edta/genome.fa.mod.EDTA.raw/LTR/genome.fa.mod.pass.list -all /path/to/edta/genome.fa.mod.EDTA.final/genome.fa.mod.out &> lai.out &`

2. 参数
- -t 24：blast使用的线程数量
- -genome genome.fa：指定基因组
- -intact genome.fa.mod.pass.list：指定LTR_retriever生成的非冗余LTR-RT文库列表
- -all genome.fa.mod.out：指定RepeatMasker注释的所有LTR序列
- -window 3000000：指定计算LAI的window，默认是3Mb
- -step 300000：指定计算LAI的step，默认是300Kb
- -q：快速评估LTR identity的模式，建议大基因组用，牺牲0.5%的精度。本人试过300Mb基因组2分钟就出结果了，所以除非特别大的基因组，否则不建议用。
- -qq：超快速模式。不评估LTR identity，只输出raw_LAT值（用于种间比较）。
- -mono [file]：用文件提供序列名称，LAI只计算指定的序列。主要用于多倍体基因组，在文件中提供代表单倍体的序列。

3. 输出
- sample.fa.mod.out.LAI：保存了raw_LAI值和LAI值，包括全基因组的和滑窗范围的。
- sample.fa.mod.out.LAI.LTR.ava.age
- sample.fa.mod.out.LAI.LTR.ava.out



