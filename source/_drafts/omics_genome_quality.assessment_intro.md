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

# 基因组质量评估
随着越来越多的组学测序数据的产生，基因组组装和注释变得越来越常见。

基因组组装和注释结果的质量评估对加强下游分析的科学性是非常有必要的。

# 评估标准
Earth Biogenome Project（EBP）在2021年3月发布了4.0版本的核基因组组装质量标准报告。

可参考：https://www.earthbiogenome.org/assembly-standards。

## 生物分组
评估标准把生物分成三组，对不同组的生物提出了不同的标准：
1. 有足够DNA和材料可用的真核生物：6.C.Q40标准。
- 连续性（contiguity）：contig N50和scaffold N50达到Mbp级别
- 错误率（error rate）：小于1/10000的错误率
- <5% false duplications
- >90% kmer completeness
- >90%的序列可以align到候选染色体上
- >90%的单拷贝保守基因是完整且单拷贝的，例如用BUSCO评估
- 来自同一个组织的>90%转录本可以mapping到基因组
- ...

2. DNA和材料有限的物种（例如单个个体的DNA<100ng）：4.5.Q40标准
- 连续性（contiguity）：contig N50和scaffold N50达到10kbp级别
- 错误率（error rate）：小于1/10000的错误率

3. 不可培养的单细胞真核生物
- 类似宏基因组学的标准，根据原核生物群落的经验来确定的。

## 附加要求
由于自动化组装程序几乎都会在组装结果中保留一些错误，所以提出了一套质量控制标准，包括：
1. 将污染源和其他物种（例如共生物种/寄生物种）的序列从目标物种的序列中分离去除。
2. 识别初级组装（单倍型或假单倍型），应该是说尽量组装到染色体水平的意思。
3. 分离去除和明确鉴定细胞器基因组。
4. 只有A,C,G,T,N碱基，序列不应以Ns开头或结尾。

此外，还鼓励：
1. 识别原始数据和生成的组装之间的不一致，以定位和消除结构错误（错误连接，丢失连接，错误重复）。
2. 染色体的识别和命名，尤其是性染色体。
3. 与已知的核型保持一致。

## 提交要求
对于达到EBP要求的参考基因组，还需提交到公开数据库INSDC（GenBank/EMBL/DDBJ）以开放给科学同行。

- 提交到公开数据库的基因组需要连接到一个BioProject，并建议以下图的结构进行组织和维护。
- 同时将基因组的物种分配到NCBI分类数据库的“txid”条目，与分类学上有效的物种进行关联。
- 

# 评估参数和评估工具


评估组装的基因组的常用方法有：
1. mapping rates：包括pacbio，illumina，RNA-seq reads等数据都可以映射回基因组，然后计算比对率来判断基因组的精确度。
2. QUAST：
3. BUSCO：基因组和预测的蛋白组都可以用BUSCO评估基因组的完整度(completeness)。
4. LAI：通过LTR组装指数评估基因组的连贯性(continuity)。

# genome assembly
## DNA-seq reads(illumina and Pacbio) mapping percentages

## genome assembly metrics
QUAST
Genometools
FastaSeqStats

## Kmer completeness
Merqury


## conserved gene space assessment
BUSCO

## intact LTR reconstruction
LAI

## contaminations specially for the smaller contigs
Blobtools


# genome annotation
## RNA-seq reads mapping percentages

2. QUAST
quast.py genome.fsa -g Adiantum.nelumboides.gff -1 ./illumina/Adiantum.reniforme_illumina.clean_R1.fq.gz -2 ./illumina/Adiantum.reniforme_illumina.clean_R2.fq.gz -t 12 --large -o an_quast

3. BUSCO
We also performed BUSCO evaluation to examine completeness of the assembly with the Eukaryota_odb10 database.

4. LAI
The LTR assembly index10 was used to assess continuity

# references
1. https://www.earthbiogenome.org/assembly-standards
