---
title: 用MEGA统计序列比对的成对突变频谱
date: 2023-11-11
categories: 
- bioinfo
- alignment
- statistics
tags: 
- Molecular Evolutionary Genetics Analysis
- MEGA
- Multiple Sequence Alignment
- MSA
- sequence alignment
- nucleotide pair frequencies
- Ti
- Tv
description: 本文介绍用MEGA软件计算比对好的核苷酸序列的两两成对的突变频谱的方法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5056815&auto=1&height=32"></iframe></div>

# 1. 序列比对和MEGA的简单介绍
- 序列比对的知识可以参考另一篇博文的内容。Sequence Alignment：https://yanzhongsino.github.io/2021/09/06/bioinfo_alignment_align_intro/
- MEGA软件的全称是Molecular Evolutionary Genetics Analysis，是用于对序列进行操作、比对、统计、系统发育和进化分析的一个软件包，拥有非常多的功能。
- MEGA官方网站是：https://www.megasoftware.net/，现在最新版本是MEGA 11。

# 2. 成对突变频谱
突变频谱是指序列间突变的类型和总数的统计。

1. 两条比对好的序列的碱基是一一对应的，序列的差异被认为是点突变的证据。如果考虑突变方向，点突变共有12种类型；如果不考虑突变方向，点突变共有6种类型。
2. 在两条序列间，对每种类型的点突变的总数进行统计，还可以根据突变类型计算Ti，Tv以及Ti/Tv ratio (Ts/Tv ratio)。
3. 在多序列比对中，也可以计算两两的成对序列的点突变的类型和数量，即成对突变频谱。

# 3. 用MEGA统计成对突变频率
## 3.1. 准备文件
1. 需要的文件是已经比对好的多序列或两条序列的文件，fasta格式或meg格式都可用。
## 3.2. 统计的操作
### 3.2.1. 在MEGA导入多序列比对文件
1. 在MEGA中打开fasta或meg格式的比对文件；
2. 弹出对话框【How would you like to open this fasta file?】，选【Analyze】；
3. 弹出对话框【Input Data Options】，选对应的数据类型，DNA序列选【Nucleotide Sequences】；
4. 弹出对话框【Confirmation-Protein-coding nucleotide sequence data】，如果是蛋白编码序列coding sequences就选【Yes】，否则选【No】；
### 3.2.2. 选择需要统计的序列
1. 此时打开了比对文件，点击MEGA界面操作栏下面的第一个小方块（显示TA，右边第二个小方块是Close Data），打开界面【Sequence Data Explorer】；此界面可以对导入的比对序列进行统计分析。
2. 界面【Sequence Data Explorer】中包含了导入的多序列比对文件的序列信息，默认选择所有序列（所有序列前的勾是勾上的），可以根据需要选择至少两条序列进行突变频谱的统计。如果选择了三条及以上序列，则突变频谱的统计结果是两两序列突变数量的平均值。
3. 在界面【Sequence Data Explorer】选择【Statistics】-【Nucleotide Pair Frequencies】-【Directional (16 Pairs)..】或【Undirectional (10 Pairs)..】。
- 【Directional】代表进行有方向的突变频谱的统计，界面【Sequence Data Explorer】中勾选的序列的顺序决定了方向，代表从前面的序列到后面的序列的突变频谱。可以在选择序列时更改序列的顺序，来计算相应方向的突变频谱。
- 【Undirectional (10 Pairs)..】则代表进行无方向的突变频谱统计，与有方向的类似，但把A-T和T-A的突变合并，依此类推。
- 有方向的突变频谱包含12种突变类型，无方向的突变频谱包含6种突变类型，这里的统计还包含A-A,T-T,C-C,G-G四种无突变类型的数量，所以分别是16 pairs和10 pairs。
4. 弹出对话框【Select Output Format】，根据需要选择输出文件格式（XLSX,XLS,TXT,CSV,ODS），以及直接展示结果（Display Results）还是保存结果到文件（Save to Disk）。

## 3.3. 结果文件
TXT格式的结果文件示例，这里选择了Liriodendron, Amborella, Nymphaea三条序列进行统计。

```
Data Filename: samples.fa
Data Title: fasta file
Nucleotide Pair Frequencies
Sites Used: All selected
No. of sequences used: 3
Sequences used
  Liriodendron
  Amborella
  Nymphaea
All frequencies are averages (rounded) over all taxa.
ii = Identical Pairs
si = Transitionsal Pairs
sv = Transversional Pairs
R  = si/sv
Domain       ii    si    sv    R     TT    TC    TA    TG    CT    CC    CA    CG    AT    AC    AA    AG    GT    GC    GA    GG Total  Domain Info
   1. Avg 23254   276   573   0.5  7099    62    67    82    86  4567   150    24    62    87  6299    65    80    21    63  5289 24102.3  Data
```

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>