---
title: 鉴定全基因复制事件(WGD)后保留的复制基因
date: 2022-10-18
categories:
- bioinfo
- WGD
tags:
- WGD
- gene duplication event
- OrthoFinder2

description: 此博客记录了鉴定全基因复制事件(WGD)后保留的复制基因的背景、原理和分析步骤。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=28996523&auto=1&height=32"></iframe></div>

# 1. 全基因组复制事件(WGD)
全基因组复制事件(WGD)是指在生物进化历史上发生的全基因组范围的染色体加倍、基因复制的事件，根据目前研究现状，WGD在植物进化历史上是很常见的。

# 2. WGD后保留的复制基因
全基因组复制事件（WGD）后基因的命运有所不同。有些复制的基因对中的一个会发生变异、甚至假基因化，从而使得目前我们观察到的基因组上这类基因是单拷贝的；有些复制的基因对则保留到现在，成为WGD的证据。

# 3. 思路
## 3.1. 参考
通过参考焦远年团队的文章: https://www.sciencedirect.com/science/article/pii/S1674205219303594 的 Methods部分，构建了以下分析思路，用来实现对发生特定WGD事件的物种的基因复制保留分析。

- 实现的目的是：找到某物种基因组上那些在进化历史上发生特定WGD事件后保留至今的基因对，尽可能减少假阳性和假阴性。
- 文章里用了多个软件，目前Orthofinder2可以实现第2，3，4，5步骤。

## 3.2. 步骤
1. 数据
- 多个物种的基因组。最少两个物种，除了发生WGD的被分析物种外，还要包括至少一个不共享此次WGD的外类群。
- 如果要分析多个物种各自的WGDs，也可以一起构建物种树和基因树，在各自筛选基因树做各自WGD保留基因的根系。
2. 基因家族（gene family）同源关系的鉴定
- 用软件OrthoFinder2或者OrthoMCL可以实现基因家族和基因的直系同源关系的推断。
- 推断将会得到所有物种的orthogroups，包括通过考虑每个orthogroup的基因的最近共同祖先来推断orthogroup的拓扑。
- 关于软件OrthoFinder2可以参考博客：https://yanzhongsino.github.io/2021/12/06/bioinfo_orthology.inference_OrthoFinder2/
3. 构建物种树
- 文章用的APG系统的物种树。
- 发现OrthoFinder2可以直接实现物种树构建。
- 或者用单拷贝基因构建，RAxML-NG和IQ-TREE等软件都可。
4. 构建基因树
- 文章里是用的是，通过OrthoMCL鉴定出来的orthogroups，筛除掉少于4个基因和不包含外类群的orthogroups来构建。
- 我发现OrthoFinder2可以直接实现基因树构建，但基因树的筛选需要手动进行。
5. 根据物种树整理基因树的节点
- 通过与物种树的节点比较，对基因树的节点进行整理，保持与物种树一致。
- 文章是用软件Notung2.9来根据物种树的拓扑，融合OrthoMCL的基因树结果。
6. 根据物种树拓扑筛选基因树
- 通过指定一些标准，从OrthoFinder2或者OrthoMCL的结果中筛选符合物种树的基因树，使用这些基因树的基因对作为有发生基因复制事件证据的基因对。
- 文章里的标准是，两个子枝(child branches)需要拥有来自至少一个最近共同祖先的基因，父母节点(parental node)和至少一个子节点（child nodes）有超过50%的支持率。
- 这个也需要手动进行。
7. 移除串联重复（tandem duplications）
- 移除串联重复tandem，这些不是全基因组复制事件中产生。
- 文章里对tandem的鉴定用的标准是两个基因的位置在五个基因以内即做看作tandem。
- 操作上，我是通过基因ID来实现的（如果基因ID的编号相差在5以内就被看作tandem，这种基因树被删除）。注释基因组时对基因ID根据位置进行排序和整理的好处这个时候就体现了。
8. 鉴定基因复制事件的基因
- 通过以上4，5，6，7步骤的筛选，我们获得代表WGD的基因树，这也就意味着得到了WGD后保留的复制基因（基因树上那些基因）。
- 把得到的基因树上的基因根据位置提取出来，分别从两个child branches中各提取一个基因，组成一个基因对。有些基因树上只有一个基因对，有些可能不止两个基因，会产生多于一个基因对。
9. 多次WGDs
- 如果系统发育树的同一枝上有超过一次的WGD事件（比如某物种特有两次WGDs），则通过计算鉴定出来的复制基因对的Ks，通过Ks区分这些WGDs。
- 例如一次WGD的Ks峰值在0.2，另一次在0.7，则可一刀切的把Ks大于两者平均值$$0.45=(0.2+0.7)/2$$的那些基因对算作0.7那次WGD保留的，Ks小于0.45的算作0.2那次保留的。
- 这个是没有其他更好的解决办法的处理方式，在文章中对水稻等发生两次WGDs的物种也是这样处理的。
10. TF分析【可选】
- 得到WGD后保留的基因对后，还可以进行转录因子（transcription factor，TF）的WGD后保留分析。

