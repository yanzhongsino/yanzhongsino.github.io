---
title: genome assessment
date: 2022-05-16
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- organelle
- transcriptome
- mapping rates
- BWA
- HiSat2
- minimap2

description: 记录基因组评估的方法，用测序reads，包括pacbio，illumina，RNA-seq reads，mapping回基因组，得到mapping rates，mapping rates越高代表基因组的。
---

<div align="middle"><music URL></div>

## 基因组评估

评估组装的基因组的连续性（continuity）和完整性（completeness）常用方法有：
1. mapping rates：包括pacbio，illumina，RNA-seq reads等数据都可以映射回基因组，然后计算比对率来判断基因组的精确度。
2. QUAST：contig N50 和 scaffold N50
3. BUSCO：基因组和预测的蛋白组都可以用BUSCO评估基因组的完整度(completeness)。
4. LAI：通过LTR组装指数评估基因组的连贯性(continuity)。


## 基因组评估的方法——mapping

把测序reads与组装好的基因组做alignment，这个操作常被称为mapping。mapping之后生成SAM/BAM格式文件，通过分析SAM/BAM格式文件，获取reads mapping回参考基因组的信息，从而评估基因组组装的质量。

主要是通过以下三个量化信息来评估：
- reads的mapping rate：mapped reads number/total reads number
- genome coverage：mapped genome length/total genome length
- depth的分布：基因组上每个碱基mapped碱基的数量称为单碱基的深度（depth），或者通过滑窗统计基因组上每个固定大小（比如1000bp）的窗口的mapped碱基的平均数量作为窗口深度，分析深度在基因组上的分布可以判断基因组组装的质量。

此外，通过可视化软件直观地查看reads在基因组上具体的mapping情况，也可以判断基因组组装是否存在问题和评估质量。

## Illumina reads

### BWA mapping
1. 工具
BWA-MEM+samtools

1. 建索引
`bwa index ref.fa`

2. bwa mapping
`bwa mem -t 4 ref.fa R1.clean.fq r2.clean.fq | samtools sort -@ 4 -m 4G > illumina.bam &`

3. samtools flagstat统计
`samtools flagstat illumina.bam >illumina.flagstat`

### 结果
1. illumina.flagstat的结果示例

```
51231959 + 0 in total (QC-passed reads + QC-failed reads) #共有51231959条reads通过QC+0条reads未通过QC，后面的信息行中+后的都是代表QC没通过的reads的数量。
12372427 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
50020312 + 0 mapped (97.63% : N/A) # 97.63%比例的reads mapping到参考序列上，这就是mapping record rate
38859532 + 0 paired in sequencing
19429766 + 0 read1 # 双端reads中read1的总数
19429766 + 0 read2 # 双端reads中read2的总数
36727148 + 0 properly paired (94.51% : N/A) # 94.51%比例的reads成对的映射上
37160066 + 0 with itself and mate mapped # read映射上但配对read没映射上的数量
487819 + 0 singletons (1.26% : N/A) # 1.26%比例的read没映射上的同时，配对read映射上了
289948 + 0 with mate mapped to a different chr # reads和配对reads映射到不同染色体的情况下的reads数量
205767 + 0 with mate mapped to a different chr (mapQ>=5) # reads和配对reads映射到不同染色体，且映射质量大于等于5的情况下的reads数量
```

2. samtools flagstat的mapping record rate

$$mapping record rate=mapped recorder number/total recorder number=((primary) mapped reads number + secondary mapped reads number)/(total reads number + secondary mapped reads number)$$

其中，$recorder number$代表sam文件中去除header部分的比对记录数量（每行一条比对记录，即行数）。

同一reads可能多次mapping，有多条记录，所以$recorder number$的数量会比$reads number$多。

- mapped recorder number用的是50020312；
- total recorder number用的是51231959；
- secondary mapped reads number是12372427；

$$mapping record rate=50020312/51231959*100\%=97.63\%$$

有文章直接用mapping record rate，但建议用mapping rate来代表mapped reads的比例。

3. 计算mapping rate
- 通常我们在文章中使用reads的比例来代表mapping rate，通过计算公式，可以利用samtools flagsta的统计数据计算mapping rate。

$$mapping rate = mapped reads number/total reads number = (mapped recorder number - secondary mapped reads number)/(total recorder number - secondary mapped reads number) = (50020312-12372427)/(51231959-12372427) = 37647885/38859532*100\% = 96.88\%$$

## PacBio/Nanopore reads：minimap2
1. 直接mapping
`minimap2 -t 8 -ax map-pb ref.fa pacbio_reads.fq >pacbio.sam &`

