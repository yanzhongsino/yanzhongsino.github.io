---
title: 系统发育网络推断 —— PhyloNetworks
date: 2021-03-27
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

# PhyloNetworks
1. PhyloNetworks简介
   - PhyloNetworks是通过基因树或多位点序列(SNaQ)的最大伪似然进行推断系统发育网络的一个Julia包。
2. PhyloNetworks原理
   - 原理：
3. PhyloNetworks输入输出
   - 输入：newick格式基因树(多个基因树组成的文件)
   - 输出：系统发育网络，基因流方向和杂交节点贡献比例
4. PhyloNetworks优势和不足
   - 推断系统发育网络，包括基因流的方向和强度。
   - 相较于其他推断系统发育网络的软件，PhyloNetworks集成了上游分析，网络估计，引导分析，下游特征进化分析，绘图等功能。
   - 不足是运行多样本(超过十个个体)和数据量大(超过1000个)会非常耗时(常常以星期/月计时)。
5. PhyloNetworks适用范围
   - PhyloNetworks适用于基因树数据
   - 适用于居群间或物种间的基因流推测
   - 适用于推断基因流方向和强度

# PhyloNetworks安装



# PhyloNetworks分析
## 输入文件


## 运行
时效
用hmax=1，12个线程，共计4.5days跑完。


### 计算CF

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


### 构建系统发育网络

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

# references
1. [PhyloNetworks paper](https://academic.oup.com/mbe/article/34/12/3292/4103410)
2. [PhyloNetworks github](https://github.com/crsl4/PhyloNetworks.jl)
3. [PhyloNetworks manual](https://crsl4.github.io/PhyloNetworks.jl/latest/)
4. [PhyloNetworks tutorials](https://crsl4.github.io/PhyloNetworks.jl/dev/)