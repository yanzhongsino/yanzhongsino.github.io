---
title: 系统发育网络推断 —— PhyloNetworks
date: 2021-04-12
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
PhyloNetworks是Julia包，Julia是一个类似R的计算机语言，可以交互式或脚本式执行命令。
## Julia 安装
如果想简要了解julia，可以看[julia tutorials](https://learnxinyminutes.com/docs/julia/)。
5. 预编译软件安装
在[julia software](https://julialang.org/downloads/)找到对应版本的预编译的软件下载解压缩即可使用。
```shell
wget https://julialang-s3.julialang.org/bin/linux/x64/1.7/julia-1.7.2-linux-x86_64.tar.gz #下载linux的64位预编译的julia
tar -xzf julia-1.7.2-linux-x86_64.tar.gz #解压缩
julia-1.7.2/bin/julia -h #查看帮助文章
```

2. 源代码编译安装
通过[julia github](https://github.com/JuliaLang/julia)获取源代码
```shell
git clone https://github.com/JuliaLang/julia.git #克隆github上的最新版源代码
cd julia
git checkout v1.7.2 #运行checkout来获取julia的最新稳定版本1.7.2
make #编译
./julia -h #查看帮助文档
```

之后把julia命令的路径添加到环境变量，便可以直接使用julia了。
julia与R很像，进入交互界面，只需要在shell里键入julia回车即可。

## 安装PhyloNetworks包
- 进入julia运行`using Pkg`-`Pkg.add("PhyloNetworks")`即可安装PhyloNetworks包。
- 在julia运行`Pkg.update()`会更新所有julia包。
- PhyloNetworks的配套包PhyloPlots是具有可视化网络和交互操作性的实用程序，例如将网络导出到 R（然后可以通过 R 绘制），建议安装：`using Pkg`-`Pkg.add("PhyloPlots")`

# PhyloNetworks分析
## 输入文件
输入文件包括一致性因子(CF)表和起始树。
1. 一致性因子(CF)表：可以从下面两种方式获取
- 每个基因估计的基因树，可以用MrBayes或RAxML建单独的基因树后合并所有基因树文件为一个基因树列表文件(比如这里用的是RAxML-NG建树后合并所有sample.raxml.support文件为一个文件raxmltrees.tre)。再从基因树列表计算CF值得到CF表(这一步参考下面步骤)。
- 每个4分类群子集的一致性因子(CF)表，即基因树频率表，可以用BUCKy获得。CF表的格式是csv(逗号分隔的表格)，表头是t1,t2,t3,t4,CF12_34,CF13_24,CF14_23,ngenes。
2. 起始树：需要一棵树作为优化起点。可以用基因树分析得到的物种树(比如这里用的是ASTRAL分析raxmltrees.tre的结果astral.tre)。

## 分析
### 用基因树计算CF值
1. 基因树raxmltrees.tre
- 通过MrBayes或RAxML建树软件对每个基因单独建树后合并成基因树文件。
- 比如我用的是RAxML-NG建树后合并所有sample.raxml.support文件为一个文件raxmltrees.tre的结果。

2. 用基因树计算CF获取tableCF.csv文件

```julia
using PhyloNetworks #导入PhyloNetworks包
raxmltrees=joinpath("raxmltrees.tre") #读取基因树文件
genetrees = readMultiTopology(raxmltrees) #解析基因树

using PhyloPlots #导入PhylpPlots包
plot(genetrees[3], :R) #画基因树文件里第三棵树

q,t = countquartetsintrees(genetrees) #读取基因树，计算四分类群的CFs
df = writeTableCF(q,t) #读取计算得到的CF值到df：基因频率

using CSV #导入CSV包
CSV.write("tableCF.csv", df) #保存df内容为tableCF.csv文件
raxmlCF = readTableCF("tableCF.csv") #读取tableCF.csv文件到raxmlCF
```

### 构建系统发育网络【耗时】
**时效参考**：29个物种的2700棵基因树，用hmax=1，10个线程，约10天才跑完。

获取了tableCF.csv文件后，可以继续在julia中用PhyloNetworks包构建系统发育网络。

```julia
using PhyloNetworks #导入PhyloNetworks包
astralfile= joinpath("astral.tre") #导入起始树文件astral.tre，这个树是个体树
astraltree = readMultiTopology(astralfile) # 读取树文件
raxmlCF = readTableCF("tableCF.csv") # 读取tableCF.csv文件，生成"DataCF" 对象
net0 = snaq!(astraltree,raxmlCF, hmax=0, filename="net0", seed=1234) #
```

### 确认最佳杂交节点
在构建h=1-10的系统发育网络之后，要确定最佳h值

### 绘制系统发育网络
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
3. [PhyloNetworks tutorials](https://crsl4.github.io/PhyloNetworks.jl/dev/)