`minimap2 -t 8 -ax map-ont ref.fa ont_reads.fq >nanopore.sam &`

- -t 8：线程
- -a：输出sam格式，默认是PAF格式
- -x: 选择数据类型，map-pb是pacbio数据，map-ont是nanopore数据。

## RNA-seq reads：HiSat2
对于RNA-seq数据，用HiSat2进行reads比对。

### hisat2比对
1. 建索引
`hisat2-build ref.fa ref.hisat`

2. mapping
`hisat2 --dta -p 8 -x ref.index -1 rna1_1.fa -2 rna1_2.fa 2>rna1_hisat.log |samtools sort -@ 12 > rna1_hisat.bam &` #样品1，保存rna1_hisat.log文件，里面有包括mapping rate的统计信息。

`hisat2 --dta -p 8 -x ref.index -1 rna2_1.fa -2 rna2_2.fa 2>rna2_hisat.log |samtools sort -@ 12 > rna2_hisat.bam &` #样品2，保存rna2_hisat.log文件，，里面有包括mapping rate的统计信息。

`samtools merge -@ 8 merged_hisat.bam rna1_hisat.bam rna2_hisat.bam`  #合并多个bam文件到一个

### hisat2比对统计结果
1. hisat2比对统计结果hisat.log示例

```
19429766 reads; of these: # reads总数
  19429766 (100.00%) were paired; of these: # 配对的reads数量
    1066192 (5.49%) aligned concordantly 0 times # 一致地比对了0次的reads数量
    14853850 (76.45%) aligned concordantly exactly 1 time # 一致地比对了1次的reads数量
    3509724 (18.06%) aligned concordantly >1 times # 一致地比对了大于1次的reads数量
    ----
    1066192 pairs aligned concordantly 0 times; of these: # 一致地比对了0次的reads数量中：
      54954 (5.15%) aligned discordantly 1 time # 不一致地比对了1次的reads数量
    ----
    1011238 pairs aligned 0 times concordantly or discordantly; of these: #一致或不一致地比对了0次的reads数量中：
      2022476 mates make up the pairs; of these: # 配对的reads数量中：
        1211647 (59.91%) aligned 0 times #比对0次的数量
        607196 (30.02%) aligned exactly 1 time #比对1次的数量
        203633 (10.07%) aligned >1 times #比对大于1次的数量
96.88% overall alignment rate # mapping rate，由mapped reads number/total reads number的比例计算得到
[bam_sort_core] merging from 20 files and 4 in-memory blocks...
```

2. hisat2结果解释
- hisat.log结果中，`19429766 reads; of these:`及大部分包含的信息中，双端测序的reads是只统计一次的。比如19429766 reads代表的是有19429766对双端测序的reads，总reads数量是$19429766*2=38859532$条。
- 在`2022476 mates make up the pairs; of these:`及之后包含的信息中，代表配对的reads数量，双端测序的reads是统计了配对的所有reads，总reads数量就是2022476条。

3. hisat2的mapping rate的计算
96.88%的overall alignment rate即为mapping rate，计算方法是：

$$mapping rate=mapped reads number/total reads number$$

  - total reads number用的$19429766*2$。
  - mapped reads number包含：$concordantly exactly 1 time(14853850*2)$，$aligned concordantly >1 times(3509724*2)$，$aligned discordantly 1 time(54954*2)$，$mates make up the pairs$中的$aligned exactly 1 time(607196)$和$aligned >1 times(203633)$。

  $$mapping rate=((14853850+3509724+54954)*2+607196+203633)/(19429766*2) = 37647885/38859532*100\%=96.88\%$$

  - mapped reads number的另一种计算方法：$concordantly exactly 1 time(14853850)$，$aligned concordantly >1 times(3509724)$，$aligned concordantly 0 times(1066192)$中aligned到的所有reads，即除了$aligned concordantly 0 times(1066192)$中的$aligned 0 times(1211647/2)$以外的所有reads。

  $$mapping rate=(14853850+3509724+54954+1066192-(1211647/2))/19429766*100\%=96.88\%$$

### samtools flagstat统计结果
1. samtools flagstat
- 如果hisat2运行时未保存log文件，也可以用`samtools flagstat in.bam >in.flagstat`来计算reads的mapping统计值。
- 与illumina reads的flagstat计算结果格式一样。
- flagstat统计结果中，记录的是sam/bam文件中reads的记录数量，双端测序包含配对的所有reads。

2. in.flagstat示例

