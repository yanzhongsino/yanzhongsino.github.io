---
title: 用PseudogenePipeline鉴定假基因及后续分析
date: 2022-09-07
categories:
- bioinfo
- pseudogene
tags:
- pseudogene
- PseudogenePipeline
- 假基因
- Ks

description: 记录用PseudogenePipeline软件鉴定全基因组范围内的假基因，并解析结果，简要记录了后续假基因和对应真基因的Ks计算，以及与WGD的关系的分析。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=95370&auto=1&height=32"></iframe></div>

# 1. 假基因
假基因(pseudogene)是类似与功能基因的非功能性DNA片段。大多数是功能基因的多余拷贝，通常是由于移码突变或提前的终止密码子被鉴定为假基因。

由于可能含有启动子元件，假基因有可能产生低水平的转录生成RNA，但通常无法产生功能性的最终蛋白产物。

# 2. PseudogenePipeline 简介
PseudogenePipeline是Shiulab开发的一个鉴定假基因的流程包，v1.0.0版本基于python2，v2.0.0版本基于python3。

# 3. PseudogenePipeline 原理
在基因组未被注释到基因的区域（基因间隔区）查找与基因相似的片段，大多是没有完整ORF区域或者基因的片段。使用tblastn进行基因转录翻译生成的氨基酸序列和基因组上基因间隔区的比对。

# 4. PseudogenePipeline 安装
## 4.1. 依赖-requirements
1. python
2. RepeatMasker
3. tfasty：是fasta36包的一部分，version 36及以上 （https://faculty.virginia.edu/wrpearson/fasta/），实测用的fasta-36.3.8h版本。下载解压即可。

## 4.2. 下载
在 https://github.com/ShiuLab/PseudogenePipeline 直接下载PseudogenePipeline，是python脚本包，无需安装即可使用。

# 5. PseudogenePipeline 使用
## 5.1. 文件准备
文件包括：
1. genome.fa：基因组文件，建议用repeats.gff为基因组的重复序列做mask后使用genome.repeatmask.fa，以减少计算量
2. sample.protein.fa：注释的基因对应的蛋白组
3. tblastn.out：genome.repeatmask.fa和sample.protein.fa进行tblastn获得的输出文件
4. sample.gff3：基因组的蛋白编码基因注释文件，有具体的格式要求

### 5.1.1. genome mask文件
用注释的重复序列repeats.gff为基因组做genome mask，把重复序列所在位置的序列替换成N，减少假基因注释的计算量。获得masked的基因组文件genome.repeatmask.fa。

### 5.1.2. tblastn.out文件
用`tblastn`为mask的基因组和注释的基因组蛋白序列做比对，比对结果用作PseudogenePipeline的输入。

1. 运行

```
makeblastdb -in genome.repeatmask.fa -dbtype nucl -out sample # 建库
tblastn -db dample -query sample.proteins.fa -num_threads 4 -out tblastn.out -outfmt 6 # tblastn比对，输出tblastn.out
```

2. notes
- 可能由于基因组大(6G)，多线程会中断，只能单线程，可以把蛋白文件sample.proteins.fa切分成多份，分别单线程tblastn，然后把结果合并。
- 注意蛋白文件和基因组文件的序列id不能有空格，否则自动去除空格和空格后内容。

### 5.1.3. sample.gff3文件
gff3文件格式类似普通的gff3格式，共九列。但有很具体的要求，主要是对第九列内容的要求，可参照软件给的例子进行增添修改以符合要求。

具体格式要求：
- gene行，第九列包含`Name=信息`，值可与`ID=信息`一致
- mRNA行，第九列包含`longest=1`或者`longest=0`信息，用于可变剪切筛选，1代表最长可变剪切，0代表非最长可变剪切。
- mRNA,exon,CDS,UTR行，第九列都包含`pacid=信息`，值可与`Parent=信息`一致，一个gene内部的所有pacid值应相同。

## 5.2. 用PseudogenePipeline鉴定假基因
1. 参数文件parameter_file示例

