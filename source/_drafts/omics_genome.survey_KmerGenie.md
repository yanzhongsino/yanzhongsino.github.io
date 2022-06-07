---
title: 用k-mer分析进行基因组调查(genome survey) —— 用KmerGenie一步实现
date: 2022-06-15
categories:
- omics
- genome
- genome survey
tags:
- genome
- genome survey
- k-mer
- KmerGenie

description: 介绍KmerGenie，用KmerGenie做基因组调查(genome survey)的k-mer频数统计和基因组特征评估。
---

<div align="middle"><music URL></div>



### KmerGenie
2. [KmerGenie](http://blog.sciencenet.cn/blog-3406804-1159967.html)
```
$cat fastq_list.txt
path_to_sample_1.fastuniq.clean.fq
path_to_sample_2.fastuniq.clean.fq
$kmergenie fastq_list.txt -o ./sample -l 17 -k 121 -s 10 -t 4 > sample.log1.txt 2> sample.log2.txt
# 默认单倍体模式，以k-mer长度17为起始，121为终止，10为间隔逐一测试；程序运行线程数4。结果输出在当前路径下，以sample为结果文件前缀名。“sample.log1.txt”和“sample.log2.txt”分别为程序运行时的正确/错误输出日志。
```
生成结果报告文件*_report.html，报告开头以折线图的形式展示出在每种长度k-mer下，估算的基因组大小，同时给出了“最佳k-mer”选择数值（其实就是将评估基因组总大小最高的那个k-mer值判定为“最佳k-mer”，为基因组组装时k-mer的选择提供参考）。

生成各k-mer取值下的频数分布表*.histo和对应的频数分布图*.histo.pdf，以及所有k-mer取值的总计*.dat和*.dat.pdf

2. KmerGenie

生成结果报告文件*_report.html展示了在每种长度k-mer下，估算的基因组大小，同时给出了“最佳k-mer”选择数值，给组装基因组提供参考。




# references
