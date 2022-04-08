---
title: 用Dsuite推断基因流
date: 2022-04-03 22:00:00
categories: 
- bio
- bioinfo
tags: 
- Dsuite
- gene flow
- hybridization
- introgressive
description: 这篇博客记录的是用Dsuite软件计算D值(ABBA-BABA统计值)和f4比等相关统计量，用于推测居群或近缘种间基因流。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1296539261&auto=1&height=32"></iframe></div>

# Dsuite
1. Dsuite简介
   - Dsuite是通过计算Patterson's D统计量(即ABBA统计量)和f4等统计量来评估种群间或近缘种间基因流的基于C语言的软件。
2. Dsuite适用范围
   - Dsuite适用于基因组学大数据和多样本(超过十个)数据
   - 适用于居群间或物种间的基因流推测
   - 即使每个群体只有一个个体也可以推测基因流
   - 还可以计算pool-seq数据的基因流
   - 相较其他计算D值软件，Dsuite还同时可以计算f4-ratio和f-branch，以及滑窗统计f相关值。
3. Dsuite输入输出
   - 输入：基因组snp的vcf格式文件，居群树文件(可选optional)
   - 输出：D值统计，f4-ratio统计，f-branch统计，f-branch树矩阵热图
4. Dsuite优势和不足
   - Dsuite的优势是运行非常快(时间以小时计算)
   - 不足是Dsuite分析结果不包含基因流的方向
5. Dsuite 原理


# Dsuite install
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

# Dsuite分析
## 1.2. 输入文件
1. sample.snp.vcf.gz
   - 基因组尺度的snp文件，vcf格式，可用bgzip压缩
2. sets.txt
   - 包括两列，tab分隔；第一列个体名，第二列物种名/居群名。
   - 对于Dtrios模块，至少指定一个个体为外类群，外类群个体的第二列写Outgroup，可有多个。
   - 要注意第二列物种名不能包含短横杠-和句点.字符，否则Dsuite Dtrios虽然不会报错，但Dsuite Fbranch模块运行报错解析不了树文件(ERROR: The tree string could not be parsed correctly)。
3. species.newick【optional】
   - Newick格式的居群/物种树文件，物种名称与sets.txt对应。
   - 树文件必须定根在Outgroup，否则dtools.py报错树文件和franch.out的拓扑不一致。
   - 去掉支持率；否则Dsuite Fbranch模块运行报错解析不了树文件，ERROR: The tree string could not be parsed correctly。
   - 枝长最好也去掉，不去枝长也不会被Dsuite用到，还可能增加报错风险。
   - 这个文件对于Dtrios模块是可选项，但建议加上好画fbranch的图。

## Dtrios模块
`Dsuite Dtrios`用于计算所有三物种组合的D和f4-ratio统计量

1. 运行
`Dsuite Dtrios sample.snp.vcf.gz sets.txt -t species.newick -o sample`

2. 常用参数
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

### 计算f-branch值

Dsuite Fbranch
`Dsuite Fbranch species.newick sample_tree.txt >fbranch.out`

需要Dsuite Dtrios并-t指定树拓扑得到的结果文件sample_tree.txt作为输入

生成fbranch.out文本文件

`Dsuite/utils/dtools.py fbranch.out species.newick --outgroup Outgroup --use_distances --dpi 1200 --tree-label-size 30`
画Fbranch的图，得到fbranch.svg和fbranch.png；--dpi设置png分辨率，--outgroup设置外类群，可以在fbranch.out里看外类群名称，--tree-label-size设置树标签大小。
如果运行log出现Plotting fbranch... Saving plots则可以看到图片生成，即使报错Segmentation fault (core dumped)也没关系。

# 2. reference
1. [Dsuite github](https://github.com/millanek/Dsuite)
2. [Dsuite paper](https://onlinelibrary.wiley.com/doi/10.1111/1755-0998.13265)
