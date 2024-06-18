---
title: 用软件Quartet Sampling来计算分支支持率，评估系统发育树不一致程度，区分强冲突和弱支持
date: 2022-11-08
categories: 
- bioinfo
- phylogeny
tags: 
- phylogenomics
- phylogeny
- phylogenetic discordance
- Quartet Sampling
description: 记录了软件Quartet Sampling的原理和使用。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=105143&auto=1&height=32"></iframe></div>


# 1. Quartet Sampling（QS）method
Quartet Sampling（QS）是为系统发育树量化分支的支持率的一种方法，用于区分植物系统发育树上的强烈拓扑冲突和信息不足（缺乏支持）导致的支持率低，即区分强冲突（不一致）和弱支持（不确定性）。

## 1.1. 原理
### 1.1.1. 四重采样
1. 对于给定的系统发育树，每个内部枝将树划分为四个不重叠的分类群子集（图A）。
2. 这四组分类群（也被称为元-四重奏，meta-quartet）可以以三种拓扑关系存在：与给定拓扑一致的关系，以及两种替代的不一致关系（Replicate 1和Replicate 2）。
3. QS方法从四个子集中的每一个子集中重复随机采样一个分类单元，然后基于给定序列数据（完全比对或随机采样的基因分区），评估跨越该特定分支的随机选择的四条序列（即四重奏，quartet）的所有三个可能系统发育拓扑的似然值。
4. 对于同一分支采样的所有四重奏，评估这四个采样分类群可以采用的所有三种可能拓扑的可能性。
5. 然后记录所有重复中具有最佳可能性的四重拓扑并制成表格。
6. 此过程生成一组计数（跨每个分支的所有复制），其中一致或两个不一致关系中的每一个都具有最佳可能性。
7. 该过程可以通过评估每个四重奏的完全比对的可能性（即在单矩阵框架中）或通过从多基因或全基因组比对（即在多基因树中）的单个基因/分区比对中随机抽样来执行聚结框架）。

<img src="https://bsapubs.onlinelibrary.wiley.com/cms/asset/e55c64b7-cc9d-49bb-92c9-6e59e0661dde/ajb21016-fig-0001-m.jpg" width=90% title="四重采样方法的描述" align=center/>

**<p align="center">图1. 四重采样方法的描述**
图源：https://bsapubs.onlinelibrary.wiley.com/doi/full/10.1002/ajb2.1016</p>

