---
title: 用PAML的codeML进行选择分析：（一）概况
date: 2023-10-10
categories: 
- bio
- evolution
- selection
tags:
- bioinfo
- biosoft
- PAML
- codeML
- d<sub>N</sub>
- d<sub>S</sub>
- d<sub>N</sub>/d<sub>S</sub>
- d<sub>N</sub>d<sub>S</sub>
- K<sub>A</sub>
- K<sub>S</sub>
- K<sub>A</sub>K<sub>S</sub>
- K<sub>A</sub>/K<sub>S</sub>
- omega(ω)
- omega
- ω
- 位点模型 (site model)
- 分支位点模型 (branch-site model)
- 分支模型 (branch model)
- 进化枝模型 (clade model)
- 似然比检验(LRT)
- 卡方检验(chi2 test)
description: 介绍了使用PAML的codeML计算 ω 值 (d<sub>N</sub>/d<sub>S</sub>) 进行选择分析的概况，包括选择分析的目的、检测方法和实际应用，PAML的介绍、codeML检测选择的四种模型。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=65777&auto=1&height=32"></iframe></div>

# 1. 选择分析
## 1.1. omega(ω)检测选择
d<sub>N</sub>d<sub>S</sub>(K<sub>A</sub>K<sub>S</sub>)，以及 omega(ω) 的定义，以及 ω 用于检测选择压可以参考博客**一文说清K<sub>A</sub>K<sub>S</sub>、d<sub>N</sub>d<sub>S</sub>、D<sub>n</sub>D<sub>s</sub>，以及ω**：https://yanzhongsino.github.io/2023/10/09/evolution_selection_dNdS_intro

1. omega(ω) 的含义
- omega(ω) = d<sub>N</sub>/d<sub>S</sub>(K<sub>A</sub>/K<sub>S</sub>) 
2. omega(ω) 可用于推断作用于蛋白质编码基因的自然选择的方向和程度
- 当 ω > 1 时，意味着正选择 (positive selection)；
- 当 0 < ω < 1 意味着负选择 (negative selection，也叫净化选择或纯化选择 purifying selection)；
- 当 ω = 1 则表示中性进化(neutral selection)，即不受选择。
3. 背景知识
- 通常，由于未改变编码氨基酸，大多数同义突变都是中性的，即没有受到正选择或者负选择；由于改变了编码氨基酸，大多数非同义突变受到负选择，没有保留下来。
- 所以，在现存类群中，观察到的同义突变数量比非同义突变数量会高得多，通常d<sub>N</sub> < d<sub>S</sub>。计算现存类群间 ω 时，结果通常在0.05-0.40之间，只有很少数的基因(e.g. MHC)发现 ω > 1。
4. 可能的误差
- 在基因的不同位点或进化的不同时期，正选择和负选择的组合可能会相互抵消。
- 由此产生的平均值可能会掩盖其中一种选择的存在，并降低另一种选择的似然程度。
5. 统计分析
- 有必要进行统计分析，以确定 ω 的结果是否与 1 有显著差异，或是否由于数据集有限而出现任何明显差异。
- 近似法的适当统计检验包括用正态近似值对 dN - dS 进行近似，并确定 0 是否位于近似值的中心区域。
- 可以使用更复杂的似然法来分析最大似然分析的结果，通过进行卡方检验来区分空模型和观测结果。

## 1.2. 选择分析的目的
1. 绝大多数情况下 ω < 1，因此选择分析通常是为了检测到较少见的正选择(ω >1)
- 在一个基因的所有位点或系统发育上的所有谱系(分支)上的平均 ω 值很少大于1。因此，用平均 ω 值检测正选择是非常保守的（很可能检测不出来）。
- 检测系统发育树上的特定分支或基因的单个位点是否经历正选择更有价值。所以选择分析的重点通常是**找到系统发育树上受到正选择的分支**，以及**基因上受到正选择的位点**。
2. 另外，结合系统发育树，还可以分析 ω 在进化树上的变化，虽然进化树上的 ω 都小于1，但某一支比其他所有支的 ω 都要小，这也说明这一支受到**负选择的放松**。

## 1.3. 选择分析的情况
1. 基因在**两个物种的选择分析**，较简单。
- 直接比较这两个物种的密码子序列，计算dN/dS，即 ω 值，通过 ω 值判断选择的方向和程度。若 ω > 1，即表明该基因在物种进化过程中，即由其祖先物种分化成这两个物种时，基因受到了正选择。
2. 基因在多物种的选择分析，如果仍然按照两个物种的方法，结果可能不好解释。
- 因为该基因可能在某一类群中序列很相似，其两两比较时，ω <= 1；而在另外一类群中两两比较时，很多时候 ω  > 1。最后软件可以从总体上给一个ω值，但该值不可以拿来简单地评价该基因是否受到了正选择。
- 所以，对多个物种进行正选择分析时，没法直接评价该基因是否受到了正选择。正选择只有在进行两两序列比较的时候，才能计算 ω 值，从而得到结果。
3. 基因在**多物种的选择分析**，目的则是：比较某个分枝上祖先节点和后裔节点（可以理解成，对无根树上某分枝两侧的两组物种进行比较，依然属于两两比较），从而计算该**分枝的 ω 值**。
4. 在实际数据中，基因在不同的进化分枝上具有不同的 ω 值，在序列不同的位点也具有不同的 ω 值
- 可以同时分析目标分枝的 ω 值和序列位点的 ω 值，从而判断哪一支的哪个基因受到正选择。
- 目标分枝两侧的物种数量较多时，可以对序列上的每个位点进行 ω 值分析，从而鉴定出正选择位点。

## 1.4. 选择分析的软件
基于计算 ω 值 (d<sub>N</sub>/d<sub>S</sub>) 来进行选择分析的软件有很多，最经典的是PAML的codeML，本文接下来便介绍这个程序。

PAML(Phylogenetic Analysis by Maximum Likelihood)是Yang Zihen lab开发的使用最大似然法进行DNA和蛋白序列的系统发育分析的程序包，包括许多程序，其中codeML程序整合了许多选择分析的模型，用于系统发育树上的选择分析。

