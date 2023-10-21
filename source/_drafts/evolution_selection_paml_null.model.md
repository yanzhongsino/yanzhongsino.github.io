---
title: 用PAML的codeML进行选择分析：（二）计算平均选择压 (平均 ω 值 )
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
- branch model
description: 使用PAML的codeML程序包中的空模型null model来计算系统发育树上的平均 ω 值。
---

<div align="middle"></div>

# 1. 背景知识
1. d<sub>N</sub>d<sub>S</sub>(K<sub>A</sub>K<sub>S</sub>)，以及 omega(ω) 的定义，以及 ω 用于检测选择压可以参考博客**一文说清K<sub>A</sub>K<sub>S</sub>、d<sub>N</sub>d<sub>S</sub>、D<sub>n</sub>D<sub>s</sub>，以及ω**：https://yanzhongsino.github.io/2023/10/09/evolution_selection_dNdS_intro
2. 选择分析的概况、PAML的安装和选择分析的常用模型可以参考博客**用PAML的codeML进行选择分析：（一）概况简介**：https://yanzhongsino.github.io/2023/10/10/evolution_selection_paml_intro

# 2. 分支模型 (branch model) 选择分析的目的
1. 绝大多数情况下 ω < 1，因此选择分析通常是为了检测到较少见的正选择 (ω > 1)
2. 结合系统发育树，分析 ω 在进化树上的变化。两种结果都是有意义的：
- 如果某一支（前景支）上的 ω > 1，而其他所有支（背景支）上的 ω < 1，并且似然率检验(LRT)的显著性分析支持备择假设（正选择），那么可认为前景支受到了**正选择**。
- 虽然进化树上的 ω 都小于1，但某一支比其他所有支的 ω 都要小，这也说明这一支受到**负选择的放松**。

# 3. 使用PAML的codeML中的分支模型 (branch model) 来检测分支的选择方向和强度
## 3.1. 假设
1. 零假设 (null hypothesis)：所有分支都有相同的 ω 值
2. 备择假设 (alternative hypothesis)：一些分支具有与背景分支不同的 ω 值

## 3.2. 准备输入文件
输入的序列比对文件和树文件的格式可以参考https://github.com/abacus-gene/paml-tutorial/tree/main/positive-selection/00_data 中的 Mx_aln.phy Mx_root.tree，Mx_unroot.tre。
1. 序列比对文件 (cds_aln.phy)
- phylip格式
- 比对好的编码序列 (cds) 文件，碱基数量是3的倍数（codon模式比对）
- 建议删除gaps和难以align的区域，但要按密码子删除（删除和保留都是3的倍数个碱基）
2. 树文件 (tree.newick)
- newick格式，并在首行前加上一行，包括两个数字，一个是物种/样本数量，一个是树的数量（一般是1），空格隔开。
- 只需要拓扑结构，删除枝长、支持率等信息。
3. 配置文件 (codeml.ctl)
- codeml.ctl 的例子在安装的程序包paml/examples/下有，可以复制到分析目录下，修改后使用
- 基础的配置参数如下，后续分析可基于这套参数修改：

```
      seqfile = cds_aln.phy
     treefile = tree.newick
      outfile = out.txt

        noisy = 9
      verbose = 1
      runmode = 0

      seqtype = 1
    CodonFreq = 2
*        ndata = 10
        clock = 0
       aaDist = 0
   aaRatefile = ../dat/jones.dat

        model = 2
      NSsites = 0

        icode = 0
        Mgene = 0

    fix_kappa = 0
        kappa = 2
    fix_omega = 0
        omega = .4 

    fix_alpha = 1  
        alpha = 0. 
       Malpha = 0
        ncatG = 8

        getSE = 0
 RateAncestor = 1

   Small_Diff = .5e-6
    cleandata = 0
```

4. 配置参数的含义和选择

