---
title: 用PhyloNetworks推断系统发育网络
date: 2021-03-27 16:50:00
categories: 
- bio
- bioinfo
tags: 
- PhyloNetworks
- gene flow
- hybridization
- introgressive
- population networks
description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=20707476&auto=1&height=32"></iframe><music URL></div>


时效
用hmax=1，12个线程，共计4.5days跑完。


分析

获取tableCF.csv文件
```julia
using PhyloNetworks
raxmltrees=joinpath("raxmltrees.tre");
genetrees = readMultiTopology(raxmltrees);

using PhyloPlots
plot(genetrees[3], :R); 

q,t = countquartetsintrees(genetrees);
df = writeTableCF(q,t)
using CSV
CSV.write("tableCF.csv", df);
raxmlCF = readTableCF("tableCF.csv")
```


构建系统发育网络

```julia
using PhyloNetworks
astralfile= joinpath("astral.tre") #导入树文件，这个树就是个体树
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

references
1. [PhyloNetworks paper](https://academic.oup.com/mbe/article/34/12/3292/4103410)
2. [PhyloNetworks github](https://github.com/crsl4/PhyloNetworks.jl)
3. [PhyloNetworks manual](https://crsl4.github.io/PhyloNetworks.jl/latest/)
4. [PhyloNetworks tutorials](https://crsl4.github.io/PhyloNetworks.jl/dev/)