# 2. 使用codeML进行选择分析的流程
1. 配置零假设模型(null model)和备选模型(正选择模型)的参数(配置文件codeml.ctl)；
2. 运行codeML程序分别对两个模型进行分析，获得各自的似然值 (lnL)；
3. 通过统计检验(如卡方检验)比较两个似然值(lnL)，自由度为两个模型之间自由参数数量的差异，并计算p值，用来判断两个模型间是否存在显著差异；
4. 根据统计检验结果解读，是同意备选模型 (存在正选择) 还是拒绝备选模型。
5. 如果是位点模型或分支位点模型，还可以进一步根据BEB方法判断受到正选择的位点的位置。

# 3. codeML进行选择分析的四种模型
四种模型的主要区别在于假设的不同，是否允许位点间的 ω 值和分支间的 ω 值不同，是基于先验知识选择的模型。
## 3.1. 位点模型 (site model) 
### 3.1.1. 位点模型介绍
1. 位点模型的作用
- 主要用于检测基因中的正选择位点。
2. 位点模型的主要假设
- 主要假设数据集中不同密码子位点受的选择压是来自统计分布的随机变量，因此允许密码子位点之间的 ω 不同。
- 不考虑不同支系间受的选择压力差异，假设进化树中各分枝的 ω 值是一致的。
- 所以，使用位点模型能在整体水平上检测基因的正选择位点，而不能表明基因在某个进化分枝上是否受到正选择压。
- 正选择定义为存在某些密码子的 ω >1. 执行似然比检验(LRT)与null模型的一般密码子比较，不允许任何密码子ω>1。
3. 位点模型使用的模型
- M0（单一比率），即：One-ratio model，假设所有位点具有相同的  ω 值；`model = 0, NSites = 0`
- M1a（近中性），假设仅有保守位点（0<ω <1）和中性位点（ ω =1）而没有正选择位点 (ω >1)存在，这两类位点的比率分别为p0和p1，其对应的ω 值分别为ω0、ω1； `model = 0, NSites = 1`
- M2a（正选择），该模型在M1基础上增加了第三类ω值，即假设除了保守位点和中性位点外，还存在处于正选择压力下的位点 (ω >1)，这三类位点的比率分别为p0、p1和p2，其对应的ω 值分别为ω0、ω1和ω2； `model = 0, NSites = 2`
- M3（离散），假设所有的位点ω 值呈简单的离散分布趋势； `model = 0, NSites = 3`
- M7（beta），假设所有位点的 ω 属于矩阵（0, 1）并呈beta分布； `model = 0, NSites = 7`
- M8（beta & ω ） ，该模型在M7基础上增加另一类ω 值（ω >1）； `model = 0, NSites = 8`
- M8a（beta & ω =1），与M8模型类似，但将ω 值固定为1（ω =0）； `model = 0, NSites = 8`
4. 常选的成对模型（前两对被广泛使用）
- M1a(选择约束放松)vs. M2a(正选择)
- M7 (beta) (选择约束放松)vs. M8 (beta&ω) (正选择)
- M3（分散比率） vs. M0（负选择）
- M8* （正选择）vs. M8a（选择约束放松）

### 3.1.2. 位点模型的分析过程
1. 选择零假设模型(null model)和备选模型(正选择模型)两种模型
- 有两对位点模型特别有效，M1a(选择约束放松)vs. M2a(正选择)，以及 M7 (beta) (选择约束放松)vs. M8 (beta&ω) (正选择)：
- (1) 模型M1a是null model，认为所有位点的 ω 值 < 1 或 = 1 两类; 
- (2) 模型M2a是正选择模型，存在 ω <1、=1或> 1的位点。
2. 使用codeML分别分析两个模型，获得各自的似然值 (lnL)
3. 通过统计检验(如卡方检验)比较两个似然值(lnL)，并计算p值，用来判断两个模型间是否存在显著差异
- 似然率检验 (LRT)统计量、或两倍于两个比较模型之间的对数似然差(2Δ)，可用于卡方检验。
- 自由度为两个模型之间自由参数数量的差异。例如，M1a有2个自由参数，M2a有4个，因此自由度为2，M7-M8的比较也有2个自由度。
- 若p值 < 0.05，则否定null model，认为存在正选择位点。
4. 同时，推荐采用比较模型M7和M8，其p值结果比上一种比较方法更宽松，能检测到更多的正选择基因。
- M7 模型是一个零假设模型，所有位点在 0 ≤ ω ≤ 1 的区间内，与M1a不同的是把 ω 分为10类。不允许 ω > 1 的站点。
- 备选模型 M8 模型增加了 ω > 1 的第 11 类位点。对每个位点进行测试，以确定其所属类别。
5. 如果LRT倾向于M2a或M8，我们可能会问：基因中的哪些位点处于正选择之下?
- 利用贝叶斯定理计算每个 ω > 1的候选正选择位点的后验概率来回答这个问题;
- 在codeML中实现了两种方法：
- (1) Naïve Empirical Bayes (NEB)方法(Nielsen and Yang 1998)通过使用模型中参数的最大似然估计(MLEs)计算每个位点来自不同位点类别的后验概率，而不考虑其抽样误差或不确定性。
- (2) 贝叶斯经验贝叶斯(Bayes Empirical Bayes, BEB)方法是对NEB方法的改进，并适应了MLEs中的不确定性(Yang et al. 2005)。虽然codeML报告两种方法的结果，但应该使用BEB而不是NEB。