```
      seqfile = cds_aln.phy * sequence data filename
     treefile = tree.newick      * tree structure file name
      outfile = out.txt           * main result file name

        noisy = 9  * 0,1,2,3,9: how much rubbish on the screen；控制屏幕输出的信息量
      verbose = 1  * 0: concise; 1: detailed, 2: too much；控制输出文件中的信息量
      runmode = 0  * 0: user tree;  1: semi-automatic;  2: automatic
                   * 3: StepwiseAddition; (4,5):PerturbationNNI; -2: pairwise ; 代表使用用户提供的树拓扑(0)，或者codeML根据序列自行计算(2)

      seqtype = 2  * 1:codons; 2:AAs; 3:codons-->AAs ；输入序列数据类型
    CodonFreq = 2  * 0:1/61 each, 1:F1X4, 2:F3X4, 3:codon table；密码子频率（CodonFreq）可以相等（0：每个1/61）或者不同，以解释密码子使用偏差。默认是2：F3X4，会根据序列决定密码子频率。FmutSel模型(CodonFreq = 7)为通用遗传密码的每个密码子分配了60(= 61-1)个密码子适应度参数(Yang and Nielsen 2008)。FmutSel0模型(CodonFreq = 6)是FmutSel的特例，它对同义密码子赋予相同的适应度值，因此只使用19(= 20-1)个氨基酸适应度参数。该模型假设氨基酸频率由蛋白质的功能需求决定，但同义密码子的相对频率仅由突变偏倚参数决定。
*        ndata = 10 * 基因或比对的数量
        clock = 0  * 0:no clock, 1:clock; 2:local clock; 3:CombinedAnalysis；指定分子钟，不假设分子钟(0)，严格分子钟(1)，本地分子钟(2)，和联合分析(3)。这里选0不假设分子钟，则需要使用无根树（大部分情况是这样）。
       aaDist = 0  * 0:equal, +:geometric; -:linear, 1-6:G1974,Miyata,c,p,v,a
   aaRatefile = ../dat/jones.dat  * only used for aa seqs with model=empirical(_F)
                   * dayhoff.dat, jones.dat, wag.dat, mtmam.dat, or your own

        model = 2 * 根据进行的分析和假设来选择
                   * models for codons:
                       * 0:one, 1:b, 2:2 or more dN/dS ratios for branches
                   * models for AAs or codon-translated AAs:
                       * 0:poisson, 1:proportional, 2:Empirical, 3:Empirical+F
                       * 6:FromCodon, 7:AAClasses, 8:REVaa_0, 9:REVaa(nr=189)

      NSsites = 0  * 0:one w;1:neutral;2:selection; 3:discrete;4:freqs;
                   * 5:gamma;6:2gamma;7:beta;8:beta&w;9:beta&gamma;
                   * 10:beta&gamma+1; 11:beta&normal>1; 12:0&2normal>1;
                   * 13:3normal>0

        icode = 0  * 0:universal code; 1:mammalian mt; 2-10:see below ； 编码方式
        Mgene = 0
                   * codon: 0:rates, 1:separate; 2:diff pi, 3:diff kapa, 4:all diff
                   * AA: 0:rates, 1:separate

    fix_kappa = 0  * 1: kappa fixed, 0: kappa to be estimated ；0代表从给的序列推断kappa，kappa是转换/颠换的比率。
        kappa = 2  * initial or fixed kappa
    fix_omega = 0  * 1: omega or omega_1 fixed, 0: estimate ；0代表计算omega
        omega = .4 * initial or fixed omega, for codons or codon-based AAs ；当fix_omega = 0 ,我们需要计算omega时，这个初始omega值便随意设置都ok。

    fix_alpha = 1  * 0: estimate gamma shape parameter; 1: fix it at alpha
        alpha = 0. * initial or fixed alpha, 0:infinity (constant rate)
       Malpha = 0  * different alphas for genes
        ncatG = 8  * # of categories in dG of NSsites models

        getSE = 0  * 0: don't want them, 1: want S.E.s of estimates
 RateAncestor = 1  * (0,1,2): rates (alpha>0) or ancestral states (1 or 2)

   Small_Diff = .5e-6
    cleandata = 1  * remove sites with ambiguity data (1:yes, 0:no)? 歧义数据和比对间隙的位点要保留(0)或删除(1)，建议自行删除歧义和gaps数据，并在这里选保留(0)。
*  fix_blength = 1  * 0: ignore, -1: random, 1: initial, 2: fixed, 3: proportional
       method = 0  * Optimization method 0: simultaneous; 1: one branch a time

* Genetic codes: 0:universal, 1:mammalian mt., 2:yeast mt., 3:mold mt.,
* 4: invertebrate mt., 5: ciliate nuclear, 6: echinoderm mt., 
* 7: euplotid mt., 8: alternative yeast nu. 9: ascidian mt., 
* 10: blepharisma nu.
* These codes correspond to transl_table 1 to 11 of GENEBANK.
```


