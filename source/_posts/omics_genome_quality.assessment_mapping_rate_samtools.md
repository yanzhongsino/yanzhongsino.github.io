---
title: 基因组质量评估：（五）mapping法：2. samtools计算mapping rate
date: 2022-07-23
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- transcriptome
- mapping
- mapping rate
- HiSat2
- sam
- bam
- samtools
- samtools flagstat

description: mapping法评估基因组组装质量。mapping法是指把测序的reads（包括Pacbio，Illumina，RNA-seq 等reads）映射回组装好的基因组，评估mapping rate，genome coverage，depth分布等指标，用这些指标评估基因组组装质量。这篇文章简单介绍了mapping法的其中一个评估指标：mapping rate。通过samtools计算mapping rate，和HiSat2对RNA-seq进行mapping时把mapping rate统计在log文件中，两者间的差异和转化。

mathjax: true
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=18600061&auto=1&height=32"></iframe></div>

# 1. mapping rate
通过mapping把reads与组装好的基因组进行alignment，然后分析mapped reads的sam/bam格式文件，统计mapping rate来评估基因组组装质量。期望mapping rate越接近100%，组装质量越高。

# 2. Hisat2统计的mapping rate
运行Hisat2对RNA-seq进行mapping时生成的log文件`hisat.log`会保存着比对的mapping rate信息。

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

  - total reads number用的$19429766\*2$。
  - mapped reads number包含：concordantly exactly 1 time(14853850\*2)，aligned concordantly >1 times(3509724\*2)，aligned discordantly 1 time(54954\*2)，mates make up the pairs中的aligned exactly 1 time(607196)和aligned >1 times(203633)。

  $$mapping rate=((14853850+3509724+54954)\*2+607196+203633)/(19429766\*2) = 37647885/38859532*100\%=96.88\%$$

  - mapped reads number的另一种计算方法：concordantly exactly 1 time(14853850)，aligned concordantly >1 times(3509724)，aligned concordantly 0 times(1066192)中aligned到的所有reads，即除了aligned concordantly 0 times(1066192)中的aligned 0 times(1211647/2)以外的所有reads。

  $$mapping rate=(14853850+3509724+54954+1066192-(1211647/2))/19429766\*100\%=96.88\%$$

# 3. samtools flagstat统计mapping rate
1. samtools flagstat
- 如果hisat2运行时未保存log文件，也可以用`samtools flagstat`来计算reads的mapping统计值。
- illumina reads和Pacbio reads等的sam/bam文件也可以用这种方式统计mapping rate。
- flagstat统计结果中，记录的是sam/bam文件中reads的记录数量，即mapping record rate（双端测序包含配对的所有reads）。

2. samtools flagstat统计
- `samtools flagstat output.bam > output.flagstat`

3. output.flagstat的结果示例

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

其中，recorder number代表sam文件中去除header部分的比对记录数量（每行一条比对记录，即行数）。

同一reads可能多次mapping，有多条记录，所以recorder number的数量会比reads number多。

- mapped recorder number用的是50020312；
- total recorder number用的是51231959；
- secondary mapped reads number是12372427；

$$mapping record rate=50020312/51231959*100\%=97.63\%$$

有文章直接用mapping record rate，但建议用mapping rate来代表mapped reads的比例。

4. 与hisat2的统计结果的不同
- samtools flagstat的mapping record rate（97.63%）比hisat2的mapping rate（96.88%）高一些，原因在于计算方式的区别。

5. 计算mapping rate
- 通常我们在文章中使用reads的比例来代表mapping rate（即hisat2的计算方式），通过计算公式，可以利用samtools flagsta的统计数据计算mapping rate。

$$mapping rate = mapped reads number/total reads number = (mapped recorder number - secondary mapped reads number)/(total recorder number - secondary mapped reads number) = (50020312-12372427)/(51231959-12372427) = 37647885/38859532*100\% = 96.88\%$$

# 4. references
1. hisat2和samtools flagstat计算的mapping rate不同的解释：https://zhuanlan.zhihu.com/p/73208822

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>