## 3.2. 分支位点模型 (branch-site model)
### 3.2.1. 分支位点模型介绍
1. 分支位点模型的作用
- 主要用于检测基因在某个进化枝上是否存在的正选择位点。
2. 分支位点模型的主要假设
- 主要假设不同氨基酸位点的和不同支系间受的选择压力均存在差异（既考虑位点间也考虑支系间的 ω 值存在差异）。
- 接受正选择测试的分支称为前景分支，而树上的所有其他分支都是背景分支。
- 背景分支有两类位点，保守位点0 < ω0 < 1 ，中性位点 ω1 = 1；前景分支，正选择存在的位点 ω2 > 1 。通过比较零假设模型(ω2 = 1)和备择假设模型(ω2 > 1)的差异显著性来判断是否存在正选择。
3. 分支位点模型使用的模型和主要参数
- Model A null:`model=2, NSites=2, ncatG=ignored, fix_omega = 1, omega = 1`，设定 ω 为固定值1。
- Model A :`model=2, NSites=2, ncatG=ignored, fix_omega = 0, omega = 1.5`， 估算 ω 是否大于1。
- Model B :`model=2, NSites=3, ncatG=ignored`
- Model C :`model=3, NSites=2, ncatG=ignored`
- Model D :`model=3, NSites=3, ncatG=2 or 3`
4. 常选的成对模型
- Model A (正选择) vs. Model A null (中性进化)

### 3.2.2. 分支位点模型的分析过程
1. 选择零假设模型(null model)和备选模型(正选择模型)两种模型
- 正选择模型为Model A，将 ω 值分成<1、=1、>1的三类，这和site model中的一样；
- 零假设模型和正选择类似，只是将 ω 固定成1，作为null model（Model A null）。
2. 使用codeML分别分析两个模型，获得各自的似然值 (lnL)
3. 比较两种模型的似然差异，利用卡方检验（自由度为2）算p值（chi2命令算出的值除以2）。
- 若p值< 0.05，则否定null model，认为存在正选择位点。
- 与位点模型一样，通过BEB方法识别前景支上可能处于正选择的密码子位点。
- 通过BEB方法计算正选择位点的后验概率，若存在概率值 > 0.95正选择位点，则表示基因在目标分枝上受到正选择。
4. notes
- 分支位点模型并不给出分枝上的 ω 值。这表明虽然考虑了目标分枝上具有不同的 ω 值，但仍然以分析位点上的 ω 为主。
- 值得注意的是，在分支位点模型下可能检测到正选择位点，但在目标分枝上的 ω 值仍然可能低于1。可能软件作者基于这点考虑，就没有给出目标分枝上的 ω 值，以免影响一些人对正选择结果的判断。
- 要注意应该预先指定前景支。如果在没有先验生物学假设的情况下使用相同的数据集对树的多个分支进行正选择测试，则可能需要对多个测试进行校正。
- Bonferroni修正可能过于保守，而Rom程序(Rom 1990)的功效略高，是首选(Anisimova and Yang 2007)。人们还可以使用控制假阳性(FDR)的程序，它是所有被拒绝的零假设中真零的预期比例，或者是所有阳性测试结果中假阳性结果的比例(Benjamini和Hochberg 1995,2000)。需要注意的是，如果这些序列非常分散或存在严重的模型违规，则多重测试校正可能是不可靠的。

## 3.3. 分支模型 (branch model)
1. 分支模型的作用
- 主要用于检测在某个分枝上，其 ω 值是否显著高于背景分枝，即基因在目标分枝上进化速度是否加快。
2. 分支模型的主要假设
- 分支模型认为基因序列上所有位点的 ω 值是一致的；但系统发育树上不同分支的 ω 值是不同的。
3. 分支模型的分析过程
- 对两种模型进行比较：
- (1) 第一种模型为null model，所有分枝具有相同的 ω 值；
- (2) 第二种备择模型认为目标分枝具有一个 ω 值，其它所有分枝具有一个相同的 ω 值。
- 比较两种模型的似然差异，利用卡方检验（自由度为1）算p值。若p值 <= 0.05，且目标分枝上的 ω 值高于背景值，则认为该基因为快速进化基因。
- 一般情况下，该方法计算得到的p值会低于分支位点模型的结果。
4. 分支模型使用的模型
- 分支模型主要用于对系统发育树中不同支系 ω 值差异性进行界定，主要有三个模型：
- One-ratio model : `model = 0, NSites = 0` 假设系统发育树中所有支系的  ω 值相等；
- Free-ratio model :`model = 1, NSites = 0` 假设系统发育树中所有支系的 ω 值不相等；
- Two-ratio model :`model = 2, NSites = 0` 假设前景枝和背景枝的ω 值不同； 
5. 常选的成对模型
- one ratio（负选择） vs. free ratio（自由比率）
- one ratio（负选择） vs. two ratio （正选择）
- two ratio natural（中性进化） vs. two ratio （正选择）

## 3.4. 进化支模型 (clade model)
与分支位点模型类型 (branch-site model)，能同时检测多个进化支 (clade)，该模型并没有将背景支的dN/dS值约束在（0,1）。有CmC和 CmD 两种模型，主要参数如下：
1. 进化支模型使用的模型
- M2a_model: `model = 0, NSsites = 22`
- Clade Model C (CmC): `model = 3, NSsites = 2, ncatG=2 or 3`
- Clade Model D (CmD): `model = 3, NSsites = 3, ncatG=ignored`
2. 常选的成对模型
- Clade Model C vs. M2a_model

# 4. 用codeML的模型执行选择分析的常见场景
根据不同的分析目的，可以选择不同的模型进行选择分析，常见场景包括以下几种。以后还可以每种应用场景写一篇博客。
1. 计算平均选择压 (平均 ω 值 )
- 在所有位点和分支均为 1 个相同的 ω 的模型(m0)下，计算 ω 作为对基因的平均选择压力的度量。
2. 用位点模型 (site model) 检测编码序列中的哪些位点处于正选择
3. 用分支模型(branch model) 检测系统发育树上一个或多个分支是否处于正选择或负选择是否放松
4. 用分支位点模型 (branch-site model) 检测特定分支发生正选择的位点集。
5. codeML的最新实现，可以使用由数千个基因比对组成的基因组数据集进行正选择检测。

# 5. 用codeML进行选择分析的实操
linux系统下使用PAML的codeML，windows系统下可以使用PAML的图形界面版本PAMLX，或者基于PAML的可视化软件EasyCodeML。
## 5.1. 安装PAML
### 5.1.1. 方法一：conda安装【方便快捷】
`conda install -c bioconda paml`

### 5.1.2. 方法二：下载github库并编译【开发者手册推荐】
1. 下载最新版本的PAML：`git clone git@github.com:abacus-gene/paml.git`
2. 用git clone是复制了PAML文件夹，进入paml/src目录，编译

