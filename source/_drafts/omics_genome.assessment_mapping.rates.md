---
title: genome assessment
date: 2022-05-16
categories:
- omics
- genome
- genome assessment
tags:
- genome
- genome assessment

description: 记录基因组评估的方法，用测序reads（包括pacbio，illumina，RNA-seq reads） mapping回基因组，得到mapping rates，mapping rates越高代表基因组的。
---

<div align="middle"><music URL></div>

## 基因组评估

评估组装的基因组的常用方法有：
1. mapping rates：包括pacbio，illumina，RNA-seq reads等数据都可以映射回基因组，然后计算比对率来判断基因组的精确度。
2. QUAST：
3. BUSCO：基因组和预测的蛋白组都可以用BUSCO评估基因组的完整度(completeness)。
4. LAI：通过LTR组装指数评估基因组的连贯性(continuity)。


## 基因组评估
1. mapping rates
Illumina genomic and RNA-seq reads were aligned to the genome using BWA-MEM53 and HISAT2, respectively, to calculate mapping rate.

用BWA+SAMtools映射illumina reads到基因组，然后用samtools flagstat统计mapping rates

用HISAT2映射RNA-seq reads到基因组
hisat2-build -p 24 ../../../genome/Adiantum.nelumboides.fa Adiantum.nelumboides.hisat2 &
hisat2 --dta -p 8 -x Adiantum.nelumboides.hisat2 -1../Leaf.clean.R1.fq.gz -2 ../Leaf.clean.R2.fq.gz |samtools sort -@ 12 > root.bam & #这一步会输出关于mapping rates的信息。

比如这样的输出，28767481的总reads数量，有85.96%的总mapping rate，这个mapping rate是计算了concordantly exactly 1 time(22653070)，aligned concordantly >1 times(1172778)，和aligned concordantly 0 times(4941633)中aligned到的所有reads。即除了aligned concordantly 0 times(4941633)中的aligned 0 times(8080542/2)以外的所有reads。

```
28767481 reads; of these:
  28767481 (100.00%) were paired; of these:
    4941633 (17.18%) aligned concordantly 0 times
    22653070 (78.75%) aligned concordantly exactly 1 time
    1172778 (4.08%) aligned concordantly >1 times
    ----
    4941633 pairs aligned concordantly 0 times; of these:
      233031 (4.72%) aligned discordantly 1 time
    ----
    4708602 pairs aligned 0 times concordantly or discordantly; of these:
      9417204 mates make up the pairs; of these:
        8080542 (85.81%) aligned 0 times
        1218747 (12.94%) aligned exactly 1 time
        117915 (1.25%) aligned >1 times
85.96% overall alignment rate
[bam_sort_core] merging from 24 files and 12 in-memory blocks...
```

2. QUAST
quast.py genome.fsa -g Adiantum.nelumboides.gff -1 ./illumina/Adiantum.reniforme_illumina.clean_R1.fq.gz -2 ./illumina/Adiantum.reniforme_illumina.clean_R2.fq.gz -t 12 --large -o an_quast

3. BUSCO
We also performed BUSCO evaluation to examine completeness of the assembly with the Eukaryota_odb10 database.

4. LAI
The LTR assembly index10 was used to assess continuity