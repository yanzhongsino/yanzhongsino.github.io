---
title: 基因流推断 —— Dsuite
date: 2022-04-10
categories: 
- bio
- bioinfo
- gene flow
tags: 
- Dsuite
- gene flow
- hybridization
- introgressive
description: 这篇博客记录的是用Dsuite软件计算D值(ABBA-BABA统计值)和f4-ratio等相关统计量，并推测居群或近缘种间基因流。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=171959&auto=1&height=32"></iframe></div>

# 1. Dsuite
1. Dsuite简介
   - Dsuite是通过计算Patterson's D统计量(即ABBA统计量)和f4等统计量来评估种群间或近缘种间基因流的基于C语言的软件。
2. Dsuite 原理
   - D值（即ABBA统计量）和f4-ratio统计可以表示为适用于四个分类群的双等位基因SNP：P1,P2,P3,O，拓扑是 (((P1,P2),P3),O)。
   - 其中外类群O携带祖先等位基因A，衍生等位基因用B表示。BBAA,ABBA,BABA分别代表四个分类群携带等位的三种模式。
   - 在没有基因流的零假设下，由于具有相同频率的不完全谱系分类，预计P3与P1或P2共享衍生等位基因B的两种模式ABBA和BABA的频率相等，如果ABBA和BABA的频率有显著差异则代表在P3和P1或P2间存在基因渐渗。
   - D=(nABBA-nBABA)/(nABBA+nBABA)；在外群对于祖先等位基因A是固定的（外群中B的频率为0）假设下，D统计量是等位基因模式计数的归一化差异。
   - 如果外群中衍生等位基因B不为0，则Dsuite的D值是Patterson's D，适用于无根的四分类群树。
3. Dsuite输入输出
   - 输入：基因组snp的vcf格式文件，居群树文件(可选optional)
   - 输出：D值统计，f4-ratio统计，f-branch统计，f-branch树矩阵热图
4. Dsuite优势和不足
   - Dsuite的优势是运行非常快(时间以小时计算)
   - 不足是Dsuite分析结果不包含基因流的方向
5. Dsuite适用范围
   - Dsuite适用于基因组学大数据和多样本(超过十个)数据
   - 适用于居群间或物种间的基因流推测
   - 即使每个群体只有一个个体也可以推测基因流
   - 还可以计算pool-seq数据的基因流
   - 相较其他计算D值软件，Dsuite还同时可以计算f4-ratio和f-branch，以及滑窗统计f相关值。

# 2. Dsuite install
1. 安装Dsuite主程序
Dsuite是C编写的，需要编译；需要GCC(>=4.9.0)和zlib压缩库。
编译后可执行文件在Dsuite/Build/目录下，可以加到环境变量中或使用绝对路径。

```shell
git clone https://github.com/millanek/Dsuite.git
cd Dsuite
make
```

2. 安装python3 Fbranch绘图脚本【可选但推荐】
如果要绘制f-branch计算结果，则需要安装这个绘图脚本。注意`--prefix=`后没有任何内容。

```shell
cd utils
python3 setup.py install --user --prefix=
```

# 3. Dsuite分析
## 3.1. 输入文件
1. sample.snp.vcf.gz
   - 基因组尺度的snp文件，vcf格式，可用bgzip压缩
2. sets.txt
   - 包括两列，tab分隔；第一列个体名，第二列物种名/居群名。
   - 对于Dtrios模块，至少指定一个个体为外类群，外类群个体的第二列写Outgroup，可有多个。
   - 要注意第二列物种名不能包含短横杠-和句点.字符，否则Dsuite Dtrios虽然不会报错，但Dsuite Fbranch模块运行报错解析不了树文件(ERROR: The tree string could not be parsed correctly)。
3. species.newick【optional】
   - Newick格式的居群/物种树文件，物种名称与sets.txt对应。
   - 外类群名称替换成Outgroup，与sets.txt对应。
   - 树文件必须定根在Outgroup，否则dtools.py报错树文件和franch.out的拓扑不一致。
   - 去掉支持率；否则Dsuite Fbranch模块运行报错解析不了树文件，ERROR: The tree string could not be parsed correctly。
   - 枝长最好也去掉，不去枝长也不会被Dsuite用到，还可能增加报错风险。
   - 这个文件对于Dtrios模块是可选项，但建议加上好画fbranch的图。