```shell
 # Move to the cloned repository. Make sure you have cloned it in the correct location of your local disk.
 cd paml
 # If there are executable files for Windows, you may remove them.
 rm bin/*.exe
 # Move to the `src` directory to compile the software.
 cd src
 make -f Makefile
 # List all the compiled programs that are part of PAML and
 # remove unnecessary files.
 ls -lF
 rm *.o
 # Create a `bin` directory, which you can then export to your PATH.
 mkdir ../bin
 # Move the compiled programs to `bin`.
 mv baseml basemlg chi2 codeml evolver infinitesites mcmctree pamp yn00 ../bin
 # Test that you can run some PAML programs such as BASEML, CODEML, or evolver.
 bin/baseml
 bin/codeml
 bin/evolver
```

3. 可以把包含所有执行文件的paml/bin/目录放到环境变量中

## 5.2. 运行codeML
### 5.2.1. 必需的输入文件
输入的序列比对文件和树文件的格式可以参考https://github.com/abacus-gene/paml-tutorial/tree/main/positive-selection/00_data 中的 Mx_aln.phy Mx_root.tree，Mx_unroot.tre。
1. 序列比对文件 (cds_aln.phy)
- phylip格式
- 比对好的编码序列 (cds) 文件，碱基数量是3的倍数（codon模式比对）
- 建议删除gaps和难以align的区域，但要按密码子删除（删除和保留都是3的倍数个碱基）
2. 树文件 (tree.newick)
- newick格式，并在首行前加上一行，包括两个数字，一个是物种/样本数量，一个是树的数量（一般是1），空格隔开。
- 只需要拓扑结构，删除枝长、支持率等信息。
- 如果包含多棵树，需要一个头文件。
3. 配置文件 (codeml.ctl)
- codeml.ctl 的例子在安装的程序包paml/examples/下有，可以复制到分析目录下，修改后使用
- codeml.ctl 的参数需要根据使用的模式，运行的目的等具体情况进行修改

