---
title: 基因组质量评估：（四）用LAI评估基因组组装的连贯性
date: 2022-07-20
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- LAI
- LTR

description: 用LTR Assembly Index (LAI)这个参数来评估基因组组装的连贯性。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=26834823&auto=1&height=32"></iframe></div>

# 1. LAI简介
## 1.1. LAI
1. LTR

关于重复序列和LTR的相关知识可以参考博客：基因组注释（一）：重复序列注释：https://yanzhongsino.github.io/2021/08/02/omics_genome.annotation_repeat/。

2. LAI

LTR组装指数(LTR Assembly Index, LAI)是用完整LTR-RTs在所有LTR-RTs的占比来评估基因组组装连贯性的一个指数。

## 1.2. LAI评估基因组组装质量
- LTR在基因组中的比例很高，且由于其重复性组装难度高，所以它的组装质量与基因组组装有密切关系。
- 期望基因组组装的连贯性越高，LTR的连贯性也越高。所以可以用LAI来评估基因组组装的连贯性。
- LAI与基因组大小、基因组LTR-RT含量和基因空间评估指标（例如BUSCO和CEGMA）无关。
- 使用LAI评估基因组，完整LTR-RTs和总LTR-RTs至少占基因组大小的0.1%和5%。

## 1.3. LAI计算和评估方法
1. LAI计算公式

$$Raw LAI = (Intact LTR retrotransposon length / Total LTR sequence length) × 100$$

2. LAI评估方法

根据LAI开发者的文章，把LAI值分成三个类别，Draft级别（0≤LAI<10），Reference级别（10≤LAI<20），Gold级别（20≤LAI）。

LAI值按照如下评估标准进行分类：

|Category|LAI|Examples|
|---|---|---|
|Draft|0 ≤ LAI < 10|Apple (v1.0), Cacao (v1.0)|
|Reference|10 ≤ LAI < 20|Arabidopsis (TAIR10), Grape (12X)|
|Gold|20 ≤ LAI|Rice (MSUv7), Maize (B73 v4)|

<iframe height=850 width=90% src="https://academic.oup.com/view-large/126092648" frameborder=0 allowfullscreen></iframe>

# 2. 计算LAI指数
## 2.1. 注释LTR时，顺带计算LAI【推荐】
用LTR_FINDER_parallel和ltrharvest分别预测LTR，然后基于预测结果用LTR_retriever鉴定和确认LTR。

### 2.1.1. LTR_FINDER_parallel预测LTR

`LTR_FINDER_parallel -seq genome.fa -threads 10 -harvest_out -size 1000000 -time 300 -w 2 -C -D 15000 -d 1000 -L 7000 -l 100 -p 20 -M 0.85`

- 预测LTR,生成genome.fa.finder.combine.scn
- 其中，-w 2 -C -D 15000 -d 1000 -L 7000 -l 100 -p 20 -M 0.85参数部分是作者推荐；
- -size, -time, -try1三个参数作者不建议修改
- -harvest_out指定输出harvest格式的结果

### 2.1.2. ltrharvest预测LTR
1. 建基因组索引

`gt suffixerator -db genome.fa -indexname index/sample -tis -suf -lcp -des -ssp -sds-dna`

2. 预测LTR

`gt ltrharvest -index index/sample -minlenltr 100 -maxlenltr 7000 -mintsd 4 -maxtsd 6 -motif TGCA -motifmis 1 -similar 85 -vic 10 -seed 20 -seqids yes > genome.harvest.scn`

生成genome.harvest.scn

### 2.1.3. 合并预测结果【可选&不推荐】

`cat genome.harvest.scn genome.fa.finder.combine.scn > genome.rawLTR_merge.scn`

因为LTR_FINDER_parallel和ltrharvest的预测结果都是一行一条信息，可以直接合并。

### 2.1.4. LTR_retriever鉴定LTR

`LTR_retriever -genome genome.fa -inharvest genome.harvest.scn -infinder genome.fa.finder.combine.scn -threads 12 -u 4.79e-9`

- 这步骤耗时长。运行LTR_retriever默认会计算LAI指数，不用单独运行LAI。
- 生成的genome.out.LAI包含整个genome和每个contig的LAI指数。
- 如果前一步合并了多个LTR预测结果，则只需要在`-inharvest genome.rawLTR_merge.scn`输入合并结果。

## 2.2. 单独计算LAI
- 运行EDTA/LTR_retriever预测LTR的基础上，可以利用已有的LTR预测结果计算LAI值，单独计算LAI耗时不长（<10min）。
- 虽然EDTA也是调用LTR_retriever预测LTR，但默认不计算LAI。

1. 输入文件
- genome.fa.mod.pass.list：如果用EDTA调用LTR_retriever预测了LTR，则在此路径下可以找到这个文件：`/path/to/edta/genome.fa.mod.EDTA.raw/LTR/genome.fa.mod.pass.list`；如果用LTR_retriever直接预测的，在`LTR_retriever/genome.fa.mod.pass.list`
- genome.fa.mod.out：如果用EDTA预测了LTR，则在此路径下可以找到这个文件：`/path/to/edta/genome.fa.mod.EDTA.final/genome.fa.mod.out`

2. 运行LAI

`nohup LAI -t 24 -genome genome.fa -intact genome.fa.mod.pass.list -all genome.fa.mod.out &> lai.out &`

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

# 3. references
1. paper：https://academic.oup.com/nar/article/46/21/e126/5068908
2. github：https://github.com/oushujun/LTR_retriever


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>