## 3.2. Dtrios模块
`Dsuite Dtrios`用于计算所有三物种组合的D和f4-ratio统计量

1. 用法
`Dsuite Dtrios sample.snp.vcf.gz sets.txt -t species.newick -o sample`

2. 输入和常用参数
- sample.snp.vcf.gz：变异文件
- sets.txt：分组文件
- -t species.newick：指定物种树文件
- -o sample：指定输出文件前缀，默认是sets
- -p 5：如果样品中包含pool-seq数据，-p用于设置最小深度，设置后从等位基因深度估计群体的等位基因频率。
- -c：不输出sample_combine_stderr.txt和sample_combine.txt；这两个文件用作DtriosCombine的输入，如果不需要可加上-c不输出这两个文件。

3. 结果
7Mb的29个物种的snp数据花费大概3-4小时。

运行结束生成文件：
sample_BBAA.txt，sample_Dmin.txt，sample_tree.txt三个文件结构一致，是在不同假设前提下（意味着三物种组合的不同排序）的统计量输出。包含所有三物种组合的D统计量，Z-score，未调整的p-value，f4-ratio，三种模式（BBAA,ABBA,BABA）的计数。
- sample_BBAA.txt：不参考-t的拓扑，尝试推断树拓扑的统计量。假设正确的树是 BBAA 模式比不一致的 ABBA 更常见的树，并且对每个三重奏进行排序。 BABA 模式，假设是由不完整的谱系分类或基因渗入引起的。
- sample_Dmin.txt：不管关于树拓扑的任何假设，不尝试推断树拓扑，而是输出最小D值的三物种组合排序的统计量。
- sample_tree.txt：如果-t指定物种树则输出此项；与指定树一致的排序的统计量。
- sample.txt
- sample_combine_stderr.txt和sample_combine.txt：用作DtriosCombine的输入，如果不需要可删除

## 3.3. 计算和绘制f-branch
### 3.3.1. 计算f-branch值
`Dsuite Fbranch`是一种启发式方法，执行f-branch计算，用于解释f4-ratio相关结果。

1. 用法
`Dsuite Fbranch species.newick sample_tree.txt >fbranch.out`

2. 输入
- species.newick：指定物种树文件
- sample_tree.txt：需要Dsuite Dtrios并-t指定树拓扑得到的结果文件sample_tree.txt作为输入

3. 结果
fbranch.out：f-branch统计量保存成矩阵格式

### 3.3.2. 绘制f-branch图
用dtools.py脚本绘制f-branch图

1. 用法
`Dsuite/utils/dtools.py fbranch.out species.newick --outgroup Outgroup --use_distances --dpi 1200 --tree-label-size 30`

如果运行log出现Plotting fbranch... Saving plots可以看到图片文件生成，即使报错Segmentation fault (core dumped)也没关系。

2. 输入和常用参数
- fbranch.out：指定fbranch文件
- species.newick：指定物种树文件
- --outgroup：指定外类群（与fbranch.out和species.newick一致，一般是Outgroup）
- --use_distances：画树时使用newick文件里节点距离
- --dpi：设置png分辨率，有些期刊投稿要求1200，800，600不等；最好高点。
- --tree-label-size：设置树节点标签大小

3. 结果
画Fbranch的图，得到fbranch.svg和fbranch.png；

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/bioinfo_geneflow.Dsuite_fbranch.png?raw=true" width=90% title="f-branch示意图(A4,(A3,(A2,A1)B)C)D" align=center/>

**<p align="center">Figure 1. f-branch示意图</p>**

# 4. reference
1. [Dsuite github](https://github.com/millanek/Dsuite)
2. [Dsuite paper](https://onlinelibrary.wiley.com/doi/10.1111/1755-0998.13265)