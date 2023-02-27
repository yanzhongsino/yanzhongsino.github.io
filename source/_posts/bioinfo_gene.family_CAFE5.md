---
title: 分析基因家族扩张和收缩 —— CAFE5
date: 2021-10-29 16:50:00
categories:
- bio
- bioinfo
tags:
- gene family
- biosoft
- CAFE
- phylogeny
description: 记录了分析基因家族扩张收缩的软件CAFE5的安装与使用。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=17059176&auto=1&height=32"></iframe></div>

# 1. CAFE
CAFE(Computational Analysis of gene Family Evolution)是一款以解释系统发育历史的方式分析基因家族大小变化的软件，这种分析常被称为基因家族收缩扩张(Gene family expansions and contractions)分析。

CAFE使用出生和死亡过程来模拟用户指定的系统发育树中的基因获得和丢失，可计算由父节点到子节点的基因家族大小转移率，也可推断祖先物种的基因家族大小，在该模型下生成的基因家族规模分布可以为评估观察到的类群之间家族规模差异的显著性提供基础。

自2005年Hahn课题组提出评估基因家族进化速度和模式的算法，2006年第一个版本CAFE发表后，2020年推出了最新版本[CAFE5](https://academic.oup.com/bioinformatics/article-abstract/36/22-23/5516/6039105?redirectedFrom=fulltext)，之前旧版本的基本模型假设所有基因家族都具有相同的进化速率。新版本支持伽马分布速率类别对家族之间的速率变化进行显式建模。

# 2. CAFE5安装

```
git clone https://github.com/hahnlab/CAFE5.git
cd CAFE5
./configure
make
```

conda可以安装CAFE4
`conda install -c bioconda cafe`安装的是CAFE4

# 3. CAFE5使用
## 3.1. CAFE5输入文件
CAFE5需要至少两个输入文件，一个是基因家族计数文件gene_families.txt，一个是树文件tree.txt
### 3.1.1. gene_families.txt
1. gene_families.txt文件格式
制表符分隔的基因家族计数文件，通常用OrthoMCL, SwiftOrtho, FastOrtho, OrthAgogue, OrthoFinder等软件获取计数信息。

第一列是功能描述Desc，可以为null，第二列是Family ID，其余列是每个物种以及包含对应的基因家族基因的数量。

```gene_families.txt示例
Desc	Family ID	human	chimp	orang	baboon	gibbon	macaque	marmoset rat	mouse	cat	horse	cow
ATPase	ORTHOMCL1	 52	 55	 54	 57	 54	  56	  56	 53	 52	57	55	 54
(null)	ORTHOMCL2	 76	 51	 41	 39	 45	  36	  37	 67	 79	37	41	 49
HMG box	ORTHOMCL3	 50	 49	 48	 48	 46	  49	  48	 55	 52	51	47	 55
(null)	ORTHOMCL4	 43	 43	 47	 53	 44	  47	  46	 59	 58	51	50	 55
Dynamin	ORTHOMCL5	 43	 40	 43	 44	 31	  46	  33	 79	 70	43	49	 50
```

2. 用orthofinder2的结果文件Orthogroups.GeneCount.tsv转换成gene_families.txt文件
`awk -v OFS="\t" '{$NF=null;print $1,$0}' Orthogroups.GeneCount.tsv |sed -E -e 's/Orthogroup/desc/' -e 's/_[^\t]+//g' >gene_families.txt`

3. 如果是MCL获得的基因家族数据。cafe5的tutorials目录下有脚本mcl2rawcafe.py用于把mcl的输出转化成cafe5的输入。
`python mcl2rawcafe.py -i dump.blast_output.mci.I30 -o gene_families.txt -sp "ENSG00 ENSPTR ENSPPY ENSPAN ENSNLE ENSMMU ENSCJA ENSRNO ENSMUS ENSFCA ENSECA ENSBTA"`。

4. 剔除不同物种间的基因拷贝数变异特别大的基因家族。
否则可能无法估计参数，运行中断报错Failed to initialize any reasonable values

- cafe5的tutorial目录下脚本clade_and_size_filter.py可以筛除一个或以上物种有超过100个基因拷贝的基因家族。
`python clade_and_size_filter.py -s -i gene_families.txt -o gene_familie_filter.txt`

脚本运行失败。

- 也可以用awk命令来筛除3-13列中基因家族数量大于等于100的行，可以根据自己的数据更改。
`awk 'NR==1 || $3<100 && $4<100 && $5<100 && $6<100 && $7<100 && $8<100 && $9<100 && $10<100 && $11<100 && $12<100 && $13<100 {print $0}' gene_families.txt >gene_families_filter.txt`

### 3.1.2. tree.txt
1. tree.txt文件格式
tree.txt是二叉的（binary），有根的（rooted），超度量(时间树，ultrametric)的newick格式树。

可以用R包Ape的函数`is.binary`, `is.rooted`, `is.ultrametric`对树是否是二叉有根超度量做检验。

tree.txt文件内容的示例：
((((cat:68.710507,horse:68.710507):4.566782,cow:73.277289):20.722711,(((((chimp:4.444172,human:4.444172):6.682678,orang:11.126850):2.285855,gibbon:13.412706):7.211527,(macaque:4.567240,baboon:4.567240):16.056992):16.060702,marmoset:36.684935):57.315065):38.738021,(rat:36.302445,mouse:36.302445):96.435575);

2. 用paml的Figtree转换成tree.txt文件（newick格式）
`grep -E -v "NEXUS|BEGIN|END" FigTree.tre|sed -E -e "s/\[[^]]*\]//g" -e "s/[ \t]//g" -e "/^$/d" -e "s/UTREE/tree tree/" >tree.txt`

3. CAFE5软件下脚本prep_r8s.py用于构建r8s的批量运行脚本，可用批量运行脚本获得tree。

## 3.2. 运行CAFE5
`cafe5 -i gene_families_filter.txt -t tree.txt -p -k 2 -o k2p`
参数：
-p代表指定root frequency distribution为泊松分布（默认是均匀分布uniform distribution）。
-k 3代表使用GAMMA模型（默认是base模型）并且使用3种gamma rate（代表不同基因家族有着不同的进化速率）。-k的值需要运行多次比较likelihood并确保收敛后才知道使用哪个最好，一般来说2-5之间试一试。
-o 指定输出目录，默认是results。

## 3.3. CAFE5的结果
### 3.3.1. 结果文件
1. model_asr.tre
- 每个基因家族的树的合集，nexus格式
- 树上的物种ID后的下划线隔开的数字是预期的基因家族大小
- 树上的星号代表这个物种的基因家族有显著变化
2. model_clade_results.txt
- 进化树上每个节点扩张或者收缩了多少基因家族
3. model_family_results.txt
- 基因家族变化的p值和是否显著的结果
4. model_results.txt
- 模型，最终似然值，最终Lambda值等参数信息。

还生成了一些其他文件。

### 3.3.2. 选择最优结果
k值的结果比较：
查看k2p，k3p，k5p，k6p等不同的结果文件Gamma_results.txt文件中的第一行信息，Model Gamma Final Likelihood (-lnL)值，挑选最大的为最优结果。

### 3.3.3. 结果整理

1. 对特定物种显著扩张或显著收缩基因的提取

```shell
cat Gamma_change.tab |cut -f1,15|grep "+[1-9]" >sample.expanded #提取Gamma_change.tab第15列代表物种sample的扩张的orthogroupsID
cat Gamma_change.tab |cut -f1,15|grep "-" >sample.contracted  #提取Gamma_change.tab第15列代表物种sample的收缩的orthogroupsID
grep "sample<13>\*" Gamma_asr.tre > sample_significant_trees.tre # 根据sample ID和编号提取sample分支的基因家族显著扩张或收缩的基因家族树（Gamma_asr.tre文件中默认以p<0.05为标准判断变化是否显著）
grep -E -o "OG[0-9]+" sample_significant_trees.tre > sample_significant.ogs # 提取sample分支显著变化的OG IDs （默认以p<0.05为标准）
awk '$2 <0.01 {print $1}' Gamma_family_results.txt >p0.01_significant.ogs # 以p<0.01为标准提取所有显著扩张或收缩的orthogroupsID（根据情况调整，常用p<0.05或p<0.01）
grep -f sample_significant.ogs p0.01_significant.ogs > sample_p0.01_significant.ogs # 提取以p<0.01为标准判断显著性的sample分支基因家族显著变化的OG IDs。
grep -f sample_p0.01_significant.ogs sample.expanded |cut -f1 >s ample.expanded.significant #提取显著扩张的sample物种的orthogroupsID
grep -f sample_p0.01_significant.ogs sample.contracted |cut -f1 > sample.contracted.significant #提取显著收缩的sample物种的orthogroupsID
grep -f sample.expanded.significant ./OrthoFinder/Results_Oct14/Orthogroups/Orthogroups.txt |sed "s/ /\n/g"|grep "bv" |sort -k 1.3n |uniq >sample.expanded.significant.genes #提取显著扩张的基因列表，假设基因ID是bv的前缀。
grep -f sample.contracted.significant ./OrthoFinder/Results_Oct14/Orthogroups/Orthogroups.txt sed "s/ /\n/g"|grep "bv" |sort -k 1.3n |uniq >sample.contracted.significant.genes #提取显著收缩的基因列表，假设基因ID是bv的前缀。
seqkit grep -f sample.expanded.significant.genes sample.pep.fa >sample.expanded.significant.pep.fa #提取显著扩张的基因序列
seqkit grep -f sample.contracted.significant.genes sample.pep.fa >sample.contracted.significant.pep.fa #提取显著收缩的基因序列
```

2. 提取出指定物种的显著扩张和收缩的蛋白序列之后，就可以拿去做GO注释和基因富集分析。

## 3.4. 把每个节点收缩扩张的基因数量画在树上
有看到一个画图脚本，不适用于CAFE5，暂且记录在此。
`python python_scripts/cafetutorial_draw_tree.py -i reports/summary_run1_node.txt -t '((((cat:68.7105,horse:68.7105):4.56678,cow:73.2773):20.7227,(((((chimp:4.44417,human:4.44417):6.68268,orang:11.1268):2.28586,gibbon:13.4127):7.21153,(macaque:4.56724,baboon:4.56724):16.057):16.0607,marmoset:36.6849):57.3151):38.738,(rat:36.3024,mouse:36.3024):96.4356)' -d '((((cat<0>,horse<2>)<1>,cow<4>)<3>,(((((chimp<6>,human<8>)<7>,orang<10>)<9>,gibbon<12>)<11>,(macaque<14>,baboon<16>)<15>)<13>,marmoset<18>)<17>)<5>,(rat<20>,mouse<22>)<21>)<19>' -o reports/summary_run1_tree_rapid.png -y Rapid`

没有试过这个脚本，发表的图还是用R包ggtree自己画。

# 4. references
1. https://academic.oup.com/bioinformatics/article-abstract/36/22-23/5516/6039105?redirectedFrom=fulltext
2. https://github.com/hahnlab/CAFE5


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>