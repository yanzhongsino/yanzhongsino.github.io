---
title: 推测基因流的方法简介
date: 2021-11-12 16:00:00
categories: 
- bio
- bioinfo
tags: 
- gene flow
- population networks
description: 介绍生物信息学数据库。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1296539261&auto=1&height=32"></iframe></div>

# 检测杂交/基因流
### hybridization/introgression

1. Dsuite
2. HyDe
3. PhyloNetworks
4. TreeMix
5. PhyloNet


# Dsuite检测ABBA-BABA
Dsuite是C编写的，需要编译。


### 输入文件
- sample.snp.vcf.gz：可用bgzip压缩
- sets.txt：包括两列，第一列个体名，第二列物种名/居群名；outgroup个体的第二列写Outgroup，可有多个），这里要注意第二列物种名不能包含短横杠-和句点.这些字符，否则Dsuite Fbranch模块报错运行解析不了树文件，Dsuite Dtrios不会报错。
- speciestree.newick: 这个文件是可选的optional，是Newick格式的居群树文件，去掉支持率，枝长可要可不要。


### 运行

1. Dsuite Dtrios
`Dsuite Dtrios sample.snp.hf.f.vcf.gz sets.txt -t speciestree.newick -o sample`
花费时间：3-4h；
生成sets_BBAA.txt, sets_combine_stderr.txt, sets_combine.txt, sets_Dmin.txt, sets_tree.txt, sets.txt 等文件

- -o sample #指定输出文件前缀，默认是sets

2. Dsuite Fbranch
`Dsuite Fbranch speciestree.newick sets_tree.txt >fbranch.out`
sets_tree.txt是Dtrios模块得到的结果文件

生成fbranch.out文本文件

Dsuite/utils/dtools.py fbranch.out speciestree.newick --outgroup Outgroup --use_distances --dpi 1200 --tree-label-size 20
画Fbranch的图，得到fbranch.svg和fbranch.png；--dpi设置png分辨率，--outgroup设置外类群，可以在fbranch.out里看外类群名称，--tree-label-size设置树标签大小。

# PhyloNetworks

分析

```julia
using PhyloNetworks
astralfile= joinpath("astral.tre") #导入树文件
astraltree = readMultiTopology(astralfile) # read tree file
raxmlCF = readTableCF("tableCF.csv") # read in the file and produces a "DataCF" object
net0 = snaq!(astraltree,raxmlCF, hmax=0, filename="net0", seed=1234)
```



```julia
using PhyloNetworks
net0 = readTopology("net0.tre") #读入net
rootatnode!(net0,"L446") #重新定根，然后画图
using PhyloPlots # to visualize networks
using RCall      # to send additional commands to R like this: R"..."
imagefilename = "net0.pdf" #保存为pdf等格式
R"pdf"(imagefilename) # starts image file
plot(net0, :R, showGamma=true); # network is plotted & sent to file
R"dev.off()"; # wrap up and save image file
```



## treemix
treemix是利用等位基因频率来推断群体间分化和杂合（基因流动或基因渗入）。

用全基因组snp数据计算等位基因频率，并转换成treemix的输入文件格式。再用treemix进行基因流的分析。




reference
1. [PhyloNetworks paper](https://academic.oup.com/mbe/article/34/12/3292/4103410)
2. [PhyloNetworks manual](https://crsl4.github.io/PhyloNetworks.jl/latest/)
3. [TreeMix paper](https://www.nature.com/articles/npre.2012.6956.1)
4. [TreeMix software](https://bitbucket.org/nygcresearch/treemix/downloads/)