notes：其中第4，5，6，7步骤都是在对基因树进行筛选，把那些代表了特定WGD的基因树筛选出来，避免其他基因树的干扰（比如tandem）。把代表WGD的基因树筛选出来后，也就意味着得到了WGD后保留的复制基因（基因树上那些基因）。

# 4. 实战
## 4.1. 准备
- 数据准备：物种A有两次物种特有的WGDs，不共享这两次WGD的外类群B。至少这两个物种的基因组数据。有其他物种的也可以一起用。

## 4.2. orthofinder2构建物种树和基因树
### 4.2.1. OrthoFinder2运行
- 注意运行orthofinder2时要指定比对软件，推断基因树的方法，和画树软件。
- 建议参数：`orthofinder -f ./workdirectory -t 24 -a 8 -M msa -S blast -A mafft -T raxml-ng`
- 参考博客：https://yanzhongsino.github.io/2021/12/06/bioinfo_orthology.inference_OrthoFinder2/
- 如果之前跑过orthofinder但没推断基因树和物种树的可以指定步骤重新运行。

### 4.2.2. 结果文件
要用的主要是三个个文件，在/OrthoFinder/Results_xx/Gene_Duplication_Events/目录下的Duplications.tsv和SpeciesTree_Gene_Duplications_0.5_Support.txt，以及/OrthoFinder/Results_xx/Orthogroups下的Orthogroups.txt。

1. Duplications.tsv
- 是一个制表符分隔的文本文件，它列出了通过检查每个正群基因树的每个节点识别出的所有基因复制事件，每行代表至少一次基因复制事件。
- 七列内容分别是：
- 1.Orthogroup的ID
- 2.Species Tree node：发生复制的物种树的分支，参见Species_Tree/SpeciesTree_rooted_node_labels.txt
- 3.Gene tree node：与基因复制事件对应的节点，参见相应的orthogroup Resolved_Gene_Trees/ 中的树；
- 4.Support：存在复制基因的两个副本的预期物种的比例
- 5.Type："Terminal"：物种树终端分支上的重复；"Non-Terminal"：物种树内部分支上的重复，因此被多个物种共享；"Non-Terminal"：STRIDE检查基因树的拓扑结构在复制后应该是什么；
- 6.Genes 1：基因列表来自复制基因的一个副本，逗号分隔；
- 7.Genes 2：基因列表来自复制基因的另一个副本，逗号分隔。

```
Orthogroup      Species Tree Node       Gene Tree Node  Support Type    Genes 1 Genes 2
OG0000000       N5      n62     0.5     Non-Terminal    Mcandidum_pep_mc33950   Mdodecandrum_pep_DR005872, Mcandidum_pep_mc01326, Mdodecandrum_pep_DR006000, Mcandidum_pep_mc01492
OG0000000       N5      n63     1.0     Non-Terminal    Mdodecandrum_pep_DR005872, Mcandidum_pep_mc01326        Mdodecandrum_pep_DR006000, Mcandidum_pep_mc01492
OG0000000       N5      n177    0.5     Non-Terminal    Mdodecandrum_pep_DR002276, Mdodecandrum_pep_DR013872, Mcandidum_pep_mc36653, Mdodecandrum_pep_DR026891, Mcandidum_pep_mc22876, Mcandidum_pep_mc37513, Mdodecandrum_pep_DR026886 Mdodecandrum_pep_DR004265
```

2. SpeciesTree_Gene_Duplications_0.5_Support.txt
- 提供了物种树分支上的上述重复的总和。
- 它是一个 newick 格式的文本文件。
- 每个节点或物种名称后面的数字是在导致节点/物种的分支上发生的具有至少 50% 支持度的基因复制事件的数量。
- 分支长度是标准分支长度，如 Species_Tree/SpeciesTree_rooted.txt 中给出的。