```
51231959 + 0 in total (QC-passed reads + QC-failed reads) #共有51231959条reads通过QC+0条reads未通过QC，后面的信息行中+后的都是代表QC没通过的reads的数量。
12372427 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
50020312 + 0 mapped (97.63% : N/A) # 97.63%比例的reads mapping到参考序列上，这就是mapping record rate
38859532 + 0 paired in sequencing
19429766 + 0 read1 # 双端reads中read1的总数
19429766 + 0 read2 # 双端reads中read2的总数
36727148 + 0 properly paired (94.51% : N/A) # 94.51%比例的reads成对的映射上
37160066 + 0 with itself and mate mapped # read映射上但配对read没映射上的数量
487819 + 0 singletons (1.26% : N/A) # 1.26%比例的read没映射上的同时，配对read映射上了
289948 + 0 with mate mapped to a different chr # reads和配对reads映射到不同染色体的情况下的reads数量
205767 + 0 with mate mapped to a different chr (mapQ>=5) # reads和配对reads映射到不同染色体，且映射质量大于等于5的情况下的reads数量
```

3. samtools flagstat的mapping record rate的计算方法

$$mapping record rate=mapped recorder number/total recorder number=((primary) mapped reads number + secondary mapped reads number)/(total reads number + secondary mapped reads number)$$

其中，$recorder number$代表sam文件中去除header部分的比对记录数量（每行一条比对记录，即行数）。

同一reads可能多次mapping，有多条记录，所以$recorder number$的数量会比$reads number$多。

- mapped recorder number用的是50020312；
- total recorder number用的是51231959；
- secondary mapped reads number是12372427；

$$mapping record rate=50020312/51231959*100\%=97.63\%$$

4. 与hisat2的统计结果的不同
- samtools flagstat的mapping record rate（97.63%）比hisat2的mapping rate（96.88%）高一些，原因在于计算方式的区别。

5. 计算mapping rate
- 通常我们在文章中使用reads的比例来代表mapping rate（即hisat2的计算方式），通过计算公式，可以利用samtools flagsta的统计数据计算mapping rate。

$$mapping rate = mapped reads number/total reads number = (mapped recorder number - secondary mapped reads number)/(total recorder number - secondary mapped reads number) = (50020312-12372427)/(51231959-12372427) = 37647885/38859532*100\% = 96.88\%$$

## Hi-C reads：Juicer







# 基因组组装质量的评估
用clean reads映射（mapping）回组装好的初始基因组，然后查看mapping的效果。

如果是长度不长的细胞器基因组，可以在IGV直接查看mapping情况。
如果是大的核基因组，则考虑过滤mapping得到的sam/bam文件，然后计算深度和其他统计值来评估质量。

## 查看深度
查看mapped reads的深度。在全基因组范围是否保持相当，如果有极端高/低的深度，则怀疑组装错误，或者映射了错误的reads，这可能代表以下特殊情况。

1. 水平基因转移
在同时测序情况下，由于数量分布差异，期望测序深度是叶绿体>线粒体>核基因，且三者差异至少在一个数量级。

评估线粒体基因组时，可以考虑是否有叶绿体/核的reads映射到基因组上，并同时考虑水平基因转移的可能性。

- 显著高的深度可能是叶绿体的reads。
- 显著低的深度可能是核的reads。
- 轻微高的深度可能是核转移到线粒体的情况（一般这种情况映射到的序列不长，<100bp）。可以调整参数，只保留映射超过100bp的reads即可排除这种情况。


2. 重复序列
如果基因组上存在重复序列，组装时只得到其中一个拷贝，则在这个拷贝处的mapped reads深度会是附近序列的两倍（三个拷贝就三倍）。
重复序列一般只分析超过100bp(或者>50bp)的情况，这可以和核基因转移到线粒体的情况区别开。
重复序列之间非常接近，但不一定是完美匹配。

3. 异质性
由于一个细胞中有多个细胞器，细胞器之间还可能存在不同构象或不同碱基的位点。这种情况的存在称为细胞器的异质性（不常见）。

具有异质位点的细胞器基因组的证据：
- 排除重复序列映射的可能性。
- 异质位点映射的reads包含两种（也可能多种，以下同）碱基，一般少数种的占比超过5%就可能算异质性了。
- 异质位点映射的reads的两种碱基的深度加起来应该与周围位点的深度相当。
- 异质位点映射的reads不全都很短（>100bp），双端测序的最好是成对映射。
- 异质位点周围的位点映射了同一read，且周围位点的mapping效果很好（单一碱基，均匀深度）。



# references
1. hisat2和samtools flagstat计算的mapping rate不同的解释：https://zhuanlan.zhihu.com/p/73208822