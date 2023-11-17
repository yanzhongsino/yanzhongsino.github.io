---
title: 祖先状态重建：用PAML的baseML重建祖先序列
date: 2023-11-03
categories: 
- bio
- evolution
- ancestral state reconstruction
tags:
- bioinfo
- biosoft
- PAML
- baseML
- 祖先状态重建
description: 使用PAML的baseML程序根据系统发育关系从现存物种的序列重建祖先序列。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=195874&auto=1&height=32"></iframe></div>


# 1. 重建祖先序列
- 根据现存物种的状态（性状/序列），以及系统发育关系，可以推断祖先节点物种的状态（性状/序列），即祖先状态重建。
- 这篇博客介绍使用PAML的baseML程序根据系统发育关系从现存物种的序列重建祖先序列的用法。

# 2. 用PAML的baseML程序重建祖先序列
## 2.1. 准备输入文件
输入的序列比对文件和树文件的格式可以参考https://github.com/abacus-gene/paml-tutorial/tree/main/positive-selection/00_data 中的 Mx_aln.phy Mx_root.tree，Mx_unroot.tre 文件。
1. 序列比对文件 (cds_aln.phy)
- phylip格式
- 比对好的编码序列 (cds) 文件，碱基数量是3的倍数（codon模式比对）
- 建议删除gaps和难以align的区域，但要按密码子删除（删除和保留都是3的倍数个碱基）
2. 树文件 (tree.newick)
- newick格式，并在首行前加上一行，包括两个数字，一个是物种/样本数量，一个是树的数量（一般是1），空格隔开。
- 只需要拓扑结构，删除枝长、支持率等信息。
3. 配置文件 (baseml.ctl)
- baseml.ctl 的例子在安装的程序包paml/examples/下有，可以复制到分析目录下，修改后使用
- 基础的配置参数如下，后续分析可基于这套参数修改：

```
seqfile = aln.phy  * sequence data file name
treefile = tree.newick * tree structure file name
outfile = out.txt * main result file
noisy = 3     * 0,1,2,3: how much rubbish on the screen 控制屏幕输出的信息量
verbose = 0     * 1: detailed output, 0: concise output * 控制输出文件中的信息量
runmode = 0     * 0: user tree; 1: semi-automatic; 2: automatic * 3: StepwiseAddition; (4,5):PerturbationNNI
model = 6     * 0:JC69, 1:K80, 2:F81, 3:F84, 4:HKY85, 5:T92, 6:TN93, 7:REV, 8:UNREST, 9:REVu; 10:UNRESTu 指定核酸替代模型。模型0,1,…,8分别表示模型JC69， K80， F81， F84， HKY85， T92， TN93， REV (也称为 GTR)， 和 UNREST。注释参看Yang（1994 JME 39:105-111）。两个模型可以分别运行。model = 9是REV模型的特例，而model = 10是无限制模型的特例。当model = 9或10时，同一行里包含的主要是指定模型的额外信息。中括号中的数字是自由参数比值的数字。例如，REV应该是5而UNREST应该是11。接在这些数字后面的是圆括号中的数字。输出文件中的比值参数将按照这里的顺序摆列。括号中没有提到的比值为1。当model = 9时，你只能指定TC或CT，但不能同时指定。当model = 10时，TC和CT是不同的。
Mgene = 0      * 0:rates, 1:separate; 2:diff pi, 3:diff kapa, 4:all diff
ndata = 1      * number of data sets * 基因或比对的数量
clock = 0      * 0:no clock, 1:clock; 2:local clock; 3:CombinedAnalysis
fix_kappa = 0      * 0: estimate kappa; 1: fix kappa at value below
kappa = 2.5      * initial or fixed kappa
fix_alpha = 1      * 0: estimate alpha; 1: fix alpha at value below
alpha = 0.      * initial or fixed alpha, 0:infinity (constant rate)
Malpha = 0      * 1: different alpha¡¯s for genes, 0: one alpha
ncatG = 5      * # of categories in the dG, AdG, or nparK models of rates
fix_rho = 1      * 0: estimate rho; 1: fix rho at value below
rho = 0.      * initial or fixed rho, 0:no correlation
nparK = 0      * rate-class models. 1:rK, 2:rK&fK, 3:rK&MK(1/K), 4:rK&MK
nhomo = 0      * 0 & 1: homogeneous, 2: kappa for branches, 3: N1, 4: N2。 nhomo仅针对baseml，并涉及同一替代模型中的频率参数。nhomo = 1适合一个均匀模型，但是通过最大似然迭代估算频率参数（πT，πC和πA；πG在频率总和为1中不是一个自由参数。）。这适用于F81，F84，HKY85，T92（这种情况下πGC是一个参数），TN93，或REV模型。通常（nhomo = 0）这些通过观察到的频率的平均值来估算。在两种情况下（nhomo = 0 和 1），你需要为碱基频率计算三个（或对T92计算一个）自由参数。 nhomo = 2 适用K80，F84和HKY85模型对树中的每个分枝估算一个转换/颠换比率（k）。 选项 nhomo = 3，4，和5仅用于F84，HKY85 或 T92。
getSE = 0      * 0: don¡¯t want them, 1: want S.E.s of estimates
RateAncestor = 1      * (0,1,2): rates (alpha>0) or ancestral states，默认是0，不重构祖先状态，1是重构祖先序列。
Small_Diff = 1e-6
cleandata = 0      * remove sites with ambiguity data (1:yes, 0:no)?  歧义数据和比对间隙的位点要保留(0)或删除(1)，建议自行删除歧义和gaps数据，并在这里选保留(0)。
* icode = 0      * (RateAncestor=1 for coding genes, ¡°GC¡± in data)
* fix_blength = 0      * 0: ignore, -1: random, 1: initial, 2: fixed。设置程序处理输入树文件中枝长数据的方法：0，忽略输入树文件中的枝长信息；-1，使用一个随机起始进行进行计算；1，以输入的枝长信息作为初始值进行ML迭代分析，此时需要注意输入的枝长信息是碱基替换率，而不是碱基替换数；2，不需要使用ML迭代计算枝长，直接使用输入的枝长信息，需要注意，若枝长信息和序列信息不吻合可能导致程序崩溃；3，让ML计算出的枝长和输入的枝长呈正比。
method = 0      * 0: simultaneous; 1: one branch at a time
```