```
b_out=/path/to/tblastn.out # 前面准备的tblastn.out文件，<tblastn_output_file>
p_seq=/path/to/sample.proteins.fa # 蛋白序列，<protein_sequence_FASTA_file>
g_seq=/genome.repeatmask.fa # 基因组序列（最好是mask过的） <genome_sequence_FASTA_File>
b_filter=y # 是否做blastout的filter，<filter blastouput? y/n>
b_data=tabular # <define type of blastdata: tabular, blastall, blastplus>
p_codes=/path/to/PseudogenePipeline-master/_pipeline_scripts #<path to ../_pipeline_scripts>
o_codes=/path/to/PseudogenePipeline-master/_pipeline_scripts #<path to ../_pipeline_scripts>
f_dir=/path/to/fasta-36.3.8h/bin #<path to FASTA package install>
f_prog=tfasty36 #<tfasty program name>
blosum=/path/to/PseudogenePipeline-master/_example_files/blosum50.matrix #<path to blosum matrilx file>
gff=/path/to/sample.gff3 # gff3注释文件 <gff_file>
repCut=300 #<RepeatMasker cutoff; integer, recommended 300>
repDiv=30 #<RepeatMakser divergence; integer, recommended 30>
```

2. 运行
- `python2 PseudogenePipeline-master/_wrapper_scripts/CombinedPseudoWrapper.py parameter_file`

## 5.3. PseudogenePipeline运行结果
### 5.3.1. 结果文件
在生成的结果目录result下，保存了结果文件，包括三个文件夹。result/_intermediate保存着中间文件，result/_logs保存着log文件，result/_results保存着高置信并通过RepeatMasker过滤的鉴定出来的假基因结果文件。

#### 5.3.1.1. _results文件夹中的文件
result/_results中的结果文件包括以下文件，每个文件第一行都是以`#python`开始的生成此文件的命令，其余行才保存了假基因的信息：

1. sample.4col.true.RMfilt.hiConf
- 第一行`#python`开头的是生成此文件运行的命令。
- 其余行是假基因和位置信息，共四列。第一列是假基因对应的真基因的ID（gene38168）加上假基因位置信息（ctg947|625093-625785）加上其他比对信息，分号分隔；第二列是染色体ID；第三第四列是start和end的坐标。
- `gene38168;ctg947|625093-625785;336-566:693-4;4|0|0|0`表示在染色体`ctg947`的`625093-625785`位置有一个假基因pseudogene。
- 对应的真基因是`gene38168`；`336-566`和`693-4`分别是指真基因序列和假基因序列比对上的区域；比对的信息是从tblastn的比对结果中获取的。
- 真基因`gene38168`的**氨基酸**序列的`336-566`位置。
- 假基因的`693-4`位置是指在染色体`ctg947`的`625093-625785`位置的假基因核酸序列上的第4到693个碱基，由于起始位置比终止位置大，所以是4-693位置的反向互补序列。
- 下面是文件的部分内容。

```
gene38168;ctg947|625093-625785;336-566:693-4;4|0|0|0	ctg947	625785	625096
gene04393;ctg8706|272998-273192;228-290:190-4;2|0|1|0	ctg8706	273187	273001
gene10546;ctg5903|105682-105807;115-156:126-4;1|0|0|0	ctg5903	105807	105685
```

2. sample.4col.true.RMfilt.hiConf.cdnm
- 同文件sample.4col.true.RMfilt.hiConf的内容基本一致，区别在于第一列的内容，这里是假基因ID（Ps000001）和对应的真基因ID（gene38168）。
- 第一行`#python`开头的是生成此文件运行的命令。
- 其余行是假基因和位置信息，共四列。
- 下面是文件的部分内容。

```
Ps000001_gene38168	ctg947	625785	625096
Ps000003_gene04393	ctg8706	273187	273001
Ps000004_gene10546	ctg5903	105807	105685
```

3. sample.4col.true.RMfilt.hiConf.cdnm.gff
- 第一行`#python`开头的是生成此文件运行的命令。
- 其余行是假基因的注释信息，gff格式，共九列。
- 下面是文件的部分内容。

```
ctg947	ShiuLab	pseudogene	625096	625785	.	-	.	ID=Ps000001;Name=Ps000001_gene38168;Derives_from=gene38168;Note=pseudogene_evidence_code_4,0,0,0
ctg8706	ShiuLab	pseudogene	273001	273187	.	-	.	ID=Ps000003;Name=Ps000003_gene04393;Derives_from=gene04393;Note=pseudogene_evidence_code_2,0,1,0
ctg5903	ShiuLab	pseudogene	105685	105807	.	-	.	ID=Ps000004;Name=Ps000004_evm.model.ctg10546.26;Derives_from=gene10546;Note=pseudogene_evidence_code_1,0,0,0
```