```
(((((Mcandidum.pep_7199:1,Mdodecandrum.pep_3425:1)N5_9299:1,Egrandis.pep_12485:1)N3_296:1,((Ptrichocarpa.pep_13300:1,(Csinensis.pep_5628:1,(Graimondii.pep_15253:1,Athaliana.pep_8750:1)N10_30:1)N8_5:1)N6_61:1,(Ppersica.pep_5798:1,(Mtruncatula.pep_23336:1,Csativus.pep_3576:1)N9_48:1)N7_12:1)N4_19:1)N2_96:1,Vvinifera.pep_7596:1)N1_535:1,Mguttatus.pep_8911:1)N0_1806;
```

3. Orthogroups.txt
- Orthogroups.txt保存了鉴别的orthogroups的信息。
- 第一列是OG开头的orthogroups的ID号，冒号结束；后面不同数量的列是此orthogroup包含的各物种的基因ID。

## 4.3. 筛选基因复制事件和复制的基因对
### 4.3.1. 筛选基因家族
从Orthogroups.txt文件中筛选符合标准的orthogroups，提取orthogroups的ID号。
1. 筛选标准
- 每个基因家族至少包含四个基因。
- 每个基因家族至少包含一个外类群基因。
2. 筛选命令
- `cat /path/to/OrthoFinder/Results_xx/Orthogroups/Orthogroups.txt |awk 'NF>4 {print $0}' |grep "OUTGROUP"|sed "s/:.*//g" >orthogroups_filter.list`
- 其中`awk 'NF>4 {print $0}'`代表提取列数大于4的行，即包含至少4个基因的基因家族。
- 其中`grep "OUTGROUP"`代表提取有OUTGROUP字符串的行，这里提取包含外类群基因ID特征(比如Eugr)前缀的行。
- 其中`sed "s/:.*//g"`代表把冒号及之后的所有信息删除，只保留orthogroups的ID信息。

### 4.3.2. 筛选基因复制事件
从Duplications.tsv文件中筛选符合标准的基因复制事件，提取基因复制事件中发生复制的基因。
1. 提取特定枝上的基因复制事件
- 比如这里是要筛选Mcandidum和Mdodecandrum的最近共同祖先发生的WGDs事件，在上面SpeciesTree_Gene_Duplications_0.5_Support.txt文件的例子中，是N5代表的节点。
- 则可以通过`cat Duplications.tsv|awk -v FS='\t' '$2=="N5" {print $0}' >N5.tsv`来获取第二列是N5的复制事件
2. 其他筛选标准
- 两个子枝（child branch）都有来自分析物种A的基因。都有才代表物种A的基因复制了。
- 支持率（这里用Duplications.tsv第4列Support值）大于等于0.5。
3. 筛选命令
- `cat N5.tsv|awk -v FS='\t' '$4>=0.5 && $6 ~ "mc" && $7 ~ "mc" {print $0}' >N5_filter.tsv`
- 其中`$6 ~ "mc" && $7 ~ "mc"`代表模糊匹配第六列和第七列都包含mc字段的行。这里的mc是我们想要筛选的物种的基因ID的前缀。即实现了筛选两个子枝（child branch）都有来自分析物种A的基因。
- 其中`$4>=0.5`代表第四列大于等于0.5的行。即实现了筛选支持率大于等于0.5的行。
3. 由于OrthoFinder2运行时也会有一些基础的筛选，所以有可能这步筛选结束后与筛选前完全一致。

### 4.3.3. 整理数据
1. 取以上基因家族和基因复制事件筛选结果的交集
- `grep -f orthogroups_filter.list N5_filter.tsv |awk '{print "dup"NR"dup",$0}' |sed "s/ /\t/g" >N5_filter_OG_dup.tsv`
- 其中`awk '{print "dup"NR"dup",$0}'`代表在行首加上`dup12dup`这列，其中12是行号，目的是为每一行的基因复制事件取一个不重复的ID。
2. 整理数据
- 把N5_filter_OG_dup.tsv整理成基因对格式的文件genepairs.tsv。
- genepairs.tsv是tab分隔的文本文件，包含三列，第一列是N5_filter_OG_dup.tsv的第一列，这里是`dup12dup`形式的基因复制事件的ID；第二列和第三列分别是两个子枝（child branch）中各取一个基因生成的基因对，迭代生成所有可能的基因对。
- 写了一个python小脚本来实现N5_filter_OG_dup.tsv到genepairs.tsv的转换。

