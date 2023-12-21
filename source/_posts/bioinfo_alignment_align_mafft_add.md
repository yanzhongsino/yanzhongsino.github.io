---
title: 用MAFFT在现有多序列比对中添加新序列
date: 2023-11-16
categories: 
- bioinfo
- alignment
- align
tags: 
- MAFFT
- alignment
- Multiple Sequence Alignment
- MSA
- sequence alignment
- add
- merge
description: 用MAFFT将新序列添加到已经比对好的alignment上(Add new sequences to an existing alignment using MAFFT)，保持已比对序列的比对，对新序列进行比对。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=537596136&auto=1&height=32"></iframe></div>


# 1. MAFFT简介
MAFFT是基于渐进式比对的算法，用于进行多序列比对(Multiple Sequence Alignment,MSA)。由于比对速度快，常用于组学数据的比对。

MAFFT网页版有一个功能（--add），即向现有的多序列比对中添加新的序列，这在新增数据时非常有用。

# 2. MAFFT的--add功能
MAFFT的--add功能的网页：https://mafft.cbrc.jp/alignment/server/add.html

## 2.1. --add功能的三个模块
根据新增序列和现有多序列比对(MSA)的相对长度，分别选用--add功能的三个模块：
1. --add：align full length sequences to an MSA. 新增序列与现有MSA长度相当时，选--add模块。
2. --addfragments：align fragment sequences to an MSA. 新增序列短于现有MSA时，选--addfragments模块。
3. --addlong：align long sequences to a short MSA. 新增序列长于现有MSA时，选--addlong模块。

# 3. MAFFT网页版的--add用法
在网页https://mafft.cbrc.jp/alignment/server/add.html上选择模块，然后上传现有MSA和新增序列，选择参数，即可获得比对结果。

## 3.1. 基本参数
1. Allow unusual symbols：是否允许不明确字符。
2. UPPERCASE/lowercase：输出的比对文件的字母大小写是否与输入文件一致。
3. Direction of nucleotide sequences：是否根据第一条序列调整剩余序列的方向。
4. Output order：输出比对文件的序列顺序。
5. Sequence title：输出比对文件的序列名称是否与输入文件一致。
## 3.2. 高级参数
最常用的是第一个参数，是否要保持输出MSA的长度。
1. **Keep alignment length**：是否保持输出的比对的长度与输入的现有MSA长度一致，如果选择这个选项，则会把新增序列的insertions删除以保持长度不变。
2. Strategy：默认选择Auto
3. Progressive methods：包括FFT-NS-2,G-INS-1,L-INS-1三个选项。
4. Iterative refinement methods：包括FFT-NS-i,G-INS-i,L-INS-i三个选项。
5. 对核苷酸/氨基酸序列的分数矩阵和gap处理等参数。

## 3.3. 结果
提交之后，在网页中可以看到比对结果，并可以下载fasta/clustal格式的比对结果，也可以转化成phylip/nexus/msf/gcg等格式。

还可以点击**Refine dataset**进入编辑界面，对比对结果进行进一步的编辑，比如对指定序列进行反向互补操作，保留或删除指定序列。

# 4. MAFFT命令行版的--add用法
命令行版也可以使用--add功能，具体参数参考：https://mafft.cbrc.jp/alignment/software/addsequences.html

# 5. MAFFT的merge功能
MAFFT还有一个merge模块，网页版：https://mafft.cbrc.jp/alignment/server/merge.html

merge模块用于合并两个或更多已比对好的sub MSAs。当假定这些sub MSAs在系统发育关系上是各自独立为一单系时，即系统树上各自不重叠时，可使用merge模块。如果系统发育关系不独立，则推荐使用--add模块。

# 6. references
1. https://mafft.cbrc.jp/alignment/server/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>