## 3.3. 执行codeML
1. 零假设模型(null model)的运算
- 在基础的配置参数上修改以下参数，保存为codeml_null.ctl

```
        model = 0 * model = 0 代表零假设（所有分支的 ω 值都一样）
      NSsites = 0
     treefile = tree.newick
      outfile = out_null.txt      
```

- 执行命令`codeml codeml_null.ctl`，得到结果文件out_null.txt

2. 备择假设模型(branch model)的运算
- 标记前景分支：在树文件tree.newick的基础上标记前景分支，保存为tree_branch.newick文件。
- 标记方法：标记某一支为前景支（#1），标记某一支及所有子分支都为前景支（$1）。如只标记A和B的祖先枝为前景支，则可用`((((A,B)$1,C),D),E);`；标记A和B祖先支以及A、B分支都为前景支，用`((((A,B)$1,C),D),E);`。
- 在基础的配置参数上修改以下参数，保存为codeml_branch.ctl

```
        model = 2 * model = 2 代表备择假设（不同分支的 ω 值有2种或以上的类别）
      NSsites = 0
     treefile = tree_branch.newick
      outfile = out_branch.txt
```

- 执行命令`codeml codeml_branch.ctl`，得到结果文件out_branch.txt

## 结果文件
屏幕输出应该是不言自明的，其中包括有关分析、对齐和ML迭代进度的信息。如果输入文件中有错误，或者程序由于其他原因而失败，屏幕上也可能会打印错误消息。分析完成后，还将在输出文本文件(out_m .txt)中打印有关数据和结果的一些信息。该输出文件分为五个部分，如下所述。

out_m .txt_9、输入对齐中的位点模式概要
        输入对齐及其压缩版本打印在输出文件的顶部，其中每个位点模式只表示一次，相应的频率(模式计数)在下面的块中显示。例如，在这里使用的基因比对文件(Mx_aln.phy)中，大多数位点模式是唯一的，频率为1，而两个位点模式重复了2次和4次，如下所示。如果使用cleandata = 1，则将打印删除间隙和模糊位置之前和之后的对齐，然后是修剪对齐的位点模式计数。请注意，如果删除有间隔的列，则位点将重新编号，这可能会影响位点或分支位点模型下的输出。

out_m .txt_10、输入对齐和所选模型的摘要
        接下来，用于分析的CODEML版本连同对齐文件的名称一起打印出来。然后是对照文件中指定的模型的详细信息，比对文件中物种数(ns = 12)和密码子数(ls = 663，碱基对数[1989]分为3):

out_m .txt_11、核苷酸和密码子频率综述
接下来，每个序列中观察到的核苷酸和密码子频率以及它们在所有序列中的平均值被打印在输出文件中，然后是模型下的密码子频率，可以使用evolver等软件用于模拟目的(Yang 2007)。

