---
title: 基因组质量评估：（五）mapping法：3. 统计mapped reads的深度分布
date: 2022-07-27
categories:
- omics
- genome
- quality assessment
tags:
- quality assessment
- genome
- organelle
- plastome
- mitogenome
- mapping
- sam
- bam
- depth
- samtools

description: mapping法评估基因组组装质量。mapping法是指把测序的reads（包括Pacbio，Illumina，RNA-seq 等reads）映射回组装好的基因组，评估mapping rate，genome coverage，depth分布等指标，用这些指标评估基因组组装质量。这篇文章简单概述了mapping法的其中一个评估指标：depth，主要介绍了统计mapped reads的深度分布的一般方法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2117115&auto=1&height=32"></iframe></div>

# 1. 前情提要：mapping
此篇博客在已通过mapping获得SAM/BAM文件的基础上进行分析，

1. 关于mapping获得SAM/BAM文件的操作可以参考博客：

[基因组质量评估：（五）mapping法：1. 简介](https://yanzhongsino.github.io/2022/07/23/omics_genome_quality.assessment_mapping_intro/)

# 2. 深度（depth）分布
- 在测序是随机分布的情况下，期望在基因组的所有染色体上，mapped reads的depth是均匀分布的。
- 把reads回mapping到组装好的基因组，通过计算depth，观察depth在基因组上的分布来判断组装的质量。

# 3. 统计深度（depth）
## 3.1. samtools统计
`samtools`的`depth`调用`mpileup`模块进行mapped reads的深度统计。

### 3.1.1. samtools depth统计
1. samtools depth统计
- `samtools depth illumina.bam > depth.out`

2. 输出
- depth.out有三列数据，tab分隔。
- 第一列参考序列（染色体）名称；
- 第二列位置；
- 第三列比对上的reads数量（即depth）。

### 3.1.2. samtools mpileup统计
1. 运行
- `samtools mpileup -A -Q sample.bam > mpileup.out`

2. 输出
- mpileup.out共有6列数据，tab分隔。
- 第一列参考序列（染色体）名称；
- 第二列位置；
- 第三列参考序列的碱基；
- 第四列比对上的reads数量（即depth）；
- 第五列比对上的情况；
- 第六列比对上的碱基的质量。

3. 第五列比对上的情况，具体解释：
- \*表示模糊碱基
- 大写表示在正链不匹配
- 小写表示在负链不匹配
- \^表示匹配的碱基是一个reads的开始，\^后紧跟的ascii码减去33代表比对质量，修饰的是后面的碱基，后面紧跟的碱基代表该read的第一个碱基
- \$代表一个read的结束，该符号修饰前面的碱基
- 正则表达式`+[0-9]+[ACGTNacgtn]+`代表在该位点后插入的碱基。举例中chr1的2003928A后面有个+6GGGCCG，很可能是indel
- 正则表达式`-[0-9]+[ACGTNacgtn]+`代表在该位点后缺失的碱基

### 3.1.3. samtools mpileup和samtools depth的差异
1. 差异
- `samtools depth`是调用了`samtools mpileup`进行的。
- 所以`samtools mpileup`可以设置更多参数进行过滤，也有一些默认的过滤参数；而`samtools depth`没有进行过滤。

2. `samtools mpileup`的默认过滤
- `samtools mpileup`默认过滤掉测序质量<13的碱基；
- `samtools mpileup`默认过滤掉PE reads中比对异常的reads（包括双端都比上，但是两条配对reads之间的比对距离明显偏离了插入片段的长度分布，或者一端比对上而另一端没比对上）。除非加上-A参数保留异常reads，才与`samtools depth`一致。

## 3.2. 其他方法
除了这里提到的统计深度的方法，还有mosdepth,bamdst,quallimap等软件可以实现，在接下来的博客里会详细介绍。

# 4. 深度分布作图
统计获得滑窗或者位点的深度后，可以作点图/折线图来直观地观察深度在整个染色体上的分布。

# 5. references
1. samtools mpileup和samtools depth计算的depth不同的解释：https://zhuanlan.zhihu.com/p/73208822


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>