### 5.2.2. 配置文件的参数解释
```
* 输入输出参数：
      seqfile = cds_aln.phy *设置输入的多序列比对文件
     treefile = tree.newick      *设置输入的树文件
      outfile = out.txt           *设置输出文件

        noisy = 9  * 0,1,2,3,9: how much rubbish on the screen；设置输出到屏幕上的信息量等级：0，1，2，3，9。
      verbose = 1  * 0: concise; 1: detailed, 2: too much；设置输出到结果文件中的信息量等级：0，精简模式结果（推荐）；1，输出详细信息，包含碱基序列；2，输出更多信息。

       getSE = 0  * 0: don't want them, 1: want S.E.s of estimates.设置是否计算并获得各参数的标准误：0，不需要；1，需要。
    RateAncestor = 1  * (0,1,2): rates (alpha>0) or ancestral states (1 or 2).设置是否计算序列中每个位点的替换率：0，不需要；1，需要。当设置成1时，会在结果文件rates中给出各个位点的碱基替换率；同时也会进行祖先序列的重构，在结果文件rst中有体现。

* 数据使用说明参数：
      runmode = 0  * 0: user tree;  1: semi-automatic;  2: automatic
                   * 3: StepwiseAddition; (4,5):PerturbationNNI; -2: pairwise 。设置程序获取进化树拓扑结构的方法：0，直接从输入文件中获取进化树拓扑结构（推荐）；1，程序以输入文件中的多分叉树作为起始树，并使用星状分解法搜索最佳树；2，程序直接以星状树作为起始树，并使用星状分解法搜索最佳树；3，使用逐步添加法搜索最佳树；4，使用简约法获取起始树，再进行邻近分枝交换寻找最优树；5，以输入文件中的树作为起始树，再进行邻近分枝交换寻找最优树；-2，对密码子序列进行两两比较并使用ML方法计算DnDs，或对蛋白序列进行两两比较计算ML距离，而不进行其它参数（枝长和omega等）的计算。
    *  fix_blength = 1  * 0: ignore, -1: random, 1: initial, 2: fixed, 3: proportional。设置程序处理输入树文件中枝长数据的方法：0，忽略输入树文件中的枝长信息；-1，使用一个随机起始进行进行计算；1，以输入的枝长信息作为初始值进行ML迭代分析，此时需要注意输入的枝长信息是碱基替换率，而不是碱基替换数；2，不需要使用ML迭代计算枝长，直接使用输入的枝长信息，需要注意，若枝长信息和序列信息不吻合可能导致程序崩溃；3，让ML计算出的枝长和输入的枝长呈正比。
      seqtype = 2  * 1:codons; 2:AAs; 3:codons-->AAs ；设置输入的多序列比对数据的类型：1，密码子数据；2，氨基酸数据；3，输入数据虽然为密码子序列，但先转换为氨基酸序列后再进行分析。
    CodonFreq = 2  * 0:1/61 each, 1:F1X4, 2:F3X4, 3:codon table, 6:FmutSel0, 7:FmutSel；设置密码子频率的计算算法，默认是2。：0，表示除三种终止密码子频率为0，其余密码值频率全部为1/61；1，程序分别计算三个密码子位点的四种碱基频率，再算四种碱基频率在三个位点上的算术平均值，计算61种密码子频率时，使用该均值进行计算，这61种密码子频率之和不等于1，然后对其数据标准化，使其和为1，得到所有密码子的频率；2，F3X4，程序分别计算三个密码子位点的四种碱基频率，计算61种密码子频率时，使用相应位点的碱基频率进行计算，这61种密码子频率之和不等于1，然后对其数据标准化，使其和为1，得到所有密码子的频率；3，直接使用观测到的各密码子的总的频数/所有密码值的总数，得到所有密码子的频率。FmutSel模型(CodonFreq = 7)为通用遗传密码的每个密码子分配了60(= 61-1)个密码子适应度参数(Yang and Nielsen 2008)。FmutSel0模型(CodonFreq = 6)是FmutSel的特例，它对同义密码子赋予相同的适应度值，因此只使用19(= 20-1)个氨基酸适应度参数。该模型假设氨基酸频率由蛋白质的功能需求决定，但同义密码子的相对频率仅由突变偏倚参数决定。
    * 一般设置CodonFre的值为2。设置为0让所有密码子频率相等，不符合实际情况；设置为1没有考虑到三种位点各碱基频率的差异，得到的密码子频率不准确；设置为2能比较准确计算出各密码子的频率；设置为3由输入数据得到真实的各密码子频率，但有时候有些非终止密码子的频率为0，可能不利于后续的计算。
    cleandata = 1  * remove sites with ambiguity data (1:yes, 0:no)? 设置是否移除不明确碱基列（N、？、W、R和Y等）和含gap的列后再进行数据分析：0，不移除，但在序列两两比较的时候，还是会去除后进行比较；1，移除，有可能会导致运行失败。建议自行删除歧义和gaps数据，并在这里选保留(0)。
        ndata = 1 * 设置输入的多序列比对的数据个数。可以合并多序列比对phylip文件，然后批量化处理。这里设置大于1时，会依次分析每一个多序列比对，保存在结果文件out.txt中。
        clock = 0  * 0:no clock 不假设分子钟, 1:clock 严格分子钟; 2:local clock 本地分子钟; 3:CombinedAnalysis 联合分析。 设置进化树中各分支上的变异速率是否一致，服从分子钟理论：0，变异速率不一致，不服从分子钟理论（当输入数据中物种相差较远时，各分支变异速率不一致）；1，所有分枝具有相同的变异速率，全局上服从分子钟理论；2，进化树局部符和分子钟理论，程序认为除了指定的分支具有不同的进化速率，其它分枝都具有相同的进化速率，这要求输入的进化树信息中使用#来指定分支；3，对多基因数据进行联合分析。通常选0不假设分子钟，并使用无根树。
        Mgene = 0  * codon: 0:rates, 1:separate; 2:diff pi, 3:diff kapa, 4:all diff。设置是否有多个基因的多序列比对信息输入，以及多各基因之间的参数是否一致：0，输入的多序列比对文件中仅包含一个基因时，或多个基因具有相同的Kappa和Pi参数；1，输入文件包含多个基因，这些基因之间是相互独立的（这些基因之间具有不同的Kappa和Pi值，且其进化树的枝长也不相关）；2，输入文件包含多个基因，这些基因具有相同的Kappa值，不同的Pi值；3，输入文件包含多个基因，这些基因具有相同的Pi值，不同的Kappa值；4，输入文件包含多个基因，这些基因具有不同的Kappa值和不同的Pi值。当值是2、3或4时，多个基因的进化树枝长虽然长度不一样，但是呈正比关系。这点和参数值等于1是不一样的。
        icode = 0  * 0:universal code; 1:mammalian mt; 2-10:see below ； 设置遗传密码。其值1-10和NCBI的1-11遗传密码规则对应：0，表示通用的遗传密码。
        * Genetic codes: 0:universal, 1:mammalian mt., 2:yeast mt., 3:mold mt., 4: invertebrate mt., 5: ciliate nuclear, 6: echinoderm mt., 7: euplotid mt., 8: alternative yeast nu. 9: ascidian mt., 10: blepharisma nu.
        * These codes correspond to transl_table 1 to 11 of GENEBANK.
       Small_Diff = .5e-6 * 设置一个很小的值，一般位于1e-8到1e-5之间。推荐检测设置不同的值在比较其结果，需要该参数值对结果没什么影响。

* 位点替换模型参数：
        model = 2  * models for codons:
                       * 0:one, 1:b, 2:2 or more dN/dS ratios for branches
                   * models for AAs or codon-translated AAs:
                       * 0:poisson, 1:proportional, 2:Empirical, 3:Empirical+F
                       * 6:FromCodon, 7:AAClasses, 8:REVaa_0, 9:REVaa(nr=189)
                       * 若输入数据是密码子序列，该参数用于设置branch models，即进化树各分枝的omega值的分布：0，进化树上所有分枝的omega值一致；1，对每个分枝单独进行omega计算；2，设置多类omega值，根据树文件中对分枝的编号信息来确定类别，具有相同编号的分枝具有相同的omega值，没有编号的分枝具有相同的omega值，程序分别计算各编号和没有编号的omega值。
                        *若输入数据是蛋白序列，或数据是密码子序列且seqtype值是3时，该参数用于设置氨基酸替换模型：0，Poisson；1，氨基酸替换率和氨基酸的观测频率成正比；2，从aaRatefile参数指定的文件路径中读取氨基酸替换率信息，这些信息是根据经验获得，并记录到后缀为.dat的配置文件中。这些经验模型(Empirical Models)文件位于PAML软件安装目录中的dat目录下，例如，Dayhoff(dayhoff.dat)、WAG(wag.dat)、LG(lg.dat)、mtMAN(mtman.dat)和mtREV24(mtREV24.dat)等；
      NSsites = 0  * 0:one w;1:neutral;2:selection; 3:discrete;4:freqs;
                   * 5:gamma;6:2gamma;7:beta;8:beta&w;9:beta&gamma;
                   * 10:beta&gamma+1; 11:beta&normal>1; 12:0&2normal>1;
                   * 13:3normal>0
                   * 输入数据时密码子序列时生效，用于设置site model，即序列各位点的omega值的分布：0，所有位点具有相同的omega值；1，各位点上的omega值小于1或等于1（服从中性进化neutral）；2，各位点上的omega值小于1、等于1或大于1（选择性进化selection）；3，discrete；4，freq；5:gamma；6，2gamma；7，beta；8，beta&w；9，beta&gamma；10，beta&gamma+1；11，beta&normal>1；12，0&2normal>1；13，3normal>0。
                   * 可以一次输入多个模型进行计算并比较，其结果输出的rst文件中。
   aaRatefile = ../dat/jones.dat  * only used for aa seqs with model=empirical(_F)
                   * dayhoff.dat, jones.dat, wag.dat, mtmam.dat, or your own。
                   * 当对蛋白数据进行分析，且model = 2时，该参数生效，用于设置氨基酸替换模型。
       aaDist = 0  * 0:equal, +:geometric; -:linear, 1-6:G1974,Miyata,c,p,v,a。设置氨基酸之间的距离。
    fix_kappa = 0  * 1: kappa fixed, 0: kappa to be estimated ；kappa是转换/颠换的比率。设置是否给定一个Kappa值：0，通过ML迭代序列来估算Kappa值；1，使用kappa参数设置一个固定的Kappa值。
        kappa = 2  * initial or fixed kappa。设置一个固定的Kappa值，或一个初始的Kappa值。当fix_kappa = 1时此参数生效。
    fix_omega = 0  * 1: omega or omega_1 fixed, 0: estimate ；设置是否给定一个omega值：0，通过ML迭代序列来估算Kappa值；1，使用omega参数设置一个固定的omega值。
        omega = .4 * initial or fixed omega, for codons or codon-based AAs ；设置一个固定的omega值，或一个初始的omega值。当fix_omega = 1时此参数生效。
    fix_alpha = 1  * 0: estimate gamma shape parameter; 1: fix it at alpha。
                * 序列中不同的位点具有不同的碱基替换率，服从discrete-gamma分布，该模型通过alpha（shape参数）和ncatG参数控制。该参数设置是否给定一个alpha值：0，使用ML方法对alpha值进行计算；1，使用alpha参数设置一个固定的alpha值。
                *对于密码子序列，当NSsites参数值不为0或model不为0时，推荐设置fix_alpha = 1且alpha = 0，即不设置alpha值，认为位点间的变异速率一致，否则程序报错。若设置了alpha值，则程序认为不同密码子位点的变异速率不均匀，且同时所有位点的omega值一致，当然各分枝的omega值也会一致，这时要求NSsites和model参数值都设置为0（这一般不是我们需要的分析，它不能进行正选择分析了）。
        alpha = 0. * initial or fixed alpha, 0:infinity (constant rate)
                * 设置一个固定的alpha值或初始alpha值（推荐设置为0.5），fix_alpha = 1时此参数生效。。该值小于1，表示只有少数热点位置的替换率较高；该值越小，表示位点替换率在各位点上越不均匀；若设置fix_alpha = 1且alpha = 0则表示所有位点的替换率是恒定一致的。
       Malpha = 0  * different alphas for genes。当输入的多序列比对结果中有多基因时，设置这些基因间的alpha值是否相等：0，分别对每个基因单独计算alpha值；1，所有基因的alpha值保持一致。
        ncatG = 8  * # of categories in dG of NSsites models。
                    * 序列中不同位点的变异速率服从GAMMA分布的，ncatG是其一个参数，一般设置为5，4，8或10，且序列条数越多，该值设置越大。
                    *对于密码子序列，当NSites设置为3时，ncatG设置为3；当NSites设置为4时，ncatG设置为5；当NSites值设置>=5时，ncatG值设置为10。
       method = 0  * Optimization method 0: simultaneous; 1: one branch a time。设置评估枝长的ML迭代算法：0，使用PAML的老算法同时计算所有枝长，在clock = 0下有效；1，PAML新加入的算法，一次对一个枝长进行计算，该算法仅在clock参数值为1，2或3下工作。
```

