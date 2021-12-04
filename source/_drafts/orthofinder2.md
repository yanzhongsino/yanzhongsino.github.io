---
title: orthofinder
date: 2021-11-19 15:10:00
categories: 
- bio
- bioinfo
tags: 
description: 
---

<div align="middle"><iframe width="560" height="315" src="https://www.youtube.com/embed/wcOM3Rx43ko" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

# 1. 直系同源(orthology)推断
## 1.1. 直系同源推断
直系同源(orthology)是指直系同源基因的直系同源这种属性，相关的概念可以参考[博文-同源性相关概念](https://yanzhongsino.github.io/2021/06/23/concept_homology/)。

直系同源属性的推断常常在多个物种间进行，通常是获取多个物种的直系同源群(orthogroups)和直系同源群间的关系。

直系同源基因判定的方法有两种：
- 一种是将物种的直系同源基因两两比较，不同物种的每一对都是直系同源，但存在基因重复，使得用这种方法找到的同源关系是不具有传递性的（A和B是同源的，B和C是同源的，但是A和C不是同源的）。multiparanoid和OMA方法都是这个原理。所以需要每个同源基因都在多个集合中存在才行，进行两两比较。
- 另一种是识别完整的正交群，一个同源组orthogroup包含所有物种共同祖先基因，同时包含直系／旁系同源基因，OrthoMCL是这个原理，它是通过正交组做推断，用BLAST计算多个物种序列之间的序列相似性得分，然后用MCL聚类算法，识别该数据集中高度相似的序列组。

## 1.2. 直系同源基因推断常用软件：OrthoMCL和OrthoFinder
### 1.2.1. OrthoMCL
OrthoMCL是2003年发表的工具。发表以后得到广泛使用，但存在一些问题：
- 2013年之后就没更新，安装需要使用MySQL。
- 在BLAST序列相似度时发现，长序列比短序列更准确，有时会留下较长的质量差的序列，而丢弃较短的单质量好的序列。
- orthoBench是唯一可用的公开的正交群标准数据集，用orthoBench数据集评估orthoMCL性能，发现它对序列长度有依赖性，太长或者太短的序列都不准确。
- recall率有待提高。

### 1.2.2. OrthoFinder
OrthoFinder是2015年才出现，2019年正式发表的工具。OrthoFinder解决了一些OrthoMCL中存在的问题，展现了优势：
- 持续更新，安装友好，运行速度快。
- OrthoFinder解决了这个问题，它将BLAST评分做了转换，将原来的e值准换成bit值（比特值）来减少序列长度对准确度的影响，因为e值存在阈值，e 小于10的－180次方的都被算作0，而比特值没有阈值。
- OrthoFinder用RBNH代替了传统的RBH提高了recall。

这篇博文主要介绍OrthoFinder这个工具。

# 2. OrthoFinder
## 2.1. OrthoFinder的功能
OrthoFinder是用于推断系统发育直系同源关系的基于python的软件。OrthoFinder1模块推断直系同源群，OrthoFinder2模块在直系同源群的基础上构建物种系统发育树，步骤如下：
1. 推断所有物种的直系同源群和直系同源基因。
- 用all vs all比对所有输入的物种蛋白序列；
- RBNHs方法确定阈值；
- MCL聚类获得直系同源组。
2. 推断找到的所有直系同源基因的有根基因树。
- 为每个直系同源组构建基因系统发育树；
- 使用STAG算法从无根基因树上构建无根物种树；
- 使用STRIDE算法构建有根物种树；
- 有根物种树进一步辅助构建有根基因树。
3. 用基因树推断基因间的所有直系同源群间的关系。
4. 推断基因复制事件并交叉引用到对应的基因树和物种树上的对应节点。
5. 提供比较基因组学统计结果。

<img src="https://media.springernature.com/lw685/springer-static/image/art%3A10.1186%2Fs13059-019-1832-y/MediaObjects/13059_2019_1832_Fig2_HTML.png?as=webp" width=80% title="The OrthoFinder workflow" alt="The OrthoFinder workflow" align=center/>

## 2.2. OrthoFinder的分步原理
### 2.2.1. OrthoFinder1推断直系同源群的基本原理
- 首先BLAST all vs all 做所有物种蛋白序列的比对。选best hits Top5%，用比特值代替e值，用最小二乘法将Top5%的分数值拟合成一个线性模型，将比特值用这个模型转换之后，较长的质量差的hit将不再优于短的质量好的序列；
- 然后将基因长度和系统发育距离标准化。用的是RBNH方法，传统的是用RBH方法，它是一种高精度鉴定直系同源基因对的方法，用BLAST best hit的倒数做的，但是RBNH是用标准话的比特值来做的。

### 2.2.2. OrthoFinder1推断直系同源群的工作流程
OrthoFinder1推断直系同源群的分析过程分为如下几步:
1. BLAST all-vs-all搜索。使用BLASTP以evalue=10e-3进行搜索，寻找潜在的同源基因。(除了BLAST, 还可以选择DIAMOND和MMSeq2等其他搜索工具)。
2. 基于基因长度和系统发育距离对BLAST bit得分进行标准化。
3. 使用RBNHs方法，确定一个同源组序列性相似度的阈值，筛选出质量好的直系同源组(orthogroup graph)。
4. 构建直系同源组图(orthogroup graph)，通过归一化的比特值画出正交组的边缘连接，用作MCL的输入。
5. 使用MCL对基因进行聚类，划分直系同源组。

### 2.2.3. OrthoFinder2在OrthoFinder1的基础上构建物种系统发育树的流程
1. 为每个直系同源组构建基因系统发育树；
2. 使用STAG算法从无根基因树上构建无根物种树；
3. 使用STRIDE算法构建有根物种树；
4. 有根物种树进一步辅助构建有根基因树；
5. 基于DLC(Duplication-Loss-Coalescent)分析模型，有根基因树可以用来推断物种形成和基因复制事件；
6. 最后把结果记录在统计信息中。

## 2.3. OrthoFinder使用
### 2.3.1. OrthoFinder安装
1. conda安装【推荐】
conda安装一次解决依赖问题：`conda install orthofinder`

2. 下载解压缩
[OrthoFinder最新版下载地址](https://github.com/davidemms/OrthoFinder/releases)

下载后解压缩：`tar xzf OrthoFinder_source.tar.gz` or `tar xzf OrthoFinder.tar.gz`

即可使用：`python OrthoFinder_source/orthofinder.py -h` or `./OrthoFinder/orthofinder -h`

### 2.3.2. OrthoFinder运行
#### 2.3.2.1. 输入文件
把需要分析的物种的组学蛋白序列文件(.fa,.faa,.fasta,.fas,.pep后缀)放在同一个目录下(例如workdirectory目录下)，然后用OrthoFinder的-f参数指定目录做分析。

如果是下载的蛋白数据，使用最长转录本的蛋白序列即可。

#### 2.3.2.2. 简单版
`orthofinder -f ./workdirectory` # -f指定数据目录

#### 2.3.2.3. 参数版
`orthofinder -f ./workdirectory -t 24 -a 8 -M msa -S blast -A mafft -T raxml-ng`

- -t指定比对线程;
- -a指定分析线程;
- -M指定推断基因树的方法: dendroblast(default)/msa;
- -S指定比对软件: diamond(default)/blast/blast_gz/mmseqs/blast_nucl
- -A指定多序列比对(MSA)使用软件：mafft(default)/muscle；需要指定-M msa
- -T指定画树软件：fasttree(default)/raxml/raxml-ng/iqtree

#### 2.3.2.4. 修改参数文件config.json
1. 如果需要做更细致的参数设定，比如对调用软件blast的参数evalue阈值的修改，可以通过修改参数文件config.json来实现。
2. config.json文件位置：下载安装的在安装目录下，conda安装的在conda的bin目录下。
3. 推荐修改值：
- raxml-ng参数修改为--model LG+G8+F --threads 2 --bs-trees 1000（default是--model LG+G4 --threads 1)；
- blastp和blastn的参数修改为 -evalue 0.00001（default是0.001）

#### 2.3.2.5. RESTART
如果需要重新运行，可以使用以下参数设定从哪个步骤开始运行。
- -b  <dir> # Start OrthoFinder from pre-computed BLAST results in <dir>
- -fg <dir> # Start OrthoFinder from pre-computed orthogroups in <dir>
- -ft <dir> # Start OrthoFinder from pre-computed gene trees in <dir>

1. 添加新物种
运行之后想要添加物种可以用`orthofinder -b previous_orthofinder_directory -f new_fasta_directory`而无需重新运行先前计算的 BLAST 搜索。

这会将“new_fasta_directory”中的每个物种添加到现有物种集中，重用所有以前的 BLAST 结果，仅执行新物种所需的新 BLAST 搜索并重新计算正交群。'previous_orthofinder_directory' 是包含文件 'SpeciesIDs.txt' 的 OrthoFinder 'WorkingDirectory/'。

2. 移除物种
OrthoFinder 允许您从以前的分析中删除物种。
在先前分析的“WorkingDirectory/”中有一个名为“SpeciesIDs.txt”的文件。使用“#”字符注释掉要从分析中删除的任何物种，然后使用以下命令运行 OrthoFinder：`orthofinder -b previous_orthofinder_directory`

其中 'previous_orthofinder_directory' 是包含文件 'SpeciesIDs.txt' 的 OrthoFinder 'WorkingDirectory/'。

3. 同时添加和删除物种
前面两个选项可以结合起来，如上所述注释掉要删除的物种，然后使用命令：
`orthofinder -b previous_orthofinder_directory -f new_fasta_directory`

#### 2.3.2.6. 结果文件
OrthoFinder的标准输出包括：直系同源组，直系同源基因，有根基因树，解析基因树，无根物种树、有根物种树，基因重复事件以及相关的统计数据。

1. Phylogenetic_Hierarchical_Orthogroups 文件夹
    
    从 2.4.0 版本开始，主要文件储存在这个目录下面，orthogroups目录被弃用。
    
    OrthoFinder 通过分析有根基因树来推断每个层次级别（即物种树中的每个节点）的 HOG、正交群。这是一种比所有其他方法使用的和以前由 OrthoFinder 使用的基于基因相似性/图形的方法（已弃用的 Orthogroups/Orthogroups.tsv 文件）更准确的正交群推断方法。
    
    根据 Orthobench 基准，这些新的正交群比 OrthoFinder 2 正交群 (Orthogroups/Orthogroups.tsv) 准确 12%。通过包括外群物种，可以进一步提高准确度（在 Orthobench 上准确度提高 20%），这有助于解释有根基因树。

    由于复制本在进化之间存在突变速率的异质性，所以在研究同源基因时更希望所研究的同源基因来自相同的复制本。Hierarchical Orthogroups（HOG）就是为这一目的而设立的概念，HOG 指由最近共同祖先中某一基因进化而来的一组直系同源基因，进化过程中不涉及基因复制，所以 HOG 中不包含旁系同源。

- 参阅“Species_Tree/SpeciesTree_rooted_node_labels.txt”以确定哪个 N?.tsv 文件包含您需要的正交群。
- N0.tsv是制表符分隔的文本文件。每行包含属于单个正交群的基因。来自每个正交群的基因被组织成列，每个物种一个。附加列给出了 HOG（分层正交群）ID 和基因树中确定 HOG 的节点（注意，这可能位于包含基因的进化枝的根上方）。该文件有效地替换了使用 MCL 进行马尔可夫聚类的Orthogroups/Orthogroups.tsv 中的正交群。
- N1.txt, N2.tsv, ... : Orthogroups 从对应物种树 N1、N2 等物种进化枝的基因树推断出来。现在可以在分析中包含外群物种，然后使用 HOG 文件获取为物种树中所选进化枝定义的正交群。

2. Orthologues 文件夹
- Orthologues 目录包含每个物种的一个子目录，该子目录又包含每个成对物种比较的文件，列出该物种对之间的直向同源物。
- 直向同源物可以是一对一、一对多或多对多，这取决于直向同源物分化后的基因复制事件。
- 文件中的每一行都包含一个物种中的基因，这些基因是另一个物种中基因的直向同源物，并且每一行都与包含这些基因的直向群交叉引用。

3. Orthogroups 文件夹【2.4.0版本后弃用】
从 2.4.0 版本开始，主要文件储存在Phylogenetic_Hierarchical_Orthogroups目录下面，orthogroups目录被弃用。
- Orthogroups.tsv【2.4.0版本后弃用】：制表符分隔的文本文件。每行包含属于单个正交群的基因。来自每个正交群的基因被组织成列，每个物种一个。应改用 Phylogenetic_Hierarchical_Orthogroups/N0.tsv 中的正交群。
- Orthogroups.txt【旧格式】：包含 Orthogroups.tsv 文件内容的 OrthoMCL 输出格式文件。
- Orthogroups_UnassignedGenes.tsv：是制表符分隔的文本文件，其格式与 Orthogroups.csv 相同，但包含未分配给任何正交群的所有基因，即 MCL 中 未成功聚类（直系同源组中基因数 >= 1）的离散基因。
- Orthogroups.GeneCount.tsv：是一个制表符分隔的文本文件，其格式与 Orthogroups.csv 相同，但包含每个正交群中每个物种的基因数计数，可用于基因收缩扩张分析。
- Orthogroups_SingleCopyOrthologues.txt：是一个正交群列表，每个物种只包含一个基因，即它们包含一对一的直向同源物（单拷贝直系同源基因）。它们非常适合物种间比较和物种树推断。

4. Gene_Trees 文件夹
- 记录了每个 orthogroup（gene_num >= 4）的有根基因树结构。

5. Resolved Gene Trees文件夹
- 为具有 4 个或更多序列的每个正交群推断出有根的系统发育树，并使用OrthoFinder的杂种-重叠/重复-丢失溯祖模型(hybrid species-overlap/duplication-loss coalescent model)进行解析。

6. Species_Tree 文件夹
- SpeciesTree_rooted.txt：STAG物种树从所有orthogroups推断，含有内部节点STAG支持值，并使用STRIDE生根，计算出的有根物种树结构。
- SpeciesTree_rooted_node_labels.txt：与上述相同的树，但节点被赋予标签(N0,N1, . . . , N m N_0,N_1,...,N_mN0,N1,...,Nm)（而不是支持值）以允许其他结果文件交叉引用物种树中的分支/节点（例如基因复制事件的位置）。
- Orthogroups_for_concatenated_alignment.txt：仅在 -M msa 模式下输出，列出了所有串联起来用于推断物种树的 orthogroup ID。

7. Comparative_Genomics_Statistics 文件夹
- Duplications_per_Orthogroup.tsv：是一个制表符分隔的文本文件，它给出了每个正交群中标识的重复数。此数据的主文件是 Gene_Duplication_Events/Duplications.tsv。
- Duplications_per_Species_Tree_Node.tsv：是一个制表符分隔的文本文件，它给出了识别为沿着物种树的每个分支发生的重复数。此数据的主文件是 Gene_Duplication_Events/Duplications.tsv。
- Orthogroups_SpeciesOverlaps.tsv：是一个制表符分隔的文本文件，其中包含作为方阵的每个物种对之间共享的正交群的数量。
- OrthologuesStats_*.tsv：是制表符分隔的文本文件，包含矩阵给出每对物种之间一对一、一对多和多对多关系中的直向同源物数量。
    - OrthologuesStats_one-to-one.tsv是每个物种对之间一对一直向同源物的数量。
    - OrthologuesStats_many-to-many.tsv包含每个物种对的多对多关系中的直向同源物的数量（由于物种形成后两个谱系中的基因重复事件）。条目 (i,j) 是物种 i 中与物种 j 中的基因存在多对多直系关系的基因数。
    - OrthologuesStats_one-to-many.tsv：条目 (i,j) 给出物种 i 中与物种 j 的基因处于一对多直系关系的基因数量。这里有一个示例结果文件的演练： https : //github.com/davidemms/OrthoFinder/issues/259。
    - OrthologuesStats_many-to-one.tsv：条目 (i,j) 给出物种 i 中与物种 j 的基因处于多对一直系关系的基因数量。这里有一个示例结果文件的演练： https : //github.com/davidemms/OrthoFinder/issues/259。
    - OrthologuesStats_Total.tsv包含任何多样性的每个物种的直向同源物对的总数。条目 (i,j) 是物种 i 中在物种 j 中具有直向同源物的基因总数。

- Statistics_Overall.tsv：是一个制表符分隔的文本文件，其中包含有关正交群大小和分配给正交群的基因比例的一般统计信息。
- Statistics_PerSpecies.tsv：是一个制表符分隔的文本文件，包含与 Statistics_Overall.csv 文件相同的信息，但针对每个单独的物种。

文件“Statistics_Overall.csv”和“Statistics_PerSpecies.csv”中的大部分术语都是不言自明的，其余的定义如下。
- Species-specific orthogroup：完全由来自一个物种的基因组成的正交群。
- G50：正交群中的基因数量，使得 50% 的基因处于该大小或更大的正交群中。
- O50：正交群的最小数量，使得 50% 的基因处于该大小或更大的正交群中。
- 单拷贝正群：每个物种只有一个基因（并且没有更多）的正群。这些正交群是推断物种树和许多其他分析的理想选择。
- 未分配基因：未与任何其他基因放在同一个群中的基因。

8. Gene_Duplication_Events 文件夹
- Duplications.tsv：是一个制表符分隔的文本文件，它列出了通过检查每个正群基因树的每个节点识别出的所有基因复制事件。列是“Orthogroup”，“Species Tree node”（发生复制的物种树的分支，参见Species_Tree/SpeciesTree_rooted_node_labels.txt），“Gene tree node”（与基因复制事件对应的节点，参见相应的orthogroup Resolved_Gene_Trees/) 中的树；"Support"（存在复制基因的两个副本的预期物种的比例）；“Type”（"Terminal"：物种树终端分支上的重复，"Non-Terminal"：物种树内部分支上的重复，因此被多个物种共享，"Non-Terminal"：STRIDE检查基因树的拓扑结构在复制后应该是什么）；“Genes 1”（基因列表来自复制基因的一个副本），“Genes 2”（基因列表来自复制基因的另一个副本）。
- SpeciesTree_Gene_Duplications_0.5_Support.txt：提供了物种树分支上的上述重复的总和。它是一个 newick 格式的文本文件。每个节点或物种名称后面的数字是在导致节点/物种的分支上发生的具有至少 50% 支持度的基因复制事件的数量。分支长度是标准分支长度，如 Species_Tree/SpeciesTree_rooted.txt 中给出的。

9. Orthogroup_Sequences 文件夹
- FASTA格式文件，每个正交群的 FASTA 文件给出了正交群中每个基因的氨基酸序列。

10. Single_Copy_Orthologue_Sequences 文件夹
- FASTA格式文件，包含每个单拷贝 Orthogroup 所包含的氨基酸的序列信息。

11.  WorkingDirectory 文件夹
- OrthoFinder 运行所需的必需中间文件，如 DIAMOND 比对结果，STAG 输出的无根物种树等。

12. MultipleSequenceAlignments 文件夹
- 此文件夹仅在 -M msa 模式下输出，均为 FASTA 格式文件。
- 记录了每个 orthogroup 中序列间的多序列比对结果。
- 记录了程序通过 CMSA 算法过滤后的 orthogroup 中各序列串联后的多序列比对结果，同时比对结果中空位数 > 50% 的列已被删除。

# 3. references
[tutorial](https://davidemms.github.io/)
[github](https://github.com/davidemms/OrthoFinder)
[paper](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1832-y)
[xuzhougeng's blog](https://xuzhougeng.top/archives/OrthoFinder2-fast-and-accurate-phylogenomic-orthology-analysis-from-gene-sequences)
[website](https://blog.csdn.net/weixin_44614936/article/details/101640473)