### 1.1.2. 评估分数
QS 方法使用这些重新采样的四重树计数来计算焦点树的每个内部分支的三个分数(QC,QD,QI）和评估分类单元的QF分数。

总的来说，这四个分数提供了区分信息一致性（Quartet Concordance，QC），次生进化历史的存在（Quartet Differential，QD），信息量（Quartet Informativeness，QI）和树中单个分类群的可靠性（Quartet Fidelity，QF）的方法。 

QS方法把这些影响分开，而不是像系统发育支持的标准测量那样将它们混为一个单一的汇总支持率。

1. Quartet Concordance (QC) score - 四重奏一致性分数
- QC分数是一种类似熵的度量，量化了四种分类群的三种拓扑之间的相对支持度。
- 即支持率在三种拓扑中分配越平均，QC值越接近0；越不均，约远离0。
- 当频率最高的拓扑与输入树一致时，QC 值在 (0,1] 范围内为正。
- 当所有四重树与输入树的焦点分支一致时，QC 等于 1。
- 当其中一个不一致的拓扑是最常见的采样四重奏，QC 值在 [-1,0) 范围内为负。
- 当所有四重奏树都是两个不一致的系统发育拓扑之一时接近 -1。
- 当三个拓扑中平均分配时，QC 等于 0（或者如果三个可能中只有两个被注册为在所有复制中具有最佳可能性，则在两个之间平均分配）。
2. Quartet Differential (QD) score - 四重奏差异分数
- QD分数使用基因渐渗的f和D统计量的逻辑（尽管使用四重拓扑比例，而不是位点频率），并测量两个不一致拓扑的采样比例之间的差异。
- QD 分数没有具体量化基因渗入，也没有识别基因渗入的分类群，但确实表明一种拓扑的频率高于另一种拓扑。
- QD 的低值表明在两个不一致的拓扑结构之间存在一个倾向性的拓扑结构，这也可能表明在给定分支上有背景谱系分选之外的有偏（biased）生物过程的，包括混杂变量，例如基因渗入、强烈的异质性、异质碱基组成等。
- QD 表示支持率在两种替代拓扑上的差异水平，在 [0,1] 范围内变化，值为 1 表示两棵替代拓扑树的比例没有偏斜（skew），值为0 表示采样的所有不一致树仅来自两种替代关系之一。
3. Quartet Informativeness (QI) score - 四重奏信息分数
- 在给定最佳和次佳的差异阈值的基础上，最佳似然四重树的似然值超过了次优似然值的四重树。这种情况下，QI分数量化了给定分支的复制比例。
- 该分数表明，当分子数据在拓扑上有效地模棱两可时（即，当三个可能的四重拓扑中的两个或所有三个具有几乎无法区分的似然分数时），重复不被视为一致或不一致。
- QI在 [0,1] 范围内变化，表示超过阈值的采样四重奏的比例。
- QI 值为 1 表示所有四重奏都提供信息，而值为 0 表示所有四重奏都不确定（即，对于给定分支没有重要信息）。
- QI 代表了分支的信息量，与 QC 和 QD 结合使用，以区分具有低信息的分支与具有冲突信息（即高度不一致）的分支。
4. Quartet Fidelity (QF) score - 四重奏保真度分数
- 对于每个末端分类单元，计算 QF 分数以报告总复制的比例（跨所有分支），包括导致一致四重奏拓扑的给定分类单元。
- 因此，QF 的使用与“rogue taxon”测试的方法相似。
- 然而，一个重要的区别是使用分类群完整的引导复制来计算这些分数而不是重新采样的子树，因此在系统发育分析中会遇到与引导得分本身相同的问题（即，当所有引导时，RogueNaRok 算法不会报告rogue分类分数为 100）。
- 对于给定的分类单元，QF 分数在 [0,1] 范围，作为涉及与焦点树分支一致的分类单元的四重拓扑的比例。
- 因此，QF 值为 1 表示当一个给定的分类单元在四重奏中采样时，总是在所有内部分支中产生一致的拓扑结构。
- QF 值接近 0 表明涉及该分类单元的拓扑结构大多不一致，也可能表明序列质量或比对性（identity）较差，或者表明存在使系统无序的特定的谱系，或者分类单元在给定树中明显错位。
- 请注意，QF 与 QC、QD 和 QI 的具体不同之处在于它是跨内部分支评估的**特定分类单元**的评估，而不是内部分支的评估。

## 1.2. 程序
主要包含以下几个脚本：
1. pysrc/quartet_sampling.py：主程序，用于实现Quartet Sampling（QS）方法。
2. pysrc/merge_output.py：合并quartet_sampling.py程序的输出文件RESULT.node.scores.csv，当使用同一拓扑树运行多次时，用这个脚本合并多次运行的结果到一套结果。
3. pysrc/query_tree.py：树查询脚本，从运行后注释好的大型树中找到特定节点。
4. pysrc/utils/calc_qstats.py：用于对quartet_sampling.py程序的输出文件RESULT.node.scores.csv进行基础统计。
5. pysrc/utils/fasta2phy.py：用于把fasta格式序列转换成relaxed phylip格式。

## 1.3. 安装
无需安装，在已安装python3和RAxML的基础上，脚本可直接运行。

## 1.4. quartet_sampling.py程序
quartet_sampling.py脚本用于进行Quartet Sampling（QS）计算。
### 1.4.1. 输入
拓扑树和比对序列中的样品名称必须一致和对应。可以有多余的序列（拓扑树上没有的），程序会忽略；但不能有拓扑树上出现但缺失序列的情况。
1. 拓扑树(sample.tre)
- newick格式
- 不需要枝长数据，如果有会被忽略
- 建议去除方括号和方括号内的支持率
2. 比对好的序列(sample_aligned.phy)
- relaxed phylip格式
- DNA核酸序列或者氨基酸序列，如果是氨基酸序列在参数中加上`--amino-acid`
- `utils/fasta2phy.py`脚本可以把fasta格式转为relaxed phylip格式
3. partition分区文件【可选】
- 是可选的输入文件
- RAxML格式
- 用作串联比对序列的分区，或者用作单基因树的基因分区。
- 如果序列稀疏到某些分区包含空序列，建议用`--ignore-error`参数忽略空分区造成的RAxML错误。

### 1.4.2. 运行
1. 运行命令
- `python3 quartet_sampling.py --tree sample.tre --align sample_aligned.phy --reps 1000 --threads 4 --lnlike 2  --verbout`

2. 参数
- --tree：必需，指定拓扑树
- --align：必需，指定aligned序列
- --reps：必需，每个枝计算的重复次数，默认100，推荐1000
- --threads：必需，每个枝的重复的平行运算的线程，默认1
- --lnlike：log-likelihood阈值，默认2。用来指定最好似然树超过第二好似然树的最小差异（对每个枝来说）。推荐指定，如果不指定，则调用简单模式，用RAxML从比对序列推断树，不评估似然值，所有枝上的QI分数为0。
- --data-type：指定数据类型，默认nuc，可选amino，cat
- --engine：指定推断树和评估树模型似然值的软件，默认raxml-ng，可选raxml，paup，iqtree
- --partitions：指定分区文件（RAxML格式）。
- --genetrees：指定分区文件，用分区文件划分比对好的序列成分离的基因树区域。
- --ignore-errors：忽略RAxML和PAUP运行错误。
- --clade：执行CSV taxon list指定的特定枝的分析。
- --calc-qdstats：实验性参数，为QD树频率做卡方检验。只有脚本可用的时候才能用。
- --result-prefix：指定结果文件前缀
- --verbose：指定后提供更详尽的输出，生成RESULT.verbose文件。
- --verbout：指定后提供每个topology和QC的频率的输出，生成RESULT.verbout文件。
- --min-overlap 20000：在四分体中所有类群被采样需要的最少位点数，默认不设置，推荐20000。

3. 速度
- 速度很快，6Mb多的snp序列数据40分钟跑完。

### 1.4.3. 结果文件
1. RESULT.node.scores.csv
- csv格式文件，包含以下信息：
- node_label
2. RESULT.node.counts.csv
- tsv格式文件，包含以下信息：
3. RESULT.labeled.tre
- 一棵newick树，每个中间枝都含有QS##标识符
4. RESULT.labeled.tre.freq/qc/qd/qi
- 一棵newick树，每个中间枝都含有一致性重复频率(frequency of concordant replicates)/QC/QD/QI分数标识符
5. RESULT.labeled.tre.figtree
- FigTree格式的树，包含所有QS分数和中间枝的QC/QD/QI分数。
6. RESULT.run.stats
- 记录了nonoverlapping_count
7. RESULT.verbout
- csv格式文件，每个topology和QC的频率，标题行是topo1,topo2,topo3,topou,qc,qd,qi。

## 1.5. calc_qstats.py程序
1. 运行
- `/path/to/pysrc/utils/calc_qstats.py --data RESULT.node.scores.csv`
2. 输出

```
data read
29 26
QC
Mean (IQR)
0.54 (0.31,0.94)
QD
Mean (IQR)
0.13 (0.0,0.14)
QI
Mean (IQR)
1.0 (1.0,1.0)
QF
Mean (IQR)
0.8 (0.78,0.84)
```

# 2. 可视化脚本QS_visualization
Quartet Sampling开发者推荐了一个可视化脚本https://github.com/ShuiyinLIU/QS_visualization，直接在quartet_sampling.py生成的结果目录下运行R脚本plot_QC_ggtree.R即可得到用于出版的结果。

这个脚本调用ggtree和ggplot2来画图，可以根据需要自行修改参数画图。

1. 运行plot_QC_ggtree.R
- 在Linux环境中运行`Rscript plot_QC_ggtree.R`
- 在Windows的Rstudio中运行`source("plot_QC_ggtree.R")`

2. 结果文件
- 00.treeQC_rect_circ.pdf：可视化结果，一共三页，三张图。
- tree_qc/qd/qi.tre：把quartet_sampling.py生成的RESULT.labeled.tre.qc/qd/qi结果改格式生成的newick树文件，QC/QD/QI值作为节点标签。

00.treeQC_rect_circ.pdf示例文件：https://github.com/ShuiyinLIU/QS_visualization/blob/f2a08588d2f5cd40ce952ef4cad2f7f55144e28f/00.treeQC_rect_circ.pdf

3. plot_QC_ggtree.R脚本

```R
list.files()

library(ggtree)
library(treeio)
library(ggplot2)
library(ape)

qc <- read.tree("RESULT.labeled.tre.qc")
qd <- read.tree("RESULT.labeled.tre.qd")
qi <- read.tree("RESULT.labeled.tre.qi")


# process node labels of above three labeled trees
# qc tree
tree <- qc
tree$node.label <- gsub("qc=","",tree$node.label)
View(tree$node.label)
write.tree(tree,"tree_qc.tre")
# qd tree
tree <- qd
tree$node.label <- gsub("qd=","",tree$node.label)
View(tree$node.label)
write.tree(tree,"tree_qd.tre")
# qi tree
tree <- qi
tree$node.label <- gsub("qi=","",tree$node.label)
View(tree$node.label)
write.tree(tree,"tree_qi.tre")


# read 3 modified tree files for QC/QD/QI
tree_qc <- read.newick("tree_qc.tre", node.label='support')
tree_qd <- read.newick("tree_qd.tre", node.label='support')
tree_qi <- read.newick("tree_qi.tre", node.label='support')


# add a customized label for internode or inter-branch, i.e., qc/qd/qI
score_raw = paste(tree_qc@data$support,"/",tree_qd@data$support,"/",tree_qi@data$support,sep="")
score = gsub("NA/NA/NA","",score_raw)
score = gsub("NA","-",score)
View(score)


# set labeled QC tree as the main plot tree
tree <- tree_qc
tree@data$score <- score


#####################################################
# Partitioning quartet concordance. QC values were divided into four categories and this
# information was used to color circle points. 

# drop the internodes without QC vaule
root <- tree@data$node[is.na(tree@data$support)]

pdf(file="00.treeQC_rect_circ.pdf", width = 16, height = 18)

# (1)color circle points
ggtree(tree, size=0.5) +
  geom_tiplab(size=2) + xlim(0, 0.12) +
  geom_nodepoint(aes(subset=!isTip & node != root, fill=cut(support, c(1, 0.2, 0, -0.05, -1))),
                 shape=21, size=4) +
  theme_tree(legend.position=c(0.9, 0.18)) +
  scale_fill_manual(values=c("#2F4F4F", "#98F898", "#FFCC99","#FF6600"),
                    guide="legend", name="Quartet Concordance(QC)",
                    breaks=c("(0.2,1]","(0,0.2]","(-0.05,0]","(-1,-0.05]"),
                    labels=expression(QC>0.2, 0 < QC * " <= 0.2", -0.05 < QC * " <= 0", QC <= -0.05))

# (2)color branch
ggtree(tree, aes(color=cut(support, c(1, 0.2, 0, -0.05, -1))), layout="circular", size=1) +
  theme_tree(legend.position=c(0.85, 0.24)) +
  scale_colour_manual(values=c("#2F4F4F", "#98F898", "#FFCC99","#FF6600"),
                      breaks=c("(0.2,1]","(0,0.2]","(-0.05,0]","(-1,-0.05]"),
                      na.translate=T, na.value="gray",
                      guide="legend", name="Quartet Concordance(QC)",
                      labels=expression(QC>0.2, 0 < QC * " <= 0.2", -0.05 < QC * " <= 0", QC <= -0.05))

# (3)color circle points and label each internode with QC/QD/QI
ggtree(tree, size=0.5) +
  geom_tiplab(size=2) + xlim(0, 0.12) +
  geom_nodepoint(aes(subset=!isTip & node != root, fill=cut(support, c(1, 0.2, 0, -0.05, -1))),
                 shape=21, size=4) +
  theme_tree(legend.position=c(0.9, 0.18)) +
  scale_fill_manual(values=c("#2F4F4F", "#98F898", "#FFCC99","#FF6600"),
                    guide="legend", name="Quartet Concordance(QC)",
                    breaks=c("(0.2,1]","(0,0.2]","(-0.05,0]","(-1,-0.05]"),
                    labels=expression(QC>0.2, 0 < QC * " <= 0.2", -0.05 < QC * " <= 0", QC <= -0.05))+
  geom_text(aes(label=score, x=branch), size=2, vjust=-.5)

dev.off()
```

# 3. 案例
- 蜂斗草族的系统文章用到这个方法评估系统发育树的不一致：Out of chaos: Phylogenomics of Asian Sonerileae：https://www.sciencedirect.com/science/article/pii/S1055790322001944#b0290


# 4. references
1. Quartet Sampling软件github：https://github.com/fephyfofum/quartetsampling
2. paper：https://bsapubs.onlinelibrary.wiley.com/doi/full/10.1002/ajb2.1016
3. https://github.com/ShuiyinLIU/QS_visualization

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>