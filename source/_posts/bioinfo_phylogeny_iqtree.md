---
title: 构建系统发育进化树：iqtree2构建最大似然树
date: 2023-09-26
categories: 
- bioinfo
- phylogeny
- phylogeny inference
tags:
- biosoft
- phylogeny
- phylogeny inference
- evolutionary tree
- iqtree2
- maximum likelihood tree
- ML tree
- Bayesian information criterion
- BIC
- ultrafast bootstrap approximation
- UFBoot
description: 记录使用iqtree2来构建最大似然树。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1304261116&auto=1&height=32"></iframe></div>

# 1. 建树的原料——输入文件
1. 数据类型
- 通常是使用分子数据的比对文件进行建树，可以是DNA序列、RNA序列、氨基酸序列；
- 也可以用二进制数据binary alignments（0和1），形态数据morphological data（0-9和A-Z表示）。

2. 使用**多序列比对(Multiple Sequence Alignment,MSA)**文件来建树
- 关于MSA可以参考这篇博客(https://yanzhongsino.github.io/2021/09/06/bioinfo_align_MSA/)。

3. MSA文件（sample_aln.fas）的准备
- 如果是组学数据（大数据量），推荐使用MAFFT比对，再用trimAI过滤，以获取比对良好的sample_aln.fas文件。
- 如果是一个或几个基因数据，则可以使用MEGA等软件带的MUSCLE或Clustal软件进行比对，比对完成后一定要手动检查所有比对区域并调整。
- 注意如果是CDS区域，使用CODON模式进行三碱基联合的比对。 

# 2. 常用命令
`nohup iqtree -s samples_aln.fas -o OUTGROUP -T AUTO -m MFP -p partition.nex -b 1000 -alrt 1000 --prefix samples &`

# 3. 参数解释
1. -s samples_aln.fas：指定多序列比对文件，可以是FASTA/PHYLIP/NEXUS/CLUSTAL/MSF格式
2. -o OUTGROUP：指定外类群（只在最后生成树文件时生效），指定多个物种时用逗号隔开
3. -T AUTO：指定线程，推荐系统自动检测最佳线程数（AUTO），有时用户设定的会出现过高或过低的情况
4. -m MFP：指定模型，MFP代表调用iqtree2内置的ModelFinder Plus鉴定最佳模型，默认使用BIC分数最小的模型
5. -p partition.nex：指定分区文件，支持NEXUS格式或RAxML-style格式。如果需要对多个分区进行独立的模型指定，则使用分区文件partition.nex来指定分区。
- 如果partition.nex文件中指定了比对文件，则可以省略-s samples_aln.fas参数。
- 下面是分区文件partition.nex的NEXUS格式示例，可以指定多个文件和一个文件的多个区域。

```shell
#nexus
begin sets;
        charset part1 = sample1_aln.fas:1-860; # 指定part1对应sample1_aln.fas文件的第1-860个碱基
        charset part2 = sample1_aln.fas:980-1128; # 指定part1对应sample1_aln.fas文件的第980-1128个碱基
        charset part3 = sample2_aln.phy; # 指定part3对应sample2_aln.phy文件的所有碱基位点
		charpartition mine = HKY:part1, GTR+I+G:part2, WAG+I+G:part3; # 这行为每个part指定不同的模型
end;
```

6. -b 1000：使用bootstrap自展法计算节点支持率，最少100次，建议1000次或以上。推荐数据量少时用，更准确。
7. -B 1000 --bnni：使用超快速bootstrap自展法（ultrafast bootstrap approximation, UFBoot）计算节点支持率，最少1000次，建议5000次或以上。推荐数据量大时用，比-b快10倍左右。一般推荐加上--bnni参数，代表使用NNI优化超快速bootstrap的树。
8. -alrt 1000：使用SH近似似然比检验计算节点支持率，最小1000次，可以和-b或-B一起使用，同时计算两个支持率。
9. --prefix samples：指定生成的结果文件的前缀为samples。

# 4. 其他常用参数
1. -m MFP+ASC：当使用的输入数据是不包含恒定位点（constant sites）的比对文件时（比如SNP数据，比如形态学数据），开发者建议使用+ASC模型（ascertainment bias correction）来矫正因缺乏恒定位点（constant sites）所导致的系统发育树分枝长度过长与祖先结构错误等问题（具体解释见后文）。
- +ASC模型要求输入的数据必须是variable sites（定义看后面的解释），如果比对文件（如包含SNP位点的比对文件）出现constant sites，则报错`ERROR: Invalid use of +ASC because of 46145 invariant sites in the alignment`，并生成去除constant sites的samples_aln.fas.varsites.phy。
- 如果报错中断，可以用-s指定新生成的variable位点序列samples_aln.fas.varsites.phy，其他参数不变，继续运行建树即可。
2. --mem 100G：最大可使用内存，单位为G、M或百分数%。
3. -st DNA：指定序列的数据类型，可选DNA/CODON/AA/BIN等，默认是自动检测数据类型。
4. -redo：iqtree2会生成每一步的结果文件，如果iqtree运行命令被中断，重新运行一样命令则会默认从断点开始运行。如果想要重新运行，则需要加上-redo参数。
5. -cmax 15：如果你的序列足够长，开发者建议增加-cmax设定值，默认是10，这主要是考虑计算资源。
6. -keep-ident：默认在建树过程中会删除相同序列的样本，待建树完成后再添加，以节省分析时间。但这也会使输出文件中的样本数不足。如果群体中可能包含相同序列，使用-keep-ident参数可以保留序列相同的样本，使输出样本数与输入样本数保持一致。
7. -fast：此参数仅使用 最大简约（maximum parsimony）和 BIONJ 2 种方法构建系统发育树，仅进行 2 轮树的优化（-nstop=2），且无 OPTIMIZING CANDIDATE TREE SET 步骤。否则（不使用 -fast 参数）使用 100 种构建方法（default -ninit=100），并至多迭代 100 次（default -nstop=100）进行树的优化。

# 5. 结果文件
1. `-s`参数指定序列后会生成几个基本的结果文件：
- samples.iqtree：预测的树结果，包含最终树的文本格式，可直接查看文本看到树的拓扑结构。
- samples.treefile：NEWICK格式的ML树，可用iTOL或FigTree等树可视化软件查看。
- samples.log：运行log文件。
2. `-m`参数运行模型选择，生成额外的文件：
- samples.best_scheme.nex：最佳模型（包含碱基替换模型）文件，默认是选择BIC最小的模型为最佳模型，也可以通过参数`-AIC`或`-AICc`选AIC或AICc最小的作为最佳模型。
- samples.model.gz：所有测试模型的对数似然，它充当checkpoint文件来恢复中断的模型选择。
- sample.best_model.nex：最佳模型。
- samples.best_scheme：分区情况。
3. `-B`参数指定超快速自展法评估支持率
- samples.contree：具有指定分支的一致树(consensus tree)支持在原始比对序列上优化分支长度
- samples.splits.nex：以百分比表示的所有splits（二分区）的支持值，以引导树中的出现频率计算。该文件可以用 SplitsTree 程序查看，以探索数据中的冲突信号。因此，它比一致树的信息量更大。例如，这个文件可以看到第二好的冲突split的支持率有多高，即使第二好的拓扑没有被选入一致树。
- samples.ufboot
4. 其他结果文件
- samples.ckp.gz：iqtree自动生成的checkpoint文件，用于恢复中断的运行。使用相同的命令行再次调用iqtree，有这个文件将从上次停止点恢复分析，从而节省之前完成的所有计算时间。
- samples.mldist：似然距离。
- samples.boottrees
- samples.bionj

# 6. 结果解释
1. 多支持率情况
- 如果同时使用-b或-B和-alrt两种方法计算支持率，samples.iqtree文件的支持率在括号中同时展示：（SH-aLRT支持率/UFBoot支持率）
2. 支持率的可信度：不同算法的支持率代表的可信度不一样，一般按下面的标准认为分支具有可信度，iqtree开发者推荐同时满足 UFboot >= 95% 和 SH-aLRT >= 80% 两个标准的分支可信。
- -B的支持率 UFboot >= 95%
- -b的支持率 BS support >= 80%
- -alrt的支持率 SH-aLRT >= 80%

# 7. iqtree使用的一些原理
## 7.1. iqtree对碱基位点（site）类型的定义
iqtree 构建系统发育树时，informative sites 用于推断树的拓扑结构与分枝长度，uninformative site 则用于矫正。

1. iqtree 将位点（site）分为 3 类：constant（invariant）、singleton、informative（variable）。
2. constant site 指位点在所有样本中只包含 1 种碱基；singleton site 指位点包含 2 种类型碱基，但突变仅出现 1 次；informative site 指位点包含 2 种类型碱基，且每种碱基出现至少 2 次。
3. 从算法角度来说，iqtree 无法利用 singleton 对样本分群，即无法提供分群信息，所以 constant 与 singleton 位点属于 uninformative site 。
4. iqtree 对歧义位点（Ambiguous site）采用交集（intersection）形式分析：如果交集非空则忽略歧义碱基，若为空则视为 variant，同列中歧义碱基不堆叠。

```shell
  site        1234567
  species_1   CCCCCCC
  species_2   RTCYRCC
  species_3   CYTYYRT
  species_4   CCCYCRT

  site1 is singleton, because R={A or G}, which exclude C, uninformative.
  site2 is singleton, because Y={C or T}, which include C, ignore Y, uninformative.
  site3 is singleton, uninformative.
  site4 is constant, because Y={C or T}, which include C, ignore Y, uninformative.
  site5 is singleton, because R={A or G}, which exclude C, Y={C or T}, which include C, ignore Y, uninformative.
  site6 is singleton, because R={A or G}, which exclude C, uninformative.
  site7 is informative.
```

5. 由于歧义位点的交集策略，当同列中仅包含非空歧义碱基时，位点会被判定为 constant site（如上例中 site 4）。因此 SNP 数据从 vcf 转 phylip 后存在部分 site 被判定为 constant 的情况。

## 7.2. +ASC模型用于SNP数据或形态学数据的原理
1. 缺乏恒定位点（constant sites）的数据（例如SNP数据/形态学数据）带来的问题
- 由于个体间固定长度区间内 SNP 越多，说明个体间积累的突变越多，分离的时间越长。所以 SNP 构成的 Fasta 数据大幅增加了突变的密度，使个体间分离时间大幅提前，导致分枝过长。 
- 同时，分枝过长会压缩祖先之间的分枝长度，降低祖先之间拓扑结构推断的准确性。因为祖先与个体之间时间跨度越长，祖先的可能性就越多，如果祖先对应一个概率分布，则祖先与祖先之间的分布交集就越大。 
- 例如 A / B 的祖先是 D，C 的祖先是 E，D / E 的祖先是 F，如果 A / B / C 与 D / E 的距离越远，D / E 的可能序列就越多，D / E 分布重合的区间就越大；因为 D / E 间相似度高，所以 D / E 与 F 的距离会缩短，即压缩 D / E 与祖先 F 之间的分枝长度，降低树结构预测的准确性。 

2. 为了矫正缺乏恒定位点（constant sites）的数据导致发育树分枝过长的情况，iqtree 提供 +ASC 参数（ascertainment bias correction）。   

3. 有人分别测评了 GTR+F+G4 和 GTR+F+G4+ASC 两种替换模型下的树结构，结果参见下节（Test 2&3）。+ASC 虽然对 Log-likelihood 影响较小（1.2%=1-79848/80786），但对树结构影响较大（23.3%=1-5.536/7.216），且能够大幅提高稳定性（148%->0%）。

4. 不同参数的性能测试

<table><thead><tr><th>Test</th><th>input</th><th>model</th><th>optimize</th><th>Log-likelihood</th><th>Total tree length</th><th>Sum of internal branch lengths (ratio)</th><th>near-zero internal branches</th><th>Lmap not informative ratio</th></tr></thead><tbody><tr><td>1</td><td>min4.phy</td><td>GTR+F+G4</td><td>fast</td><td>-95636</td><td>3.751</td><td>0.799 (21%)</td><td>93</td><td>8.48%</td></tr><tr><td>2</td><td>varsites.phy</td><td>GTR+F+G4</td><td>fast</td><td>-80786</td><td>7.216</td><td>1.549 (21%)</td><td>97</td><td>14.82%</td></tr><tr><td>3</td><td>varsites.phy</td><td>GTR+F+G4+ASC</td><td>fast</td><td>-79848</td><td>5.536</td><td>1.185 (21%)</td><td>101</td><td>0%</td></tr><tr><td>4</td><td>varsites.phy</td><td>GTR+F+R6+ASC</td><td>fast</td><td>-77426</td><td>1.327</td><td>0.269 (20%)</td><td>263</td><td>0%</td></tr><tr><td>5</td><td>varsites.phy</td><td>GTR+F+R6+ASC</td><td>complete</td><td>-77239</td><td>1.079</td><td>0.210 (19%)</td><td>316</td><td>0%</td></tr></tbody></table>

- Test 1&2：使用 iqtree 过滤 SNP 数据集中的 constant site 后再输入 iqtree，不仅可以使用 +ASC 提高模型精度，还能减少建树时间（min4 299s，varsites 50s）。
- Test 2&3：ASC 使树的分枝长度大幅降低（23.3%=1-5.536/7.216），且能够大幅提高稳定性（148%->0%），但对似然值改变较小（1.2%=1-79848/80786）。 
- Test 3&4：更换模型使树的分枝长度大幅降低（75%=1-1.328/5.536），但似然值改变较小（3%=1-77426/79848）。
- Test 4&5：多轮迭代 使树的 分枝长度 大幅降低（20%=1-1.079/1.328），但似然值改变微小（3‰=1-77239/77426）。 
- Test all：所有模型中中间分枝（internal branch）长度占总枝长比例基本固定（约 20%），near-zero 与 tree length 相关性较高，当发育树整体分枝长度增加时，内部分枝长度也相应增加，其中近 0 分枝数量减少。中间分枝长度占比 20 % 说明 80% 的枝长分布在 末端分枝，个体间距离较大，分离时间较早。部分祖先间的进化关系太紧密（near-zero），用多叉树描述可能更符合这些节点。

# 8. reference
1. iqtree manual：http://www.iqtree.org/doc/
3. iqtree常见问题官方解答：http://www.iqtree.org/doc/Frequently-Asked-Questions#how-do-i-interpret-ultrafast-bootstrap-ufboot-support-values
4. ASC模型解释：https://blog.csdn.net/sinat_41621566/article/details/125427326

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>