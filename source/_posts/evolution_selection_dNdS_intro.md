---
title: 一文说清K<sub>A</sub>K<sub>S</sub>、d<sub>N</sub>d<sub>S</sub>、D<sub>n</sub>D<sub>s</sub>，以及omega(ω)
date: 2023-10-09
categories: 
- bio
- evolution
- selection
tags:
- bio
- d<sub>N</sub>
- d<sub>S</sub>
- d<sub>N</sub>d<sub>S</sub>
- d<sub>N</sub>/d<sub>S</sub>
- K<sub>A</sub>
- K<sub>S</sub>
- K<sub>A</sub>K<sub>S</sub>
- K<sub>A</sub>/K<sub>S</sub>
- D<sub>n</sub>
- D<sub>s</sub>
- omega(ω)
- omega
- ω
- selection
- positive selection
- negative selection
- purifying selection
- neutral selection
description: 记录了K<sub>A</sub>K<sub>S</sub>、d<sub>N</sub>d<sub>S</sub>、D<sub>n</sub>D<sub>s</sub>，以及omega(ω)的定义和含义，区别和算法，以及ω检测选择压的含义。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=21178392&auto=1&height=32"></iframe></div>

# 1. 背景
1. 木村资生（Masahiko Kimura）为代表的群体遗传学家划时代地提出了中性漂变理论，在分子进化领域得到了越来越多的证据。简单来说，就是在当时掌握的基因序列中，同义突变远多于非同义突变。为了更好地研究中性理论和自然选择在基因进化上所起的作用，非同义突变和同义突变的定量化研究显得十分迫切。
2. 在这一领域，三位来自九州大学的学者率先进行了有益的尝试和探索。接下来，两组来自美国德州大学奥斯汀分校（University of Texas, Houston）的科学家做出了重要的贡献。其中一组包括台湾著名学者李文雄（Wen-Hsiung Li）和吴仲义（Chung-I Wu）描述的K<sub>A</sub>K<sub>S</sub>，另一组是根井正利（Masatoshi Nei）和五条堀孝（Gojobori Takashi）描述的d<sub>N</sub>d<sub>S</sub>。
3. 在这些工作的基础上，发展出使用omega(ω)=d<sub>N</sub>/d<sub>S</sub>(K<sub>A</sub>/K<sub>S</sub>)进行选择压检验的方法。

# 2. 概念
1. **替换(substitution)**
- 进化过程中基因组上一个位点从一个碱基突变成另一个碱基的过程，与点突变的含义相近。
- 在这篇博客中，特指基因编码序列的替换，可能造成或不造成氨基酸产物的变化。
2. **同义替换(synonymous substitution)**
- 基因编码序列中一个位点的替换，没有造成氨基酸产物的变化，这种替换被称为同义替换。
3. **非同义替换(non-synonymous substitution)**
- 基因编码序列中一个位点的替换，造成氨基酸产物的变化，这种替换被称为非同义替换。
4. **同义替换率(synonymous substitutions rate, d<sub>S</sub>或K<sub>S</sub>)**
- 定义：每同义位点的同义碱基替代数 (the number of synonymous substitutions per synonymous site)，用d<sub>S</sub>或K<sub>S</sub>表示。
- 通常是通过对两条蛋白编码序列进行比较，用发生同义突变的位点数目比上可能发生同义突变的所有位点总数，比值用来估计同义替换率。
5. **非同义替换率(non-synonymous substitutions rate, d<sub>N</sub>或K<sub>A</sub>)**
- 定义：每非同义位点的非同义碱基替代数 (the number of non-synonymous substitutions per non-synonymous site)，用d<sub>N</sub>或K<sub>A</sub>表示。
- 通常是通过对两条蛋白编码序列进行比较，用发生非同义突变的位点数目比上可能发生非同义突变的所有位点总数，比值用来估计非同义替换率。
6. **omega(ω) 值(d<sub>N</sub>/d<sub>S</sub>或K<sub>A</sub>/K<sub>S</sub>比值的估计)**
- 同义替换率与非同义替换率的比值，能够反映基因的选择压力。
- 通常是通过对两条基因编码序列进行比较，用计算得到的同义替换率比上非同义替换率，得到d<sub>N</sub>/d<sub>S</sub>或K<sub>A</sub>/K<sub>S</sub>的估值ω 值。
7. **同义替换数(D<sub>s</sub>)**
- 与d<sub>S</sub>不同，代表的是发生了同义替换的碱基数目。
8. **非同义替换数(D<sub>n</sub>)**
- 与d<sub>N</sub>不同，代表的是发生了非同义替换的碱基数目。