### 5.2.3. 运行命令
`codeml codeml.ctl`

### 5.2.4. 结果文件
1. 主要结果文件在codeml.ctl设置的outfile中，例子中默认是**mlc**。
2. 主要关注mlc文件中的两个信息
- lnL是似然值(likeilhood value)的自然对数，之所以是负数，是因为计算出似然值是一个非常小的小数，如果不取对数，结果显示就是0，难以使用。
- omega (dN/dS)

```
lnL(ntime: 27  np: 29): -29984.121043      +0.000000
...
Detailed output identifying parameters

kappa (ts/tv) =  1.98351

omega (dN/dS) =  0.08975
```
3. 似然比检验(likelihood-ratio test, LRT)
- 通常在零假设模型和备择假设模型下各自得到lnL值和omega值，还需要对两个似然值进行似然比检验(LRT)。
- LRT是指根据两个竞争的统计模型的似然值的比值，评估两者的拟合度，其中一个是最大化整个参数空间，另一个则是做一些限制。如果限制条件（零假设）被观测数据所支持，那么两者的似然值的差异不会超过抽样误差。
- 因此，LRT检验的是，比值是不是和1有显著区别，或者说比值的自然对数和0有显著区别。
4. 使用卡方检验(chi2 test)来分析LRT的统计量是否有显著性，即零假设模型和备择假设模型下的似然值的差异是否显著。

# 6. 用codeML的模型执行选择分析的常见问题
## 6.1. 有根树和无根树的选择
1. 有根树的祖先节点是二叉树，而无根树为三叉树。
2. 零模型在所有测试中都是M0，它假设所有分支都是相同的ω。除检验4中的备择假设外，所有模型都使用无根树，在备择假设中，树的根周围的两个分支被赋予不同的ω比，因此需要有根树。
3. 在使用codeml时，如果没有指定有根树参数却使用了有根树作为输入，那么在输出结果中将会得到这样的报错信息："This is a rooted tree. Please check!"。
4. 对于大多数模型，即使使用有根树，其该模型似然值仍然是正确的，但是root周围两个分支的长度不稳定，因为它们的和是估计值。对于其他模型，似然估计和参数估计都是不正确的。因此，分析时确实应该注意到这一信息，并**尽可能使用一棵无根树**。

## 6.2. 是否删除多序列比对中的gap与模糊字符
1. 在多序列比对过程中，对齐gap是极其困难的，paml软件包无法处理gap。因此，我们可以通过设置cleandata = 1来去除gap；此外，还可以将gap当作为模糊字符进行处理。但是，这都不是最好的解决办法，这两种策略都低估了序列差异。
2. 除了一个或两个序列之外，大多数序列都有序列信息的位点也许应该保留，而除了一个或两个序列之外，所有序列都有对齐间隙的位点最好被移除。
3. 因此，选择合适的多序列比对软件以及过滤软件，在codeml分析前就去除gap和歧义序列，然后参数cleandata = 0是更好的选择。