4. sample.4col.true.fa.RMfilt.hiConf
- 第一行`#python`开头的是生成此文件运行的命令。
- 其余行是假基因的序列信息，fasta格式，序列ID是假基因对应的真基因的ID（gene38168）加上假基因位置信息（ctg947|625093-625785）加上其他比对信息，分号分隔。序列ID同文件sample.4col.true.RMfilt.hiConf的第一列内容一致。
- 下面是文件的部分内容。

```
>gene38168;ctg947|625093-625785;336-566:693-4;4|0|0|0
TTGCAGGTTGCAATTTTGATGTCCAAGGACCAAGCACTATATTACGCATGGTTGTTCTCCCTAATAATTTATGAATTACTTAGAAAACCTACCTTGCGCAATGTAGAAGCGTTAGTGCAAACATGGCTCCGTGTCACACAATGCATGTCACGGTCAATTAACTTTAAGAAGGTAGCAAATCGTAAAAGGAGTTGGGAGCGTAGGAGAGCATGAGCTTATCCATGACCCCAGAGGTTTCTTGAGTGCTTTTTGTTGAGTGCATATAATGCAAACATGTTCAAATGTCGCTTGTGAGTGAGTAGAGAAACCTTCCAATACGTATGTGGCAAGGTCTCTCTTGCAATGAAGAAGGTGAGCACAAACATGCGTGTTCCACTTAGTATGGAAACAAAACTGGCAATAGCTCTCAGTAGGCTAGCTTTAGGTGCAGGGCTTGAGATGTTGGGTGGTTTGCATGGCTGTGCAAAGAGCACAAGTTGTAAAGTAGTGATTGATTTTTGTAAGGCAATTGTTACATCAGGTCTTAGGGATTTGTACATTAGGTGGCCTGCCCTCTCATGGCTAGAGACACTAGCAAGTGAGTTTCAAGCTTCAAGATGCATCCCCTTTGTTGCTAGAGCCATAGACAGTTCTCACATCCCTATCATCGCCCCAAGAGACAATCACGTAGATTATTTCAATTGAAAAGGG
>gene04393;ctg8706|272998-273192;228-290:190-4;2|0|1|0
TATATGTCGTGATATCACAGCAATCAATCTATCATTAGAGAACTTAGATGATTTGTATAAGGTGTGTGGGTTCATACATGGCTTGGACATGAAGTACAAGCACAACATACGTACTCAAAACCCCAAAGACCCCATAGATGCTATTTAAGGTTCTCAAATCTATAATGACTCTCTTGATAAGAGGAGC
>gene10546;ctg5903|105682-105807;115-156:126-4;1|0|0|0
GGCATGGATGATGACCAATTTCTTTCTTTCTTGGACCTCAAGCAGAGGAGGTGGTATGAGGACCGCCATTGCTCCAAAGCATAGTCACGTTGTTCCACTCCTTCAGAGTCTCCAACTGCTTTG
```

5. sample.4col.true.fa.RMfilt.hiConf.cdnm
- 第一行`#python`开头的是生成此文件运行的命令。
- 其余行是假基因的序列信息，fasta格式，序列ID是假基因ID（Ps000001）和对应的真基因ID（gene38168）。序列ID同文件sample.4col.true.RMfilt.hiConf.cdnm的第一列内容一致。
- 下面是文件的部分内容。