# 3. K<sub>A</sub>和K<sub>S</sub>、d<sub>N</sub>和d<sub>S</sub>，以及D<sub>n</sub>和D<sub>s</sub>之间的区别
1. d<sub>N</sub>和d<sub>S</sub>与K<sub>A</sub>和K<sub>S</sub>含义是一样的，可相互替代使用
- K<sub>A</sub>和K<sub>S</sub>与d<sub>N</sub>和d<sub>S</sub>是两组科学家对每个位点的非同义和同义替换数目的不同的描述：李文雄和吴仲义使用的是K<sub>A</sub>和K<sub>S</sub>，而根井正利和五条堀孝则将同样的概念命名为d<sub>N</sub>和d<sub>S</sub>。
2. d<sub>N</sub>和d<sub>S</sub>(K<sub>A</sub>和K<sub>S</sub>)与D<sub>n</sub>和D<sub>s</sub>是不同的
- D<sub>n</sub>和D<sub>s</sub>是替换数量的估计，分别代表非同义替换位点和同义替换位点的数目，即计算d<sub>N</sub>和d<sub>S</sub>的分子。
- d<sub>N</sub>和d<sub>S</sub>(K<sub>A</sub>和K<sub>S</sub>)则是替换率的估计，分别代表非同义替换率和同义替换率。

# 4. 计算K<sub>A</sub>和K<sub>S</sub>(d<sub>N</sub>和d<sub>S</sub>)的算法模型
1. 目前计算K<sub>A</sub>和K<sub>S</sub>(d<sub>N</sub>和d<sub>S</sub>)通常是使用两条或多条现存类群的基因编码序列，并可以结合系统发育树来共同推断进化过程中发生的碱基替换情况。
2. 计算K<sub>A</sub>和K<sub>S</sub>的算法有非常多，可分为三类：**近似法(Approximate methods)**、**最大似然法(Maximum-likelihood methods)**和**计数法(Counting methods)**。
3. 在使用软件进行具体计算时，常用的包括最大似然法的代表模型YN(Yang and Nielsen,2000)，NG(Nei 1986)等。
4. 然而，除非要比较的序列关系很远（在这种情况下，最大似然法占优势），否则所使用的算法类别对所获得的结果影响很小。只要有足够的数据，三种方法都会得出相同的结果。更重要的是所选方法中隐含的假设。

## 4.1. 近似法(Approximate methods)
1. 近似法的三个基本步骤
- 计算两个序列中同义和非同义位点的数量，或通过序列长度乘以每类替换的比例来估算这一数量；
- 计算同义和非同义替换的数量；
- 纠正多次替换(multiple substitutions)的情况。
2. 近似法的特点
- 三个基本步骤，尤其是后两步，需要做出简单的假设，才能计算；
- 由于最后考虑（第三步），不可能精确确定多次替换的数量。

## 4.2. 最大似然法(Maximum-likelihood methods)
1. 最大似然法特点
- 最大似然法利用概率论同时完成近似法的所有三个步骤。
- 它通过推导出产生输入数据的最可能值来估算关键参数，包括序列之间的分歧度(divergence)和转换/颠换比率(transition/transversion ratio)。