## 6.3. 当前景支的dN/dS值显著大于背景支时，该如何解释
1. 如果检测前景支时，其dN/dS > 1时，我们可以认为它受到正选择作用。
2. 但是，如果其dN/dS <1但大于背景支时，就不能认为它是受正选择作用的，选择压力约束放松可能是较为合理的解释。
3. 此外，在Ohta's的微有害突变假说下，净化选择在大种群中比在小种群中更有效，因此不同谱系的种群规模的差异提供了另一个相容的假设。如果氨基酸的变化是稍微有害的，我们预计在大群体中它们从群体中移除的速度会比在小群体中更高。
4. 因此，即使两系在选择压力或基因功能上没有差异，我们也期望在一个大群体中看到一个较小的dN/dS比值，例如，许多核基因的dN/dS比值在啮齿类动物中低于灵长类或偶蹄类。

## 6.4. 不同的模型鉴定出不同的位点，该选择相信哪一种模型？
1. 通常，如果一个位点在一种模型下出现在选择列表中，那么在另一种模型下也会有相当大的概率。
2. 确定位点的问题很困难，而且容易出错。因此，我们通常会认为后验概率大于95%或者99%的位点，是比较可信的。

## 6.5. 如何标记前景支
1. 如果标记某一个节点，则可以使用”#”；如果是标记某一类群（节点和节点后的所有类群），则可以使用”$”。
2. 符号#的优先级高于\$，树顶端的进化枝标签优先于接近根处的祖先节点的进化枝标签。
3. 位点模型(Site models)和free ratio模型分析不需要标记前景支。但其他三个与枝相关的模型均需要提前进行前景枝的标记。枝模型(Branch model)和进化枝模型(Clade model)可以在单次分析中标记多个前景枝(或进化枝)，但枝位点模型(Branch-site model)只能一次标记一个前景枝。

## 6.6. dN/dS代表进化速率而非突变率
1. 较高的dN/dS值可以解释为正选择或者快速进化。尽管突变本身也处于选择压力之中（大多数为净化选择），但不可以解释为“基因A增加了突变的选择压力”，因为突变是随机发生的。原则上讲，突变率会同时影响dN、dS，但通常dN/dS不受突变率影响。
2. dN/dS是一种进化速率，但它不是突变速率，因为同义和非同义替代率具有不同的选择约束水平。 选择压力测试的基本原理是：它假设同义替换是中性进行，也就是说，它们大多是在遗传漂移下进化的。如果这是真实情况，那么dS可以用作（中性）突变率的替代。然而，非同义替代率总是处于净化选择压力，在正选择下程度较小。
3. 因此，dN/dS是中性偏离的度量。所以，dN > dS，既dN/dS > 1,则受到正选择；如果dN小于dS，则dN/dS < 1，则是净化选择。选择压力测试的关键就是它通过特定基因的同义替换的“中性”进化速率归一化非同义替换的速率。

## 6.7. 基因树or物种树
1. 在任何情况下，使用代表真实演化历史的基因树都是最好的。
2. 但是，有时可能无法轻易判断是否符合真实演化历史，那么你可以选择物种树作为代替。
3. 基因组水平分析，那么推荐使用物种树。你可以可以使用基因树和物种树进行数据稳健性测试。

## 6.8. 祖先节点有正选择信号而整个枝系却没有检测到如何解释
1. 如果以哺乳动物祖先为前景，假设该基因在共同祖先中存在适应性，可能是由于获得了新的功能，但随后该基因在净化选择下得到了保守进化。如果你把整个clade作为前景，那么假设该基因在整个哺乳动物中的所有分支都承受着持续的压力来改变或多样化，如果该基因涉及到防御或免疫，则可能是这种情况。
2. 对于到底是检测祖先，还是整个枝系，取决于生物学问题。例如，溶菌酶在所有colobine猴子中都应该具有相同的功能，因此蛋白质在clade内被期望受到选择性约束，但在colobine clade的分支上，该酶显然获得了一个新的功能，其正选择驱动了氨基酸的变化。有了这个假设，你就应该把分支的祖先贴上clade的标签，而不是那些在clade中的分支。

## 6.9. Clade模型判别背景支是否同样受正选择
1. 经常遇到审稿人的审稿意见是这样的：前景分支上正选择的基因的重要支持并不意味着在背景分支上没有正选择，这些基因可能仍在许多（即使不是全部）背景分支上处于正选择状态。
2. 为了检验原假设（基因仅在前景支受到正选择）是否正确，可以进一步通过Clade模型检验，因为进化枝模型允许在不限制背景dN/dS小于1的约束下估计前景支与背景支dS/dN的比率。

## 6.10. 基因受到正选择但无正选择位点
1. 一种可能性是，对整个基因有积极选择的证据，但每个单独位点的信息或证据太弱。 可以查看rst文件，该文件具有所有站点的后验概率，以查看是否是这种情况，mlc文件只列出后验概率高于0.5的文件。

## 6.11. 有正选择位点但是通过序列比对却找不到
1. 可能是因为codeml在删除gap或模糊字符的列，之后重新编号位点了（cleandata =1）。

## 6.12. dN/dS（ω）值过大，结果是否可信？
1. 遇到这种极大值的dN/dS时，比如ω = 999，首先要确保你的序列是否正确；其次，该位置的dn，ds是否远小于0.0001，枝长是否过小。
2. 显然，高度相似的序列和非常发散的序列都不是信息丰富的，很难指定确切的值。
3. 为了避免此类问题的出现，可以先通过M0模型获得枝长，再将具有分支长度的进化树应用到codeml，并在ctl中设置FIX_blength=2。

## 6.13. Branch-site检测适应性趋同进化
1. 如图所示，红色分支代表表型趋同的进化分支，如果你想通过分支位点模型来检测适应性趋同进化，应该将所有红色分支统一设置为前景分支。当然，前提是假设所有前景支均具有相同的位点受到正选择。 对于背景支是否同样存在类似的适应性趋同，可以通过clade进化支模型进行检测。