```
>Ps000001_gene38168
TTGCAGGTTGCAATTTTGATGTCCAAGGACCAAGCACTATATTACGCATGGTTGTTCTCCCTAATAATTTATGAATTACTTAGAAAACCTACCTTGCGCAATGTAGAAGCGTTAGTGCAAACATGGCTCCGTGTCACACAATGCATGTCACGGTCAATTAACTTTAAGAAGGTAGCAAATCGTAAAAGGAGTTGGGAGCGTAGGAGAGCATGAGCTTATCCATGACCCCAGAGGTTTCTTGAGTGCTTTTTGTTGAGTGCATATAATGCAAACATGTTCAAATGTCGCTTGTGAGTGAGTAGAGAAACCTTCCAATACGTATGTGGCAAGGTCTCTCTTGCAATGAAGAAGGTGAGCACAAACATGCGTGTTCCACTTAGTATGGAAACAAAACTGGCAATAGCTCTCAGTAGGCTAGCTTTAGGTGCAGGGCTTGAGATGTTGGGTGGTTTGCATGGCTGTGCAAAGAGCACAAGTTGTAAAGTAGTGATTGATTTTTGTAAGGCAATTGTTACATCAGGTCTTAGGGATTTGTACATTAGGTGGCCTGCCCTCTCATGGCTAGAGACACTAGCAAGTGAGTTTCAAGCTTCAAGATGCATCCCCTTTGTTGCTAGAGCCATAGACAGTTCTCACATCCCTATCATCGCCCCAAGAGACAATCACGTAGATTATTTCAATTGAAAAGGG
>Ps000003_gene04393
TATATGTCGTGATATCACAGCAATCAATCTATCATTAGAGAACTTAGATGATTTGTATAAGGTGTGTGGGTTCATACATGGCTTGGACATGAAGTACAAGCACAACATACGTACTCAAAACCCCAAAGACCCCATAGATGCTATTTAAGGTTCTCAAATCTATAATGACTCTCTTGATAAGAGGAGC
>Ps000004_gene10546
GGCATGGATGATGACCAATTTCTTTCTTTCTTGGACCTCAAGCAGAGGAGGTGGTATGAGGACCGCCATTGCTCCAAAGCATAGTCACGTTGTTCCACTCCTTCAGAGTCTCCAACTGCTTTG
```

#### 5.3.1.2. _intermediate文件夹中的文件
`result/_intermediate`中的结果文件包括以下文件，许多文件第一行都是以`#python`开始的生成此文件的命令，其余行才保存了信息：

1. sample.fullyFiltered.disable_count
- 第一行都是以`#python`开始的生成此文件的命令
- 保存着假基因和对应真基因的氨基酸的比对好的序列

2. sample.fullyFiltered.disable_count.RMfilt
- 前两行都是以`#python`开始的生成此文件的命令
- 保存着假基因和对应真基因的氨基酸的比对好的序列

### 5.3.2. notes
- 注释到的假基因之间是没有重叠的，一个位置的序列只能被注释为一个假基因。
- 建议做一个长度的过滤，比如长度大于300bp的假基因才被鉴定为假基因，进入到下一步的分析流程。
- result/_intermediate文件夹中保存着额外的文件，比如sample.fullyFiltered.disable_count.RMfilt文件保存着假基因和对应真基因比对好的蛋白质序列。

## 5.4. PseudogenePipeline运行的一个例子
运行相关数据：
- 6Gb大小的蕨类基因组，跑了70h。
- 70000个注释的真基因，软件结果中有222,372个假基因，其中长度大于300bp的为93594条。

# 6. 假基因后续分析
鉴定假基因后，可以通过进一步的分析来获取更多认识。

## 6.1. 批量计算pseudogene和功能性gene之间的Ks
通过计算pseudogene和功能性基因（真基因）间的Ks，可以判断是否与其他进化事件的Ks重合（比如WGD），也可以通过Ks计算假基因产生的时间的分布。

### 6.1.1. 背景
1. Ks只能计算ORF区域的（即完整密码子，3的倍数的碱基），并遇到stop codon停止。
2. 根据假基因的注释信息，找到pseudogene和对应真基因的序列，并把pseudogene和对应真基因的ORF区域提取出来。
3. 若pseudogene比对到了intron区域，删除这部分，并从真基因密码子首位开始匹配pseudogene，提取完整密码子（3的倍数个碱基）。
4. 只需要比对上的密码子区域，并要求3的倍数的碱基；而假基因常常包含许多中间的终止密码子，所以这些中间终止密码子和对应位置的真基因的三个碱基也需要删除。（还有一种策略是从第一个终止密码子处截断，之后的序列全都不要，但这种策略常常会使得提取出来的序列太短，造成更大误差，所以不推荐。）

准备以下三个输入文件，就可以根据 **博客：批量计算Ka和Ks：https://yanzhongsino.github.io/2022/09/07/bioinfo_Ks_batch.calculation.Ks** 进行Ks的批量计算。

