---
title: 基因组组装和注释结果的整理
date: 2021-08-10 16:30:00
categories: 
- omics
- genome
tags:
- genome
- genome annotation
- sort
- rename
- combine
- extract
description: 基因组和基因组注释的排序、重命名、合并和提取等整理操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1833352&auto=1&height=32"></iframe></div>


# 1. 基因组整理【组装之后，注释之前】
## 1.1. 基因组碱基整理
基因组碱基-u修改成大写字母形式，同时建议使用-w 0把序列输出限定为一行序列。

`seqkit seq -u -w 0 genome.old.fa>genome.new.fa`


## 1.2. 基因组碱基小写带来的问题：
1. edta运行TIR预测时，报错并中断运行。

```
EDTA/bin/TIR-Learner2.5/Module2/RunGRF.py", line 79, in <module>
    if (len(str(records[0].seq))>int(length)+500):
IndexError: list index out of range
cp: cannot stat 'TIR-Learner/*-p': No such file or directory
```

2. geta生成基因注释结果减少。
前期运行不受影响，会在geta生成的/out.tmp/6.combineGeneModels/中合并前面步骤预测结果时把许多基因预测结果识别为partial而筛除掉本为完整的基因，可看到/out.tmp/6.combineGeneModels/genome.gff3的gene数量是正常的，而out.tmp/6.combineGeneModels/genome.completed.gff3明显降低，/out.tmp/6.combineGeneModels/genome.partial.gff3明显升高。out.tmp/6.combineGeneModels/genome.filter.gff3是最后的基因注释结果，即out.gff3也相应减少。
3. maker的某些运行步骤也会出错。

## 1.3. 基因组序列ID修改
### 1.3.1. 排序【optional】--根据需要选择
按长度对基因组contigs进行从长到短的排序

`seqkit sort -l -r -w 0 genome.old.fa >genome.sort.fa` 

### 1.3.2. 重命名【推荐】
最好重命名成长度一致的数字，避免后期分析遇到contig1和contig11同时被匹配的问题。
1. solution A【速度最快，推荐】
序列IDs按顺序命名为1-n的数字，--nr-width可以设置数字的长度，长度为5时第一条序列名称为00001。

`seqkit replace -p '.*' -r {nr} --nr-width 5 genome.sort.fa` 

2. solution B
把序列id重命名成1-n的数字

`awk 'BEGIN{i=0 ; FS="," ; OFS=","}{ if(/>/){gsub($1,">"++i,$1);print $0}else{print $0}}' genome.sort.fa>genome.fa` 

基因组的序列id改为scaf001的形式

`sed -i "s/>/>scaf/g" genome.fa` 

3. solution C
把基因组序列id的数字1-100改成0001-0100这种数字位数相同的格式。这个没有solution A快，只推荐在不想做大规模更改的情况下使用。

用下面脚本实现：

```replace_digits.sh
#!/bin/bash
j=1
for i in `seq -w 1 0100` #把相同数字位数的0001-0100依次赋值给变量i
do
	sed -i -E -e "s/scaffold$j$/scaffold_$i/g" -e "s/scaffold$j([^0-9])/scaffold_$i\1/g" species.fa #两次替换，第一次替换scaffold$j为行尾的字符串（比如在基因组序列文件中），第二次替换scaffold$j不为行尾的字符串，[^0-9]代表不为数字的任意一个字符，\1代表替换前括号([^0-9])中的内容。
	sed -i -E -e "s/scaffold$j$/scaffold_$i/g" -e "s/scaffold$j([^0-9])/scaffold_$i\1/g" species.gff #同上，替换其他文件，比如gff文件。
	j=$(($j+1)) #把1-100依次赋值给变量j
done
```