## 6.14. 若p2（正选择位点概率）接近于0
1. p0/ω0,p1/ω1,p2=(1-p0-p1)/ω2：在正选择分析的备择假设结果文件中，通常会获得三个p值，其中，p0代表处于净化选择下的位点概率；p1代表处于中性进化下的位点概率；p2则代表处于正选择下的位点概率。

## 6.15. 分支模型检测正选择
1. 可以使用two ratio备择假设（fix_omega = 0 omega = 1）与two ratio零假设（fix_omega = 1 omega = 1）进行统计假设检验。

## 6.16. 选择约束放松
1. codeml检测选择约束放松可以通过2步完成：首先，确定dN/dS显著增加的情况（由于正选择或者选择约束放松）；然后，过滤掉显著正选择的情况。

## 6.17. 不同模型下的起始ω
1. 针对于CladeC和CladeD模型，一般要设置几个不同的起始ω，测试其lnL值是否稳定（ω=0.001，ω=0.01，ω=0.1，ω=0.5，ω=1，ω=1.5），最终一般会选取lnL较大的值作为最终值。

## 6.18. 枝长会影响PAML结果
1. 正常分析，应该先用M0估计树的枝长，Kappa值，然后用跑出来的树作为初始树，并设置fix_blength = 2。

## 6.19. 基因家族选择压力分析
1. https://groups.google.com/g/pamlsoftware/c/bsfGWYHPCgA
2. https://groups.google.com/g/pamlsoftware/c/KqhrPFYnWMI

## 6.20. CladeC检测正选择
1. CladeC经常用于检测，不同分支的分化选择压力，但有时候检测到前景枝dN/dS>1，此时我们需要进一步利用CladeC的零假设进一步测试（fix_omega=1,omega=1），或者利用branch-site进一步验证一致性。

## 6.21. Free ratio检测结果是否可以作为正选择的证据
1. free ratio模型估计通常会造成较大的抽样误差，例如，较短的分支通常会具有较大的dS/dN。所以，一般对于free ratio出现dN/dS > 999或者dN、dS < 0.0005结果不建议采用。

## 6.22. CladeC模型检测多组支系选择压力差异
1. 在dataset中，有三个clade：A, B, C，那么在clade 1和clade 2之间是否存在显著差异？
- 假设，前景支的标注为`((A1,A2)$1 , (B1,B2)$2 , (C1,C2)$0);`
- 首先，为了测试clades之间的显着差异，可以比较CladeC与M2a_rel。M2a_rel假设$1、$2、$0等都是在相同的选择压力下进化的，这个测试应该有两个自由度。
- 其次，为了测试clade A和B之间的显着差异，同时允许clade C是不同的，您可以比较使用上面提供的树运行CMC与使用更简单的树运行CMC的契合程度。在这种情况下，更简单的树会将clade A和B分配给同一个组，这个测试应该有一个自由度。

## 6.23. 多个前景支的支位点模型检测
当数据集中有多个前景支时：1）进行多个测试，然后在每个测试中设置一个感兴趣的分支作为前景分支；2）只进行一次测试，在此测试中中将所有感兴趣的分支设置为前景分支。那么，此时又会出现另一个问题就是进行多个测试时其他感兴趣分支是否应该被去除？这也许应该取决于具体的生物学问题。

## 6.24. p值校正
如果校正之后没有显著结果，我们可以选择adjP排序。
参考：https://groups.google.com/g/pamlsoftware/c/vty5QrRCUCk

## 6.25. 分支位点模型 (branch-site model) 与位点模型 (site model) 的比较
1. 位点模型 (site model) 的一个相当大的缺点是，ω 是以位点所有密码子的平均值计算的。因此，位点模型不适用于 ω 在不同品系之间也有差异的数据。
2. 分支位点模型 (branch-site model) 也是寻找正选择的位点，但会预先指定可能出现不同 ω 率的支系。
- 相比于位点模型的优点是考虑了不同的分枝具有不同的选择压，即具有不同的 ω 值。
- 分支位点模型将不同支系序列被先验地划分为可能发生正选择的前景支系群和只发生净化选择或中性进化的背景支系群。
3. 分支位点模型让目标分枝具有一个不同的 ω 值，并没有让所有分枝的 ω 值独立进行计算（理论上这样是最好的）。这样算法很复杂，程序运行非常非常消耗时间。但其实也没必要这样做，因为正选择分析其实是两条序列比较后，分析dN/dS，再找正选择位点，其分析结果就应该是某个分枝上基因是否受到正选择，在序列某个位点上受到正选择。

## 6.26. 正选择基因的判断
- Q：若在目标分枝上，其 ω 值小于1，但是却能找到正选择位点。即该基因在该分枝上的dN/dS < 1，但是在某些位点上，dN/dS > 1。那么该基因是否属于正选择基因？ 
- A：属于。之所以为正选择基因，主要是因为基因的个别位点或多个位点存在正选择。当只有个别位点受到正选择压时，而其它多个位点存在纯化选择时，可能导致整体上的 ω 值小于1。此时，该基因也应该是属于正选择基因。

# 7. reference
1. PAML User Guide：https://github.com/abacus-gene/paml/blob/master/doc/pamlDOC.pdf
2. PAML 中文用户手册：https://blog.sciencenet.cn/blog-3433349-1241310.html
3. PAML开发者网站：http://abacus.gene.ucl.ac.uk/software/#phylogenetic-analysis-by-maximum-likelihood-paml
4. 陈连福博客-基因正选择分析原理：http://www.chenlianfu.com/?p=3084%E5%92%8Chttp://blog.sciencenet.cn/blog-460481-1163040.html
5. 视频-使用codeml的分支模型（branch test）检测进化树上某一支的选择压力：https://www.bilibili.com/video/av10469605/?from=search&seid=8859495264060497812&vd_source=86d4997d664e6bb659b1d0f0dcd15267
6. wiki-Ka/Ks ratio：https://en.wikipedia.org/wiki/Ka/Ks_ratio
7. paml检测正选择的初学者指南：https://wap.sciencenet.cn/blog-3434047-1389140.html?mobile=1
8. PAML选择压力分析：https://yuzhenpeng.github.io/2019/12/24/paml/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>