---
title: r8s估算系统发育时间
date: 2021-10-29 16:50:00
categories:
- bio
- biosoft
tags:
- tutorial
- biosoft
- divergence time
- r8s
- phylogeny
description: 记录系统发育时间估算软件r8s的安装和使用
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=17059176&auto=1&height=32"></iframe></div>

# 1. r8s
r8s 是美国加利福尼亚大学戴维斯分校的进化生物学家 Mike Sanderson 在2003年编写的用于估算进化树分化时间的软件，在进化生物学、分子生物地理学等学科有着广泛的应用，是估算分化时间常用的软件之一。该软件中的一些方法如 NPRS 和 PL 是软件作者最先提出的，目前在同类的其他软件中还难以实现。

# 2. r8s的安装
wget http://loco.biosci.arizona.edu/r8s/r8s.dist.tgz
tar zxf r8s.dist.tgz

或者sourceforge下载https://sourceforge.net/projects/r8s/

# 3. r8s的输入文件
## 3.1. 输入文件r8s.input
一个例子，包括两个模块，一个树模块，一个r8s参数设置模块。
树模块可以用raxml-ng等建树软件获得的树，包括枝长信息。
```
#nexus
begin trees;
tree tree_1 = (A:0.5069367934,(((((B:0.5881634293,C:0.2730677988)100:0.0325689044,D:0.2516482074)100:0.0150403964,E:0.2672133160)86:0.0239229988,((G:0.4074076834,H:0.4308871688)100:0.0260782585,I:0.2360982873)100:0.0237486276)100:0.0147845808,(J:0.5746285099,K:0.2089809581)92:0.0360035242)100:0.1832776220,L:0.2440847991);
end;

begin r8s;
blformat lengths=persite nsites=300000 ulrametric=no;
mrca pointa A B;
mrca pointb K L;
fixage taxon=pointa age=520;
constrain taxon=pointb min_age=350 max_age=410;
divtime method=PL algorithm=TN;
set smoothing=100;
showage;
describe plot=chronogram;
describe plot=chrono_description;
end;
```

## 3.2. r8s参数设置模块

1. blformat命令：进化树基本信息，需要在计算divergence time/rates analysis之前运行。
- lengths选项：total|persite
    描述进化树长度的单位。MP简约法获得的进化树，枝长单位是碱基替换个数，lengths选项应该用total；ML极大似然法获得的进化树，枝长单位是期望的碱基替换数，lengths选项应该用persite。
- nsites选项：number
    在进化分析中用到的碱基位点数量。
- ultrametric选项：yes|no
    表示该进化树是否经过了校正。如果选yes，直接给出进化时间，不对内部进行任何更改。如果要校对，则应该用calibrate命令。如果不想对ultrametric的进化树的分化时间全部重新估算，应该选yes。
- round选项：yes|no
    是否对枝长信息四舍五入。只针对nonultrametric树（因为ultrametric树的枝长经过四舍五入会转变成not truly ultrametric树。）。round选项将每个分枝上的替换数转换为整数。如果用户自己提供了已经校对时间的进化树，chronograms，需要设定round=no。

2. mrca命令：为节点定名。
mrca(most recent common ancestor)可以命名两个或多个类群的最近共同祖先。
例如想要给树 tree test=(a,(b,c));中a、b的共同祖先定名为node1，可以通过命令 mrca node1 a b来实现。

3. fixage命令：设定节点的分化时间。
在mrca命令的基础上，设定已知的节点分化时间点，r8s需要至少一个内部节点进行fixage。
用法fixage taxon=node1 age=100;

4. constrain命令：设定节点的分化时间。
在mrca命令的基础上，设定已知的节点分化时间上下限。
用法constrain taxon=node1 min_age=50 max_age=200;

5. divtime命令：分化时间估算
依据设定好的分化时间，对未知节点的分化时间进行估算。
用法 divtime method=LF|NPRS|PL algorithm=POWELL|TN|QNEWT
- method选项（计算方法）：LF|NPRS|PL
    分化时间估算的方式。推荐PL罚分似然法：如果各枝长变化速率差异很大，则smoothing值应该取的很小；如果各枝长变化速率差异很小，则smoothing值应该取的很大。
- algorithm选项（数学算法）：POWELL|TN|QNEWT
    算法，作者推荐TN算法。
- crossv选项：基于化石的进化树校正

divtime method=PL crossv=yes cvstart=0 cvinc=1 cvnum=18;
上述命令，是设置 smoothing 的值从 1, 10, 100, 1000 ... 1e17, 来计算，最后得到最佳的 smoothing 值。

若使用 fixage 对 2 个节点的分歧时间进行了固定，则可以运行命令：
divtime method=PL crossv=yes fossilfixed=yes cvstart=0 cvinc=1 cvnum=18;

若使用 fixage 对 1 个节点进行分歧时间固定，同时使用 constrain 对 2 个节点进行了约束，则可以运行命令：
divtime method=PL crossv=yes fossilconstrained=yes cvstart=0 cvinc=1 cvnum=18;

得到最优的smoothing值后，再用set设置，然后进行divtime分歧时间和替换速率的计算。

6. smoothing
divtime使用PL方法，需要设置参数smoothing的值。通过设置多个smoothing的值来进行一些计算，选择最优的值之后用set设置。

7. showage命令：显示分化时间和分化速率

8. describe命令：显示进化树及树的说明
- cladogram
    得到分支树的图，图上有各个节点的编号，和showage的结果结合观察。
- phylogram
    得到进化树的图，枝长表示替换数。
- chronogram
    得到超度量树的图，枝长表示时间。
- ratogram
    得到树图，枝长表示替换速率。


- phylo_description
    得到树的ASCII文字结果，枝长表示替换数。
- chrono_description
    得到树的ASCII文字结果，枝长表示时间。
- rato_description
    得到树的ASCII文字结果，枝长表示替换速率。

- node_info
    得到节点的信息表格

做超度量时间树，一般选择
describe plot=chronogram;
describe plot=chrono_description;

# 4. r8s运行
`r8s -b -f r8s.in > r8s.out`

-b是batch process the datafile，-f用于指定输入文件


结果保存在r8s.out文件中


references：
https://academic.oup.com/bioinformatics/article/19/2/301/372781
https://naturalis.github.io/mebioda/doc/week1/w1d5/r8s1.7.manual.pdf
http://image.sciencenet.cn/olddata/kexue.com.cn/upload/blog/file/2010/3/201032420201531842.0.pdf
http://www.chenlianfu.com/?p=2280