out_m .txt_12、树得分和估计模型参数值的总结
        对数似然值、自由参数个数、估计分支长度以及模型下的其他参数(本例中为M0)如下面的前四行所示。自由参数包括分支长度、平衡频率、替换/颠换速率比(κ)和ω分布中的参数。注意,无根树与n个分支：2×n−3分支长度(即2×12−3 = 21我们的树的分支长度n = 12个分类群)和3 mutation-bias参数(四个频率之和为1)。它可能很难匹配的ml 4号线到相应的分支系统学的长度或模型参数,但Newick树中的分支长度再次打印。最后一块显示了κ(替换/颠换率比)和ω的估计，以及突变偏差(核苷酸频率)参数。突变选择模型通过模拟突变偏差和选择下突变的固定来适应不同的密码子频率:较高的突变偏差参数，例如π * A，意味着突变过程偏向于A (Yang和Nielsen 2008，等)。1和4)。下面是每个分支的t(分支长度)、dN(非同义速率)和dS(同义速率)的估计值。输出如下所示:

        由于指定了同质模型(M0) (model = 0和NSsites = 0)，因此在dN/dS列下的每个分支报告相同的ω值(0.3357)(见上面的粗体值)。非同义位点的树长dN = 1.1386，同义位点的树长dS = 3.3921， ω = dN/dS = 0.3357，表明黏液病毒基因总体上处于纯化选择（ ω<1）阶段，非同义突变进入固定的几率(ω = dN/dS = 0.3357)平均为同义突变的三分之一。

## 3.4. 比较结果
### 3.4.1. 似然率检验 (Likelihood Ratio Test,LRT)
如果零假设被数据支持，那么两种假设的似然值差异不会超过抽样误差。LRT检验的是，两种假设的似然值的比值与1是否有显著差异，或者似然值的自然对数的差值与0是否有显著差异。

在两次运算的结果文件`out_null.txt`和`out_branch.txt`中分别寻找LRT的结果lnL值：
1. 零假设模型的似然值的对数：`lnL(ntime:14  np:16): -1296.438022  +0.000000`
2. 备择假设模型的似然值的对数：`lnL(ntime:14  np:17): -1292.210434  +0.000000`
- np代表参数的数量number of parameters

### 3.4.2. 用卡方检验(chi^2)计算p值，分析似然值差异的显著性
lnL1-lnL0 满足卡方分布(chi^2)：；计算p值
1. ▲LRT= abs (2 × (lnL1 - lnL0)) = abs (2 × (-1292.210434+1296.438022)) = 8.455176
2. 自由度df = np1 - np0 = 17 - 16 = 1
3. 卡方分布的显著性p值计算：在linux系统中之间使用命令`chi2 1 8.455176`来计算p值
- 在屏幕得到结果`df =  1  prob = 0.003640060 = 3.640e-03`
- p=0.00364,远小于0.01。可以拒绝零假设，接收备择假设。即前景分支的 ω 值对系统发育树的平均 ω 值来说有显著差异。

### 3.4.3. omega(ω) 值
在两次运算的结果文件`out_null.txt`和`out_branch.txt`中分别寻找omega(ω) 值的结果：
1. 零假设模型的 ω 值：`omega (dN/dS) = 0.12795`
- 代表零假设下，整个树上的平均 ω 值为0.12795
2. 备择假设模型的 ω 值：`ω (dN/dS) for branches: 0.08179 0.21097`
- 代表备择假设下，背景支的平均 ω 值为 0.08179 ，前景支的平均 ω 值为 0.21097。
3. 由于p值<0.01，接受备择假设，代表前景支的 ω 值明显比背景支大，由于前景支和背景支的 ω 值都<1。所以前景支应该经历了负选择的放松。


# 4. reference
1. PAML User Guide：https://github.com/abacus-gene/paml/blob/master/doc/pamlDOC.pdf
2. PAML 中文用户手册：https://blog.sciencenet.cn/blog-3433349-1241310.html
3. PAML开发者网站：http://abacus.gene.ucl.ac.uk/software/#phylogenetic-analysis-by-maximum-likelihood-paml
4. 陈连福博客-基因正选择分析原理：http://www.chenlianfu.com/?p=3084%E5%92%8Chttp://blog.sciencenet.cn/blog-460481-1163040.html
5. 视频-使用codeml的分支模型（branch test）检测进化树上某一支的选择压力：https://www.bilibili.com/video/av10469605/?from=search&seid=8859495264060497812&vd_source=86d4997d664e6bb659b1d0f0dcd15267
6. wiki-Ka/Ks ratio：https://en.wikipedia.org/wiki/Ka/Ks_ratio
7. paml检测正选择的初学者指南：https://wap.sciencenet.cn/blog-3434047-1389140.html?mobile=1

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>