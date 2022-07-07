---
title: 基因组质量评估：（二）统计mapped reads的深度分布
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

description: 记录基因组评估的方法，用测序reads（包括pacbio，illumina，RNA-seq reads） mapping回
基因组，得到mapping rates，mapping rates越高代表基因组的。
---

<div align="middle"><music URL></div>

## 基因组评估的方法

把reads回mapping到组装好的基因组，通过计算depth，观察depth的分布来判断组装的质量。

期望在基因组的所有染色体上，depth是均匀分布的。

### BWA mapping
1. 工具
BWA-MEM+samtools

1. 建索引
`bwa index ref.fa`

2. bwa mapping
`bwa mem -t 4 ref.fa R1.clean.fq r2.clean.fq | samtools sort -@ 4 -m 4G > illumina.bam &`

3. samtools mpileup统计

`samtools mpileup -A -Q sample.bam > mpileup.out`

mpileup.out共有6列数据，tab分隔。

第一列参考序列（染色体）名称，第二列位置，第三列参考序列的碱基，，第四列比对上的reads数量（即depth），第五列比对上的情况，第六列比对上的碱基的质量。

第五列比对上的情况，具体解释：
- *表示模糊碱基
- 大写表示在正链不匹配
- 小写表示在负链不匹配
- ^表示匹配的碱基是一个reads的开始，^后紧跟的ascii码减去33代表比对质量，修饰的是后面的碱基，后面紧跟的碱基代表该read的第一个碱基
- $代表一个read的结束，该符号修饰前面的碱基
- 正则表达式式+[0-9]+[ACGTNacgtn]+代表在该位点后插入的碱基。举例中chr1的2003928A后面有个+6GGGCCG，很可能是indel
- 正则表达式’-[0-9]+[ACGTNacgtn]+’代表在该位点后缺失的碱基

4. samtools depth统计
`samtools depth illumina.bam > depth.out`

depth.out有三列数据，tab分隔。

第一列参考序列（染色体）名称，第二列位置，第三列比对上的reads数量（即depth）。


5. samtools mpileup和samtools depth的统计差异
`samtools depth`是调用了`samtools mpileup`进行的。

`samtools mpileup`可以设置参数进行过滤，也有一些默认的过滤参数；而`samtools depth`没有进行过滤。

与`samtools depth`不同的点主要是：
- `samtools mpileup`默认过滤掉测序质量<13的碱基；
- `samtools mpileup`默认过滤掉PE reads中比对异常的reads（包括双端都比上，但是两条配对reads之间的比对距离明显偏离了插入片段的长度分布，或者一端比对上而另一端没比对上）。除非加上-A参数保留异常reads，才与`samtools depth`一致。


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
1. samtools mpileup和samtools depth计算的depth不同的解释：https://zhuanlan.zhihu.com/p/73208822