### 6.1.2. 准备sample.pairs文件
sample.pairs保存着两列数据，tab分隔，第一列是假基因的ID，第二列是对应的真基因的ID。

可以用下面的命令从结果文件中提取假基因和对应的真基因的ID，获得sample.pairs。

`cat ./_results/sample.4col.true.RMfilt.hiConf.cdnm  |grep -v "#"|sed -e "s/\t.*//g" -e "s/_/\t/" >sample.pairs `

### 6.1.3. 准备用于计算Ks的CDS序列文件(all.cds)和氨基酸序列文件(all.pep)
用于计算Ks的CDS序列文件需要包括所有假基因和对应真基因的CDS序列，用于计算Ks的氨基酸序列文件需要包括所有假基因和对应真基因的氨基酸序列。

注意：all.cds和all.pep可以包含多余的序列但不能少。因为是根据sample.pairs文件的指定序列对计算Ks的。

1. 提取cds序列
- 用cufflinks的gffread工具从gff文件提取CDS序列: `gffread sample.4col.true.RMfilt.hiConf.cdnm.gff -g genome.fa -x pseudo.cds`
- 另一种方法直接从结果文件中提取cds序列：`cat sample.4col.true.fa.RMfilt.hiConf.cdnm |grep -v "#" >pseudo.cds`
2. 合并假基因CDS序列和基因组所有真基因的CDS序列（sample.cds）到一个文件all.cds
- `cat pseudo.cds sample.cds > all.cds`
3. 用cufflinks的gffread工具从gff文件提取pep序列
- `gffread sample.4col.true.RMfilt.hiConf.cdnm.gff -g genome.fa -y pseudo.pep`
4. 合并假基因氨基酸序列和基因组所有真基因的氨基酸序列（sample.pep）到一个文件all.pep
- `cat pseudo.pep sample.pep > all.pep`

### 6.1.4. 计算Ks
准备好以上三个文件：sample.pair, all.cds, all.pep 即可用ParaAT.pl和KaKs_Calculator2.0两个软件，参考**博客：批量计算Ka和Ks：https://yanzhongsino.github.io/2022/09/07/bioinfo_Ks_batch.calculation.Ks**的步骤计算假基因和对应真基因间的Ks，从而获知遗传上的分化及分布。

## 6.2. 检测假基因和共线性区块的关系
在2022年08月在GBE上发表的荷叶铁线蕨基因组的文章（https://academic.oup.com/gbe/article/14/8/evac127/6659224）中的一个应用，判断是否是pseudogene造成共线性blocks变短，用到的方法细节记录在此。

1. 分析MCScanX结果中找到的共线性blocks，看blocks是否有成对出现在同一个contig上的，两对出现在同一contig上的blocks是否分别有共线性对应。
- 结果是不存在符合条件的两对blocks。
- 原因是contig的长度短造成的，contig长度在1-2Mbp，一个block含5个及以上基因，就占据了一条contig的全长，所以同一个contig上有多个block出现的概率变小。
2. 在最长的blocks里找pseudogene造成blocks内部gene无共线性的例子。即block内部无共线性的gene对应block上对面位置是否有这个gene的pseudogene存在。
- 结果是在超过7个gene的blocks里找到5个blocks存在这种情况。

```
Alignment 288: ctg4124-ctg7158：Ane23910-Ane23911之间有Ane41918的pseudogene(ctg4124 6381349 6381618)；Ane23909-Ane23910之间有Ane41900的pseudogene（重排）；
Alignment 102： ctg11329-ctg6741：Ane66914-Ane66915之间有Ane38759的pseudogene；Ane66920-Ane66921之间有Ane38765的pseudogene(ctg11329        2818519 2817964)；
Alignment 139: ctg1252-ctg4677：Ane27375-Ane27376之间有Ane07597的pseudogene；
Alignment 422: ctg7700-ctg9700：Ane46157-Ane461158之间有Ane57929的pseudogene（重排）(ctg7700 2107090 2106980)；
Alignment 396: ctg6709-ctg8004：Ane38599-Ane38600之间有Ane47496的pseudogene（重排）(ctg6709 1117953 1117655)；
```

## 6.3. 非批量计算Ks
如果只有少量样本对（关注的假基因和对应真基因）需要计算Ks，可以手动进行。比如上面的共线性区块分析中找到的5个blocks中的少量样本对。

