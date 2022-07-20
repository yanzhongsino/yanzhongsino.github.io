---
title: 基因组注释：LTR的注释，和计算基因组中LTR的插入时间
date: 2022-07-20
categories:
- omics
- genome
tags:
- genome
- LTR

description: 计算基因组中LTR的插入时间的分布，推断LTR产生历史。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=26834823&auto=1&height=32"></iframe></div>

# 1. LTR简介

1. LTR

关于重复序列和LTR的相关知识可以参考博客：基因组注释（一）：重复序列注释：https://yanzhongsino.github.io/2021/08/02/omics_genome.annotation_repeat/。

LTR是末端重复序列，产生LTR时，两端的重复序列是一致的，随着时间产生divergence。
期望进化时间越长，divergence越大。所以可以通过末端两段序列的divergence来判断产生LTR的时间。

# 2. 注释LTR
用LTR_FINDER_parallel和ltrharvest分别预测LTR，然后基于预测结果用LTR_retriever鉴定和确认LTR。

## 2.1. LTR_FINDER_parallel预测LTR

`LTR_FINDER_parallel -seq genome.fa -threads 10 -harvest_out -size 1000000 -time 300 -w 2 -C -D 15000 -d 1000 -L 7000 -l 100 -p 20 -M 0.85`

- 预测LTR,生成genome.fa.finder.combine.scn
- 其中，-w 2 -C -D 15000 -d 1000 -L 7000 -l 100 -p 20 -M 0.85参数部分是作者推荐；
- -size, -time, -try1三个参数作者不建议修改
- -harvest_out指定输出harvest格式的结果

## 2.2. ltrharvest预测LTR
1. 建基因组索引

`gt suffixerator -db genome.fa -indexname index/sample -tis -suf -lcp -des -ssp -sds-dna`

2. 预测LTR

`gt ltrharvest -index index/sample -minlenltr 100 -maxlenltr 7000 -mintsd 4 -maxtsd 6 -motif TGCA -motifmis 1 -similar 85 -vic 10 -seed 20 -seqids yes > genome.harvest.scn`

生成genome.harvest.scn

## 2.3. 合并预测结果【可选&不推荐】

`cat genome.harvest.scn genome.fa.finder.combine.scn > genome.rawLTR_merge.scn`

因为LTR_FINDER_parallel和ltrharvest的预测结果都是一行一条信息，可以直接合并。

## 2.4. LTR_retriever鉴定LTR

`LTR_retriever -genome genome.fa -inharvest genome.harvest.scn -infinder genome.fa.finder.combine.scn -threads 12 -u 4.79e-9`

- 这步骤耗时长。运行LTR_retriever默认会计算LAI指数，不用单独运行LAI。
- 生成的genome.out.LAI包含整个genome和每个contig的LAI指数。
- 如果前一步合并了多个LTR预测结果，则只需要在`-inharvest genome.rawLTR_merge.scn`输入合并结果。
- 生成的文件genome.fa.pass.list最后一列Insertion_Time是计算的每一个LTR的插入时间。
- 插入时间默认用的碱基替代速率default值是水稻的1.3e-8，可以用-u参数设置碱基替代速率，比如设置成蕨类的4.79e-9。

# 3. LTR insert time 画图

```R
library(ggplot2)
library(ggpmisc)
data <- read.table("insert_time.txt",header=T)
p <- ggplot(data, aes(insert.time)) + geom_density(size=1,color="black")+xlab("Synonymous substitution rate(Ks)")+ylab("Percent of Total Paralogs")+theme(panel.grid=element_blank())+theme(panel.border = element_blank())+theme(axis.line = element_line(size=0.8, colour = "black"))+scale_y_continuous(breaks=seq(0, 2.5, 0.2))+xlim(0,3)
pb <- ggplot_build(p)
pic <- p + stat_peaks(data = pb[['data']][[1]], aes(x=x, y=density), geom= 'text', color="red",vjust=-0.5)
ggsave(file="ltrd.peak.pdf",plot=pic,width=10,height=5)
```

ltr的insert time的峰值在4.614*10^5。

把>1945000的时间提取出来，用T-Test检验获得insert time的95% CI置信区间在4590102-4639346之间。

# 4. references
1. github：https://github.com/oushujun/LTR_retriever


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>