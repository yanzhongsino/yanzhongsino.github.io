---
title: GFF3文件处理软件 —— GFF3toolkit工具包的介绍和使用
date: 2022-05-24
categories: 
- bioinfo
- fileformat
tags: 
- GFF3
- GFF3toolkit
description: 介绍GFF3格式文件处理软件 —— GFF3toolkit工具包，包括检测GFF3格式错误，修正GFF3格式错误，合并GFF3格式文件，排序GFF3格式文件，用GFF3格式文件生成序列等。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283095&auto=1&height=32"></iframe></div>

# 1. GFF3toolkit 简介
[GFF3toolkit](https://github.com/NAL-i5K/GFF3toolkit)是用于处理GFF3格式文件的一个基于python的工具包，功能包括检测GFF3格式错误，修正GFF3格式错误，合并GFF3格式文件，排序GFF3格式文件，用GFF3格式文件生成序列等。

# 2. GFF3toolkit 安装
`pip install gff3tool`

# 3. GFF3toolkit 模块
GFF3toolkit包含许多模块：
- gff3_QC：检测gff3格式错误
- gff3_fix：修正gff3格式错误
- gff3_merge：合并两个gff3文件
- gff3_sort：根据scaffold，coordinates坐标来排序gff3文件
- gff3_to_fasta：根据基因组fasta和注释gff生成gene/cds/protein/exon等序列

# 4. GFF3toolkit 使用
## 4.1. gff3_merge
用于合并两个gff3注释文件。
1. 命令
`gff3_merge -g1 sample1.gff3 -g2 sample2.gff3 -f genome.fa -og merged.gff3 -r merged.report`

2. 参数
- -g1 sample1.gff3：指定待合并的gff3文件1
- -g2 sample2.gff3：指定待合并的gff3文件2
- -f genome.fa：指定基因组文件
- -og merged.gff3：指定合并后结果文件
- -r merged.report：指定合并报告结果文件
- -noAuto：这个参数关闭添加replace tags的自动任务。默认是开启的。
- -a：默认开启添加replace tags的自动任务只应用在没有replace tags的transcript转录本上，-a参数设置为应用在所有转录本上。

3. 输出
- merged.gff3：合并后的gff文件
- merged.report：合并报告

## 4.2. gff3_sort
用于排序gff3文件。根据scaffold(seqID)，coordinates坐标排序基因，根据特征关系(feature relationship)排序基因内注释。

排序的前提假设：
- 任何没有Parent属性的注释都是父注释。
- 所有子注释都排在其父注释之后，并在新的父注释之前。

1. 命令
`gff3_sort -g sample.gff3 -og sorted.gff3`

2. 参数
- -g sample.gff3：指定输入gff3文件
- -og sorted.gff3：指定输出排序好的gff3文件
- -t sort_template.txt：指定排序模板，按照指定模板的features进行基因内注释的排序。
- -i：按特征类型(feature type)对多亚型基因(multi-isoform gene)进行排序，默认是关闭的。
- -r：此参数按gff3文件里scaffold(seqID)出现的顺序排序，默认是按照scaffold的数字排序。

3. 排序模板sort_template.txt文件示例
```
gene pseudogene
mRNA
exon
CDS
```

4. 输出
- sorted.gff3：排序结果文件

## 4.3. gff3_QC
用于检测gff3注释的格式错误
1. 命令
`nohup gff3_QC -g sample.gff3 -f genome.fa -o sample.qc -s statistic.txt >qc.log 2>&1 &`

2. 参数
- -g sample.gff3：指定需要质控的gff文件
- -f genome.fa：指定基因组文件
- -o sample.qc：指定质控结果输出文件
- -s statistic.txt：指定统计输出文件

3. 输出
- sample.qc：质控结果

```
Line_num	Error_code	Error_level	Error_tag
['Line 1']	Esf0014	Error	["##gff-version" missing from the first line]
['Line 1']	Esf0041	Error	[Unknown reserved (uppercase) attribute: "Augustus_transcriptSupport_percentage"]
['Line 749964']	Ema0002	Warning	[Protein sequence contains internal stop codons at bp 2397350]
['Line 750456']	Ema0001	Warning	[Parent feature start and end coordinates exceed those of child features]
['Line 18852', 'Line 18838']	Emr0002	Warning	[Incorrectly split gene parent?]
```

- statistic.txt：质控结果的统计

```
Error_code	Number_of_problematic_models	Error_level	Error_tag
Esf0014	1	Error	##gff-version" missing from the first line
Esf0041	378399	Error	Unknown reserved (uppercase) attribute
Ema0006	9	Info	Wrong phase
Esf0027	8	Error	Phase is required for all CDS features
Ema0002	468	Warning	Protein sequence contains internal stop codons
Ema0001	1024	Warning	Parent feature start and end coordinates exceed those of child features
Emr0002	31	Warning	Incorrectly split gene parent?
```

- qc.log：运行log文件，也包含了质控相关信息。

## 4.4. gff3_fix
用于修正gff3注释文件格式错误
1. 命令
`nohup gff3_fix -qc_r sample.qc -g sample.gff3 -og corrected.gff3 >fix.log 2>&1 &`

2. 参数
- -qc_r sample.qc：指定质控结果文件作为修正参考，这里用gff3_QC的输出文件
- -g sample.gff3：指定需要修正的gff文件
- -og corrected.gff3：指定输出修正后的gff文件

3. 输出
- corrected.gff3：修正后的gff文件
- fix.log：运行log文件。

## 4.5. gff3_to_fasta
用于根据基因组fasta和注释gff生成gene/cds/protein/exon等序列，速度很快。
1. 命令
`gff3_to_fasta -g sample.gff3 -f genome.fa -st all -d simple -o sample`

2. 参数
- -g sample.gff3：指定需要修正的gff文件
- -f genome.fa：指定基因组文件
- -st all：指定输出的序列类型，all/gene/exon/cds/pep/trans/pre_trans/user_defined。
- -u mRNA CDS：如果用-st user_defined指定了输出自定义的序列，则需要-u参数指定自定义序列名称的parent和child类型。比如-u mRNA CDS会输出CDS序列。
- -d simple：指定输出fasta文件的序列ID的格式，simple是只输出gff文件的ID到fasta文件，complete输出完整的信息。
- -noQC：默认会执行质控程序，如果不想做质控，加上这个参数。
- -o sample：指定输出文件前缀

3. 输出
输出指定序列，比如：
- sample_gene.fa
- sample_exon.fa：exon序列是根据注释情况每条注释生成一条exon序列。
- sample_cds.fa：cds序列是一个mRNA的一条CDS序列。
- sample_pep.fa
- sample_trans.fa
- sample_pre_trans.fa

# 5. reference
1. gff3toolkit github：https://github.com/NAL-i5K/GFF3toolkit

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>