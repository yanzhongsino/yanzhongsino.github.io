---
title: 系统发育网络推断 —— PhyloNetworks
date: 2022-04-14
categories: 
- bioinfo
- gene flow
tags: 
- PhyloNetworks
- gene flow
- hybridization
- introgressive
- population networks
- Julia
description: 这篇博客描述了用Julia包PhyloNetworks来推断系统发育网络，并选择最佳杂交次数和网络可视化的操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=108998&auto=1&height=32"></iframe></div>

# 1. PhyloNetworks
1. PhyloNetworks简介
   - PhyloNetworks是通过基因树或多位点序列(SNaQ)的最大伪似然进行推断系统发育网络的一个Julia包。
2. PhyloNetworks原理
   - 原理：通过SNaQ来实现网络推断，SNaQ通过估计4分类群子集的最大伪似然来加速运算，估计的网络不受根的影响。
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

# 2. PhyloNetworks安装
PhyloNetworks是Julia包，Julia是一个类似R的计算机语言，可以交互式或脚本式执行命令。
## 2.1. Julia 安装
如果想简要了解julia，可以看[julia tutorials](https://learnxinyminutes.com/docs/julia/)。

1. 预编译软件安装
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
julia与R很像，进入交互界面，只需要在shell里键入`julia`回车即可，退出交互界面是ctrl+D。

## 2.2. 安装PhyloNetworks包
- 进入julia运行`using Pkg`-`Pkg.add("PhyloNetworks")`即可安装PhyloNetworks包。
- 在julia运行`Pkg.update()`会更新所有julia包。
- PhyloNetworks的配套包PhyloPlots是具有可视化网络和交互操作性的实用程序，例如将网络导出到 R（然后可以通过 R 绘制），建议安装：`using Pkg`-`Pkg.add("PhyloPlots")`

# 3. PhyloNetworks分析
## 3.1. 输入文件
1. CF表
- 输入文件可以有多种形式，不管哪种形式，最终PhyloNetworks的SNaQ用到的都是CF表。
- 每个4分类群子集的一致性因子(CF)表，即基因树频率表。CF表的格式是csv(逗号分隔的表格)，表头是t1,t2,t3,t4,CF12_34,CF13_24,CF14_23,ngenes。
- 可以用建树分析得到的基因树来计算CF；也可以用经过对齐的基因序列，通过TICR管道通过BUCKy估计CF获取CF表（用BUCKy需要考虑基因树估计误差）。

2. 起始树
- 除了CF表外，SNaQ还需要一棵起始树作为优化起点。
- 可以用基因树分析得到的物种树(比如这里用的是ASTRAL分析raxmltrees.tre的结果astraltree.tre)。

下面介绍两种获取CF表的方式：
### 3.1.1. 从基因树到CF表
1. 文件准备：基因树raxmltrees.tre
- 通过MrBayes或RAxML建树软件对每个基因单独建树后合并成基因树列表文件。
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

### 3.1.2. TICR：从对齐序列(sequence alignment)到CF表
waiting tag...

参考网址https://crsl4.github.io/PhyloNetworks.jl/latest/man/ticr_howtogetQuartetCFs/

## 3.2. 构建系统发育网络【耗时】
**时效参考**：29个物种的2700棵基因树，用hmax=1，10个线程，约10天才跑完。

获取了CF表tableCF.csv文件后，可以继续在julia中用PhyloNetworks包构建系统发育网络。

### 3.2.1. 构建网络
1. 构建网络起点
```julia
using PhyloNetworks #导入PhyloNetworks包
astralfile= joinpath("astraltree.tre") #导入起始树文件astraltree.tre，这个树是个体树
astraltree = readMultiTopology(astralfile) # 读取树文件
raxmlCF = readTableCF("tableCF.csv") # 读取tableCF.csv文件，生成"DataCF" 对象
net0 = snaq!(astraltree,raxmlCF, hmax=0, filename="net0", seed=1234) # 用astraltree,raxmlCF，指定hmax=0来运行。
```

部分屏幕输出：
```
MaxNet is (C,D,((B,A):1.395762055180493,(O,E):0.48453400554506426):10.0);
with -loglik 53.53150526187732
```

2. 把得到的net0作为起点来构建hmax=1的网络
```
julia命令：
julia>net1 = snaq!(net0, raxmlCF, hmax=1, filename="net1", seed=2345)

屏幕输出：
best network and networks with different hybrid/gene flow directions printed to .networks file
MaxNet is (C,D,((O,(E,#H7:::0.19558838614943078):0.31352437658618976):0.6640664399202987,(B,(A)#H7:::0.8044116138505693):10.0):10.0);
with -loglik 28.31506721890958
```

3. 输出文件
snaq输出文件包括：
- net1.out：主输出文件，代表上面的net1；包含括号格式的网络树和参数，每一次run的输出保存在这个文件
- net1.networks：网络树列表，包含了每一次run的网络树结果。这些网络树是对评估的最佳网络net1的修改（通过移动杂交节点来改变网络方向），对每个修改后的网络，计算了伪似然分数（pseudolikelihood score，loglik或-Ploglik）
- net1.err： 如果有任何错误信息会生成这个文件would provide info about errors, if any

可以在julia中查看文件，用`less("net1.out")`命令，下箭头向下滚动，`q`退出查看。

4. 把最佳网络写入文件
```
julia> net1 = readTopology("net1.out") # 读取net1.out文件到net1；如果接着上面运行net1已经赋值了，则不必读取
julia> net1 # 输出net1的信息到屏幕
HybridNetwork, Rooted Network
12 edges
12 nodes: 6 tips, 1 hybrid nodes, 5 internal tree nodes.
tip labels: C, D, O, E, ...
(C,D,((O,(E,#H7:::0.196):0.314):0.664,((A)#H7:::0.804,B):10.0):10.0);

julia> writeTopology(net1)  # 输出net1的到屏幕, 包含完整精度的枝长和 γ
"(C,D,((O,(E,#H7:::0.19558838614943078):0.31352437658618976):0.6640664399202987,((A)#H7:::0.8044116138505693,B):10.0):10.0);"

julia> writeTopology(net1, round=true, digits=2) # 精度digits为两位
"(C,D,((O,(E,#H7:::0.2):0.31):0.66,((A)#H7:::0.8,B):10.0):10.0);"

julia> writeTopology(net1,di=true) # 适用于dendroscope使用的括号格式，忽略γ值
"(C,D,((O,(E,#H7):0.31352437658618976):0.6640664399202987,((A)#H7,B):10.0):10.0);"

julia> writeTopology(net1, "bestnet_h1.tre") # 把net1的最佳网络写到文件bestnet_h1.tre中
```

这里写入文件bestnet_h1.tre的内容和net1.networks的第一行(删除网络树分号后内容)是一样的，也可以直接在shell里用`head -1 net1.networks|sed "s/;.*/;/g" >bestnet_h1.tre`来获取。

5. 迭代运行
在得到net1后，可以基于net1来运行计算有两个杂交节点(hybrid nodes)的net2，继续迭代可以得到net3等。
```julia
net2 = snaq!(net1,raxmlCF, hmax=2, filename="net2", seed=3456)
net3 = snaq!(net2,raxmlCF, hmax=3, filename="net3", seed=4567)
```

6. 可视化网络
在julia里可以用PhyloPlots绘制得到的网络图。
```julia
using PhyloPlots
plot(net1, :R);
```

<img src="https://crsl4.github.io/PhyloNetworks.jl/latest/assets/figures/snaqplot_net1_1.svg" title="系统发育网络图net1示例" width="60%"/>

**<p align="center">Figure 1. 系统发育网络图net1示例**
from [PhyloNetworks Tutorial](https://crsl4.github.io/PhyloNetworks.jl/latest/man/snaq_plot/)</p>

这个网络中A作为杂交后代，80.4%是B的姐妹，19.6%是E的姐妹。

### 3.2.2. 并行计算
- 最好根据使用的服务器的处理器/核心(processors/cores)的数量来决定用多少线程，超过服务器有的核心数量不会让运行更快。
- 并行运算的线程设置其实等于计算次数，例如设定12线程，会计算12次。

#### 3.2.2.1. 交互式并行计算
交互式里并行计算可以用两种方法设置并行核心数量
1. 用`julia -p 12`进入julia代表使用12个核心。
2. 用`julia`进入julia（默认使用1个核心），然后用`using Distributed`-`addprocs(11)`向julia添加11个核心数量（现在一共12个核心）。

notes
- 在`julia`中可以用`using Distributed`-`nworkers()`查看现在正在使用的核心总数。
- 在添加核心后导入包，比如添加后运行`using PhyloNetworks`才能让这个包的函数使用到多线程。

#### 3.2.2.2. 脚本式并行计算【推荐】
使用脚本来并行运行julia

1. julia脚本runSNaQ.jl的内容
```
#!/usr/bin/env julia

# file "runSNaQ.jl". run in the shell like this in general:
# julia runSNaQ.jl hvalue nruns
# example for h=2 and default 10 runs:
# julia runSNaQ.jl 2
# or example for h=3 and 50 runs:
# julia runSNaQ.jl 3 50

length(ARGS) > 0 ||
    error("need 1 or 2 arguments: # reticulations (h) and # runs (optional, 10 by default)")
h = parse(Int, ARGS[1])
nruns = 10
if length(ARGS) > 1
    nruns = parse(Int, ARGS[2])
end
outputfile = string("net", h, "_", nruns, "runs") # example: "net2_10runs"
seed = 1234 + h # change as desired! Best to have it different for different h
@info "will run SNaQ with h=$h, # of runs=$nruns, seed=$seed, output will go to: $outputfile"

using Distributed
addprocs(nruns)
@everywhere using PhyloNetworks
net0 = readTopology("astraltree.tre");
using DataFrames, CSV
df_sp = DataFrame(CSV.File("tableCF.csv", pool=false); copycols=false);
d_sp = readTableCF!(df_sp);
net = snaq!(net0, d_sp, hmax=h, filename=outputfile, seed=seed, runs=nruns)
```

2. 使用julia脚本
举例：`julia runSNaQ.jl 1 12`

julia运行时间比较长，建议用nohup和&来放后台：`nohup julia runSNaQ.jl 1 12 >h1.log 2>&1 &`

3. 运行说明
- 脚本接受的运行参数有两个，第一个参数设置最大杂交次数hmax，第二个参数设置并行运行的核心数（也代表重复运行次数）。
- 这里默认输入文件在当前目录下，包括CF表tableCF.csv文件和起始树astraltree.tre文件。

4. 迭代运行
- 如果时间充裕，可选迭代运行的方式。
- 跑完hmax=1参数后，可以用hmax=1的树结果作为运行hmax=2的起点树。跑完hmax=2参数后，可以用hmax=2的树结果作为运行hmax=3的起点树。直到跑到hmax=5或者hmax=10的，能找到最佳hmax值。这就是迭代运行的方式。
- 用`head -1 net1.networks|sed "s/;.*/;/g" >bestnet_h1.tre`获取hmax=1的最佳网络树。
- 然后更改脚本里指定的起始树为`bestnet_h1.tre`，开始搜索h=2的网络树。
- 具体更改julia脚本中的两行为下面的参数。

```
net1 = readTopology("bestnet_h1.tre");
net2 = snaq!(net1, d_sp, hmax=h, filename=outputfile, seed=seed, runs=nruns)
```

- 类似的操作，在获得hmax=2的结果后继续获得hmax=3的结果。依次获得hmax更高的结果。

5. 不迭代并行运行
- 如果时间不充裕，可选不迭代运行的方式。
- 即hmax=1，2，3，4，5...的程序同时运行，起始树全部都选择前面获得的astral树。
- 操作上，可以准备多份runSNaQ.jl文件，分别命名为runSNaQ_h1.jl,runSNaQ_h2.jl,runSNaQ_h3.jl...同时运行`nohup julia runSNaQ_h1.jl 1 12 >h1.log 2>&1 &`,`nohup julia runSNaQ_h2.jl 2 12 >h2.log 2>&1 &`,`nohup julia runSNaQ_h3.jl 3 12 >h3.log 2>&1 &`...
- julia脚本内需要修改变量名称，以免同时运行时相互干扰。

```
net0_h1 = readTopology("astraltree.tre");
net1_h1 = snaq!(net1_h1, d_sp, hmax=h, filename=outputfile, seed=seed, runs=nruns)
```

```
net0_h2 = readTopology("astraltree.tre");
net1_h2 = snaq!(net0_h2, d_sp, hmax=h, filename=outputfile, seed=seed, runs=nruns)
```

#### 提交slurm集群
waiting tag...

PhyloNetworks教程里的建议是在上一步运行julia脚本之后用slurm提交julia作业到集群。但我没具体去研究，提交到集群后似乎可以自动化运行h=0到h=3。

设置slurm的脚本复制到下面：

```
#!/bin/bash
#SBATCH -o path/to/slurm/log/file/runsnaq_slurm%a.log
#SBATCH -J runsnaq
#SBATCH --array=0-3
#SBATCH -c 30
## --array: to run multiple instances of this script,
##          one for each value in the array.
##          1 instance = 1 task
## -J job name
## -c number of cores (CPUs) per task

echo "slurm task ID = $SLURM_ARRAY_TASK_ID used as hmax"
echo "start of SNaQ parallel runs on $(hostname)"
# finally: launch the julia script, using Julia executable appropriate for slurm, with full paths:
/workspace/software/bin/julia --history-file=no -- runSNaQ.jl $SLURM_ARRAY_TASK_ID 30 > net$SLURM_ARRAY_TASK_ID_30runs.screenlog 2>&1
echo "end of SNaQ run ..."
```

后面有时间再仔细研究研究：
[PhyloNetworks SNaQ部分](https://crsl4.github.io/PhyloNetworks.jl/latest/man/snaq_plot/)

## 3.3. 选择最佳杂交次数
### 3.3.1. 运行不同杂交次数
一般来说需要运行多次网络构建，最后根据结果来确定最佳的杂交次数(也就是h值)。
- 根据估计的杂交次数来运行，比如估计有4次杂交，可以计划运行从h=0到h=6。每个指定的h最好运行在8次以上。
- 也可以根据运行结果的loglik值来决定运行的最大h值。

有两种运行策略，迭代和非迭代：
1. 迭代
- 迭代运行策略指先运行h=0，待结束后用h=0的结果作为起始树运行h=1；待h=1运行结束再用h=1的结果作为起始树运行h=2，直到h=6。

2. 非迭代
- 非迭代运行策略就是把astraltree.tre作为所有h取值的起始树，这样可以同时运行h=0到h=6，加快获得结果的时间。

### 3.3.2. 选择最佳h值
根据loglik值随h值的变化，当loglik降到了最低且在h继续增大时保持平稳时的h值即为最佳h值。
1. loglik值
- 每个估计的网络都有一个伪偏差loglik值：负对数似然的倍数直到一个常数。
- 这个loglik值越低越好（如果网络完美地拟合数据，这个常数就为0）。
- 每个loglik值可以在结果文件net1.network中查看。

2. 绘制loglik值随h值的变化
得到不同h值下的最佳网络的loglik值后，就可以绘制loglik值随h值的变化。

例如官方教程里给出的loglik值从h=0-3的变化。可以用教程里建议的julia的R"plot"绘图（如下）或者julia绘图，也可以在R或者excel等软件中自行画折线图。

```julia
scores = [net0.loglik, net1.loglik, net2.loglik, net3.loglik]
hmax = collect(0:3)
R"plot"(hmax, scores, type="b", ylab="network score", xlab="hmax", col="blue");
```

<img src="https://crsl4.github.io/PhyloNetworks.jl/latest/assets/figures/snaqplot_scores_heuristic.svg" title="loglik随h变化的示例图" width="60%"/>

**<p align="center">Figure 2. loglik随h变化的示例图**
from [PhyloNetworks Tutorial](https://crsl4.github.io/PhyloNetworks.jl/latest/man/snaq_plot/)</p>

从图中可以看到loglik值在h=1时降到了最低且在h继续增大时保持平稳。所以建议最佳杂交次数是h=1。

3. 用R包ggplot2绘制

```R
library(ggplot2)
hmax<-c(0:10)
score<-c(246.5604838333106,10.072487746425647,10.072487747047507,10.07248774671892,10.072487747122702,66.88016392780662,10.072487746482409,10.072487765482101,66.88016941187634,10.072487747318323,10.072487746650621)
df <- data.frame(h = hmax, loglike = score)
ggplot(data = df, aes(x = ind, y = loglike)) + geom_line()+ geom_point(shape=1,size=3,color= "blue")+ theme_classic()+xlab("the number of hybrid nodes")+ylab("pseudolikelihood scores")+scale_x_continuous(breaks=seq(0, 10, 1))+ theme_classic()+theme(axis.title.x = element_text(size = 15, face = "bold"),axis.title.y = element_text(size = 15, face = "bold"),axis.text.x = element_text(size = 13, face = "bold"),axis.text.y = element_text(size = 13, face = "bold"))
```

## 3.4. 系统发育网络可视化
找到最佳的杂交次数后，可以把对应的系统发育网络绘制出来，比如这里的h=1。

### 3.4.1. 对网络重新定根
由于SNaQ算法不考虑root，大部分情况下bestnet_h1.tre结果都需要重新定外类群为树的根。

```julia
using PhyloNetworks
net = readTopology("net1_12runs.out") # 读取最佳杂交次数的结果
writeTopology(net,"bestnet_h1.tre") # 保存最佳杂交次数的最佳杂交树
net1 = readTopology("bestnet_h1.tre") #读取bestnet_h1.tre到net1
rootatnode!(net1,"C") #重新定根到样本C，可直接用于之后的PhyloPlots画图
net1_reroot = rootatnode!(net1,"C") # 重新定根到样本C，并保存到net1root
writeTopology(net1_reroot) # 输入net1root的内容到屏幕，引号内的内容就是要保存的重新定根后的网络
writeTopology(net1_reroot, "net1_reroot.tre") # 把重新定根的网络写到文件net1root.tre中
```

### 3.4.2. 使用配套包PhyloPlots绘制系统发育网络
可以使用配套julia包PhyloPlots绘制系统发育网络
1. 保存图为pdf
```julia
using PhyloNetworks
net1_reroot = readTopology("net1_reroot.tre") # 读取重新定根后的net1root.tre到net1root
using PhyloPlots # 导入PhyloPlots包，用于绘图
using RCall      # 导入RCall，用于使用R的命令
imagefilename = "net1_reroot.pdf" # 创建pdf格式文件
R"pdf"(imagefilename) # 开始绘图
plot(net1_reroot, :R, showGamma=true); # 绘制网络并把网络图发到文件
R"dev.off()"; # 保存和退出
```

2. 保存成svg格式
```julia
using PhyloNetworks
net1root = readTopology("net1_reroot.tre") # 读取重新定根后的net1root.tre到net1root
using PhyloPlots
using RCall
imagefilename = "net1_reroot.svg"
R"svg"(imagefilename, width=4, height=3) # 开始绘图，设定宽和高
R"par"(mar=[0,0,0,0]) # 去除图的边界
plot(net1root, :R, showGamma=true, showEdgeNumber=true); # 绘制网络并把网络图发到文件
R"dev.off()";
```

<img src="https://crsl4.github.io/PhyloNetworks.jl/latest/assets/figures/snaqplot_net1_2.svg" title="系统发育网络示例图" width="60%"/>

**<p align="center">Figure 3. 系统发育网络示例图**
from [PhyloNetworks Tutorial](https://crsl4.github.io/PhyloNetworks.jl/latest/man/snaq_plot/)</p>

这里的图和前面的Figure 1一样。

### 3.4.3. 使用ggtree绘制系统发育网络
waiting tag...

## 3.5. 更多分析
更多分析可参考[PhyloNetworks tutorials](https://crsl4.github.io/PhyloNetworks.jl/dev/)

# 4. references
1. PhyloNetworks paper：https://academic.oup.com/mbe/article/34/12/3292/4103410
2. PhyloNetworks github]：https://github.com/crsl4/PhyloNetworks.jl
3. PhyloNetworks tutorials：https://crsl4.github.io/PhyloNetworks.jl/dev/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>