# 2. 基因组注释整理【注释结果整理】
## GFF3toolkit【推荐】
推荐使用GFF3toolkit来进行gff3注释文件的整理，具体介绍参考博客[GFF3toolkit blog](https://yanzhongsino.github.io/2022/05/24/biosoft_fileformat_GFF3toolkit/)。

GFF3toolkit包含许多模块：
- gff3_QC：检测gff3格式错误
- gff3_fix：修正gff3格式错误
- gff3_merge：合并两个gff3文件
- gff3_sort：根据scaffold，coordinates坐标来排序gff3文件
- gff3_to_fasta：根据基因组fasta和注释gff生成gene/cds/protein/exon等序列

## 2.1. 合并多个注释结果
基因组的基因结构注释，如果使用了多款软件进行，想要合并多套注释结果，并让注释的基因根据染色体和位置信息排序，可参考这个办法。

合并两个基因注释文件（若是多个就依次合并）

1. 输出A.gff中符合要求的行，即A.gff中与B.gff中位置有overlap的行。
`bedtools intersect -a A.gff -b B.gff -wa >A.dup.gff` 
2. 输出只在A.gff存在的行，即把A.dup.gff从A.gff中删除，需要两个文件都已排序，若未排序先sort排序再处理。
`comm -1 A.gff A.dup.gff >A.filter.gff` 

3. 合并B.gff和A.gff中与B.gff不重复的部分。
`cat B.gff A.filter.gff >sample.gff` 

## 2.2. 重命名注释结果【可选】
根据需要选择【非必要的】，根据染色体位置顺序对注释的基因结果进行排序和重命名整理。
除了基因注释外，不同分析源都可能发现新的基因，增加注释的基因数量，从而打乱顺序。

### 2.2.1. 排序
1. 把sample.gff3中gene替换成1gene，mRNA替换成2mRNA，exon替换成3exon，CDS替换成4CDS；目的是确保排序后单个基因内部的顺序是1-2-3-4。
`sed -i -e "s/gene/1gene/g" -e "s/mRNA/2mRNA/g" -e "s/exon/3exon/g" -e "s/CDS/4CDS/g" sample.gff3`
2. 根据染色体位置排序。按照第一列第四个字符，第四列数值，第五列数值逆序依次排序。
`cat sample.gff3 |sort -k 1.4n -k 4n -k 5nr >sample.sort.gff3`
3. 获得的sample.sort.gff3文件再1gene替换成gene，234类似替换。
`sed -i -e "s/1gene/gene/g" -e "s/2mRNA/mRNA/g" -e "s/3exon/exon/g" -e "s/4CDS/CDS/g" sample.sort.gff3`

### 2.2.2. 重命名
1. 复制一份注释文件用于修改和替换
`cp sample.sort.gff3 sample.new.gff3`
2. 获取旧ID的list。获取sample.sort.gff3第九列中的ID值。
`cat sample.new.gff3 | awk '$3=="gene" {print $9}'|sed -e "s/;.*//g" -e "s/ID=//g" >old.name`
3. 获取新ID的list。
`for i in $(seq -w 1 `cat old.name|wc -l`); do echo scaf$i; done >new.name`
4. 合并旧的和新的ID。
`paste -d "/" old.name new.name > old2new.name`
5. 生成替换脚本old2new.sh。
`sed -e "s/^/sed -i \"s\//g" -e "s/$/\/g sample.new.gff3/g\"" old2new.name >old2new.sh`
6. 运行替换脚本，会直接替换sample.new.gff3文件的旧ID为新ID，可能会运行较长时间。
`sh old2new.sh`

ps：这部分代码可实现，但是是非常冗余的代码量（水平有限），且未必适用所有注释文件，谨慎参考。

## 2.3. 从注释文件提取序列
用gffread根据注释文件从基因组提取基因序列:
- 从基因组genome.fa和注释文件sample.gff3获取cds序列
`gffread -x sample.cds.fa -g genome.fa sample.gff3`
- 从基因组genome.fa和注释文件sample.gff3获取exon序列
`gffread -w sample.exon.fa -g genome.fa sample.gff3`
- 从基因组genome.fa和注释文件sample.gff3获取protein序列
`gffread -y sample.protein.fa -g genome.fa sample.gff3`

## 2.4. 从注释文件提取intron信息
脚本[extract_intron_info.pl](https://github.com/yanzhongsino/bioscripts/blob/main/modifiedscripts/extract_intron_info.pl)


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>