```python
# 此脚本用于从文本文件中提取指定列的内容，并根据需要迭代生成新格式。

import sys, math, os

# 解析文件的列内容
with open('N5_filter_OG_dup.tsv','r') as f: #读取N5_filter_OG_dup.tsv文件
    for l in f.readlines(): #按行提取
        line = l.strip()
        dr = line.split("\t")[0] #提取每行的第一列，写入dr
        genes1 = line.split("\t")[6] #提取每行的第七列，写入genes1
        genes2 = line.split("\t")[7] #提取每行的第八列，写入genes2
        gene_1 = list(genes1.split(",")) #提取genes1中的geneID，保存成list格式
        gene_2 = list(genes2.split(",")) #提取genes2中的geneID，保存成list格式

# 迭代生成基因对
        for i in range(len(gene_1)):
            for j in range(len(gene_2)):
                with open('genepairs.tsv', mode='a') as gp:
                    gp.write(dr+'\t'+gene_1[i]+'\t'+gene_2[j]+'\n')
```

### 4.3.4. 筛除串联重复（tandem duplications）
从上面的结果文件genepairs.tsv中筛除鉴定为串联重复产生的基因对，生成genepairs_del_tandem.tsv。（ps：也可以从N5_filter_OG_dup.tsv筛除）
1. 串联重复的标准
- 取的文章里的标准：在两个子枝（child branch）中的基因在五个基因范围内则被看作是串联重复导致的。
2. 筛除
- 由于我注释基因组的时候，基因ID是根据位置排序产生的，所以这里筛除就直接把genepairs.tsv文件的第二列和第三列的数字相减再取绝对值，如果数字小于等于5的话就把这行删除。另外注意一下染色体首尾的基因就好。
- 如果是注释基因组时基因ID命名没有规则，就需要自行想办法了。

### 4.3.5. 基因复制树和复制的基因对
到这里，就获得了物种A发生特定WGD后保留的基因复制事件（N5_filter_OG_dup.tsv）和相应的基因对（genepairs.tsv或者进行筛除串联重复后的genepairs_del_tandem.tsv）。

## 4.4. 多个WGDs的情况
如果同一个枝（branch）上发生了两次或以上的WGDs事件，通过基因树和物种树无法区分这两次WGDs，此时就需要通过计算基因对的Ks后通过Ks来区分。

### 4.4.1. 计算基因对的Ks
1. 文件准备
- 此前得到了genepairs.tsv文件（或者进行筛除串联重复后的genepairs_del_tandem.tsv），即有了基因对ID文件。
2. 运行
- 参考博客：https://yanzhongsino.github.io/2022/09/07/bioinfo_Ks_batch.calculation.Ks/

### 4.4.2. 区分两次WGDs
通过两次WGDs的Ks峰值来区分，比如两次WGDs的峰值分别为0.2和0.7，取平均值是0.45，则把Ks大于等于0.45的看作0.7代表的WGD，把Ks小于0.45的看作0.2代表的WGD。

有两种方式区分，一种是把基因复制事件分类到两次WGDs中，一种是把基因对分类到两次WGDs中。我之前用的基因复制事件，但想想用基因对可能更准确。

1. 把基因复制事件分类到两次WGDs
- 计算得到基因对的Ks后，把每次基因复制事件的所有基因对的Ks取平均值，根据Ks平均值来区分。
2. 把基因对分类到两次WGDs
- 直接根据基因对的Ks来区分。

## 4.5. WGD后保留基因的分析
得到特定WGD后保留的基因对后，可以用富集分析或其他分析看保留了一些什么基因，是否有进化意义，或者是否与适应、特定的遗传性状等相关。

## 4.6. WGD后保留TF的分析
分析WGD后保留的基因中的转录因子（transcription factor，TF）也是一个可考虑的点。
除了直接看WGD后保留的基因中包含了什么种类和多少数量的TF外，还可以通过利用转录因子数据库PlantTFDB来做WGD后保留的每种TF的保留模式的进一步分析。

详情可参考博客：https://yanzhongsino.github.io/2022/10/18/bioinfo_transcriptionFactor/

# 5. references
1. paper: https://www.sciencedirect.com/science/article/pii/S1674205219303594

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>