## 4.3. 计数法(Counting methods)
- 为了量化替换的数量，我们可以重建祖先序列并记录推断出的位点变化（直接对现存物种序列差异进行计数可能会低估）；
- 将位点的替换率拟合到预先确定的类别中（贝叶斯方法；对于小数据集效果不佳）；为每个密码子生成单独的替换率（计算成本高）。

# 5. d<sub>N</sub>和d<sub>S</sub>(K<sub>A</sub>和K<sub>S</sub>)和 omega(ω) 的计算过程
由于算法的多样性，计算过程也有多种多样。以根井正利所著的经典书目《分子进化与系统发育》中的进化路径法(Evolutionary Pathway Methods)为例，对K<sub>A</sub>和K<sub>S</sub>(d<sub>N</sub>和d<sub>S</sub>)的计算过程加以简单介绍。

## 5.1. 命名变量
同义替换率 d<sub>S</sub>或K<sub>S</sub> = 同义替换数 D<sub>s</sub> / 同义替换位点总数 Ts
非同义替换率 d<sub>N</sub>或K<sub>A</sub> = 非同义替换数 D<sub>n</sub> / 非同义替换位点总数 Tn

## 5.2. 以TTT(Phe)转变为GTA(Val)为例
1. 先考虑TTT的序列替换的所有可能：
- 查看密码子表可知，TTT的前两个位点的任意变化，都将导致氨基酸的变化，而第三个位点的突变，有2/3的可能性会导致氨基酸变化（只有TTT变为TTC都是编码苯丙氨酸Phe）。
- 所以，对于TTT，非同义替换位点总数 Tn = 2 + 2/3 = 8/3，而同义替换位点总数 Ts = 0 + 1/3 = 1/3。
2. 再比较TTT到GTA变化的简约路径，共有两条：
- TTT(Phe)-GTT(Val)-GTA(Val)
- TTT(Phe)-TTA(Leu)-GTA(Val)
3. 假设以上两条路径概率相等，那么非同义替换数 D<sub>n</sub> = (1+2)/2 = 1.5，同义替换数 D<sub>s</sub> = (1+0)/2 = 0.5。
4. 于是，非同义替换率 d<sub>N</sub> = 非同义替换数 D<sub>n</sub> / 非同义替换位点总数 Tn = 1.5/(8/3) = 0.56，同义替换率 d<sub>S</sub> = 同义替换数 D<sub>s</sub> / 同义替换位点总数 Ts =  = 0.5/(1/3) = 1.5。
5. 最后 omega(ω) = d<sub>N</sub>/d<sub>S</sub> = 0.56/1.5 = 0.37。

## 5.3. 实际计算
以上是使用其中一种算法计算 omega(ω) 的过程，实际的计算过程常常借助开发的整合了相应算法的程序，比如PAML的codeML，KaKs_Calculator等，在这些程序中计算 omega(ω) 是通过对d<sub>N</sub>/d<sub>S</sub>的估计进行计算的。

# 6. omega(ω) (d<sub>N</sub>/d<sub>S</sub>或K<sub>A</sub>/K<sub>S</sub>的估值) 用于检测选择压
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
- 可以使用更复杂的似然法来分析最大似然分析的结果，通过进行卡方检验来区分空模型（Ka/Ks = 1）和观测结果。
6. omega(ω) 结果解读
- 由于绝大多数基因 ω < 1，代表受到正选择的基因(ω > 1)是很少见的。
- 结合系统发育树，通常可以分析 ω 在进化树上的变化，如果树上的 ω 都小于1，但某一支比其他所有支的 ω 都要小，这也说明这一支受到的负选择在放松。

# 7. reference
1. wiki-Ka/Ks ratio: https://en.wikipedia.org/wiki/Ka/Ks_ratio
2. KaKs和dNdS的区别和算法：https://ibook.antpedia.com/x/147335.html

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>