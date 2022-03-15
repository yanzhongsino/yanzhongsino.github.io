---
title: 基因流
date: 2021-11-12 16:00:00
categories: 
- bio
- bioinfo
tags: 
- gene set enrichment analysis
- GSEA
- topGO
description: 介绍生物信息学数据库。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=108151&auto=1&height=32"></iframe></div>

# 检测杂交/基因流


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