---
title: genome submit
date: 2022-01-30 14:30:00
categories:
- bio
- bioinfo
tags:
- genome submit

description: 记录基因组提交到NCBI的genebank的步骤和注意事项。
---

<div align="middle"><music URL></div>

# 基因组提交



### table2asn
table2asn软件用于把基因组和其他信息转换成NCBI认可的sqn格式的一个文件。


nohup table2asn -t template.sbt -indir ./ -M n -Z -locus-tag-prefix L6164 -gaps-min 1 -gaps-unknown 100 -l paired-ends -j "[organism=Bauhinia variegata][strain=pink_petal]" -logfile bv.log &

-t指定template.sbt文件
-indir指定存放genome.fsa和genome.tbl/genome.gff文件的目录
-M n
-Z
-locus-tag-prefix在向NCBI申请BioProject(以及BioSample)后会分配一个locus-tag-prefix前缀，在这里指定，不指定可能在submit.dr中报错FATAL: INCONSISTENT_PROTEIN_ID。
-gaps-min 1 -gaps-unknown 100指定序列中的N的含义，如未正确指定，可能在submit.stats中报错SEQ_INST.InternalNsInSeqRaw。-gaps-min 指定代表gap的连续Ns的最小数量，1指把大于等于1个的连续Ns碱基都转换成assembly_gaps。-gaps-unknown 指定代表未知长度的gap的连续Ns的
-l paired-ends
-j "[organism=Bauhinia variegata][strain=pink_petal]"用于指定样本名[organism]和品种名[strain]，不指定可能在submit.stats中报错SEQ_DESCR.BioSourceMissing和SEQ_DESCR.NoSourceDescriptor。
-logfile bv.log指定log文件，如果运行有错误存放在log文件

# references