### 6.3.1. 提取序列
通过PseudogenePipeline运行结果的比对信息提取假基因和对应的真基因比对上的ORF区域，并提取3的倍数的碱基，以计算Ks。

1. 拿`_results/sample.4col.true.fa.RMfilt.hiConf`中的一条pseudogene序列的ID举例说明位置的含义

- `gene38168;ctg947|625093-625785;336-566:693-4;4|0|0|0	ctg947	625785	625096`表示在染色体`ctg947`的`625093-625785`位置有一个假基因pseudogene。
- 对应的真基因是`gene38168`；`336-566`和`693-4`分别是指真基因序列和假基因序列比对上的区域；比对的信息是从tblastn的比对结果中获取的。
- 真基因`gene38168`的**氨基酸**序列的`336-566`位置。
- 假基因的`693-4`位置是指在染色体`ctg947`的`625093-625785`位置的假基因核酸序列上的第4到693个碱基，由于起始位置比终止位置大，所以是4-693位置的反向互补序列。

2. 根据pseudogene的序列位置和对应的真基因的位置信息，提取两者比对上的cds序列
- 先从`_results/sample.4col.true.fa.RMfilt.hiConf`中提取出`gene38168`对应的假基因pseudogene；
- 根据假基因的比对位置`693-4`提取染色体`ctg947`的`625093-625785`位置的`4-693`的核酸，由于起始位置比终止位置大，所以是`4-693`位置的反向互补序列。
- 再根据真基因的ID`gene38168`提取真基因的cds序列；
- 根据真基因的蛋白比对位`336-566`，通过计算起始位置$$336\*3-2=1008$$，和终止位置$$566\*3=1698$$，找到cds对应的`1008-1698`位置提取出来。

3. 比对和删改
- 把两者序列保存在一个fasta文件，并用mafft/muscle等工具做align；
- 以真基因序列为参考，根据align结果对提取的假基因序列进行删除和修改。比如删除比对到intron的部分；若是假基因中包含indel，则在indel处找到上一个密码子的第三位，在这个位置删除对应密码子；最终使得两条序列都包含3的倍数的对应密码子的orf序列。
- 删改好后再做一次align，每个假基因和对应真基因保存成一个文件；

### 6.3.2. 计算Ks
1. 改格式
- `AXTConvertor gene_pseudo.fa gene_pseudo.axt`
- KaKs_Calculator的改格式工具，改比对后的fas格式（gene_pseudo.fa）为axt（gene_pseudo.axt）。
- axt格式包括三行，第一行两个序列ID之间用短横杠-相连，第二行第一条序列，第三行第二条序列。

2. 计算Ks
- `KaKs_Calculator -i gene_pseudo.axt -o gene_pseudo.kaks -m PN`
- 计算KaKs，用PN模型；结果就在gene_pseudo.kaks文件中。

3. 合并所有计算的ks结果

| gene-pseudo              | Ka        | Ks        | Ka/Ks    |
| ------------------------ | --------- | --------- | -------- |
| Ane07597-Ane07597.pseudo | 0.166453  | 0.830702  | 0.200376 |
| Ane38759-Ane38759.pseudo | 0.0068327 | 0.0579878 | 0.11783  |
| Ane38765-Ane38765.pseudo | 0.0181923 | 0.0594878 | 0.305815 |
| Ane41900-Ane41900.pseudo | 0.356588  | 1.23697   | 0.288276 |
| Ane41918-Ane41918.pseudo | 0.0304071 | 0.033778  | 0.900203 |
| Ane47496-Ane47496.pseudo | 0.025231  | 0.210919  | 0.119624 |
| Ane57929-Ane57929.pseudo | 0.0380011 | 0.0703679 | 0.540034 |

- 从结果中看出，Ane38759，Ane38765，Ane579290三个基因和它们的假基因之间的Ks接近WGD分析中计算的WGD所在的Ks=0.06；可以作为共线性blocks内部假基因的证据。

# 7. references
1. wiki:pseudogene:https://en.wikipedia.org/wiki/Pseudogene
2. PseudogenePipeline github：https://github.com/ShiuLab/PseudogenePipeline
3. 2022年08月在GBE上发表的荷叶铁线蕨基因组的文章：https://academic.oup.com/gbe/article/14/8/evac127/6659224

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>