## 2.2. 执行
- 执行命令`baseml baseml.ctl`

## 2.3. 结果文件
推断的祖先序列在结果文件rst中，与提供的比对好的现存物种序列长度一致，可免比对，直接提取使用。
### 2.3.1. out.txt
- 主要结果文件，默认名称是mlc。
- 内容包括每条序列的A/T/C/G的频率，一致性位点数量，成对遗传距离矩阵，Pairwise deletion matrix，带枝长的树，树节点和枝的编号，以及kappa值。
### 2.3.2. rst：包含三块内容
Marginal reconstruction和Joint reconstruction是两种祖先序列重构的算法。如果对所有分类单元的合集都感兴趣，用joint；如果对特定分类单元的祖先感兴趣，用marginal。
1. 运行的参数以及树的信息
- 给系统发育树的每个node进行编号的信息，以及每个branch是哪两个node连接而成；
2. Marginal reconstruction of ancestral sequences的结果
- Prob of best state at each node, listed by site. 分位点记录的每个node的序列和频率，以及重建的祖先序列的概率；
- Summary of changes along branches. 分branch记录突变的位点；
- List of extant and reconstructed sequences（现存node和祖先node的序列）；
- Overall accuracy of the 91 ancestral sequences
- Counts of changes at sites. 分位点记录突变的计数；
3. Joint reconstruction of ancestral sequences的结果
- Reconstruction (prob.), listed by pattern
- List of extant and reconstructed sequences（现存node和祖先node的序列）；
### 2.3.3. 其他文件
另外还有一些其他结果文件，暂时还未用到。
1. rst1
2. lnf
3. rub

# 3. reference
1. PAML User Guide：https://github.com/abacus-gene/paml/blob/master/doc/pamlDOC.pdf
2. PAML 中文用户手册：https://blog.sciencenet.cn/blog-3433349-1241310.html
3. PAML开发者网站：http://abacus.gene.ucl.ac.uk/software/#phylogenetic-analysis-by-maximum-likelihood-paml
4. paml软件进行祖先酶序列重建极简教程视频：https://www.bilibili.com/video/BV1AX4y1K7H4/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>