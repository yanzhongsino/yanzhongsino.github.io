---
title: 用R包MSCquartets评估不完全谱系分选的程度
date: 2023-03-13
categories:
- bioinfo
- phylogeny
- ILS
tags:
- R packages
- MSCquartets
- phylogeny
- ILS
- gene flow
- MSC
- simplex

description: 造成系统发育冲突的原因包括不完全谱系分选（Incomplete lineage sorting, ILS），基因渐渗、杂交、遗传重组、水平基因转移等导致基因流的进化历史。本文章介绍了用R包MSCquartets评估不完全谱系分选的程度，以及除了ILS以外的其他原因是否参与了系统冲突。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=105203&auto=1&height=32"></iframe></div>

# 1. R包MSCquartets
## 1.1. 简介
MSCquartets(Rhodes, et al. 2020)被用来进行基于枝长的假设检验，评估在不完全谱系分选（Incomplete lineage sorting, ILS）下的多物种溯祖（multispecies coalescent, MSC）模型的物种树或物种网。它首先确定所有基因树中每组四分类群的三个可能拓扑结构的数量，这些被称为四分体计数一致性系数（quartet count concordance factor, qcCF），可以通过比较生成P值来判断基因树与物种树的拟合度。所有四分类群的四分体一致性系数可以用simplex图来表示，以全面了解整个基因树数据集的不协调情况。

## 1.2. 分析步骤
1. 用MSCquartets(Rhodes, et al. 2020)的quartetTable函数计算基因树的所有四分体的计数一致性系数（qcCFs），并生成四分体计数表。
2. 用MSCquartets的quartetTableResolved函数生成只保留解决四分体的计数表。
3. 用MSCquartets的quartetTreeTestInd函数，ASTRAL构建的物种树作为指定物种树，在"T1 model"模型下进行MSC下的树型拟合假设检验。
4. 用MSCquartets的quartetTestPlot函数为每个拒绝水平的所有的四分类群子集生成一个simplex图(Allman, et al. 2021)。不同的拒绝水平包括α= 0.01、0.001、1e-04和1e-06。

## 1.3. 结果解释
要么qcCFs与MSC下的ILS在指定的统计学意义水平上是一致的，要么ILS以外的一些机制（例如：杂交，或基因复制和丢失，以及大量的基因树推断错误等）更可能解释基因树的不一致。

在simplex图中，带颜色的符号用来显示假设检验结果。当带颜色的符号处在接近图形中心点的位置，表明有大量的ILS主导；当符号更接近顶点，表明ILS的作用很小；那些被拒绝的符号应该由ILS以外的其他过程来解释。

# 2. 用MSCquartets检验ILS

1. 输入文件
- genes.tre：基因树文件，保存多棵基因树
- 物种树：可以用文本输入R`stree="(((t5,t6),t4),((t1,t2),t3));"`

2. 运行

```R
install.packages("MSCquartets") # 安装R包MSCquartets
library("MSCquartets") # 载入R包MSCquartets

gtrees=read.tree(file="genes.tre") # 读取树文件
tnames=taxonNames(gtrees) # 从gtrees中获取所有类群的名称
tnames # 查看tnames，看看类群名称，后面的quartetTable函数通过tnames选择部分感兴趣的类群进行分析。
QT=quartetTable(gtrees,tnames[6:11]) # 这一步耗时较长。用tnames选择感兴趣的类群（这里选的第6到第11个共6个类群），也可以不设置，默认从gtrees里选择所有类群。这里用的例子是名称为t1-t6的6个类群。
RQT=quartetTableResolved(QT) # 生成已解决的四分体计数表
stree="(((t5,t6),t4),((t1,t2),t3));" # 通过文本导入物种树
pTable=quartetTreeTestInd(RQT,"T1",speciestree=stree) # 执行多个独立的假设检验，在MSC下的四分体计数拟合到物种树的检验
pTable=quartetStarTestInd(pTable) # 
quartetTestPlot(pTable, "T1", alpha=.01, beta=1) # 绘制simplex图，一般在多个alpha水平下绘制，都保存。
quartetTestPlot(pTable, "T1", alpha=.001, beta=1)
quartetTestPlot(pTable, "T1", alpha=.0001, beta=1)
quartetTestPlot(pTable, "T1", alpha=.000001, beta=1) 
```

3. 结果

在simplex图中，带颜色的符号用来显示假设检验结果。当蓝色圆形符号（fail to reject tree）处在接近图形中心点的位置，表明有大量的ILS主导；当蓝色符号更接近顶点，表明ILS的作用很小。而那些代表拒绝的红色三角形符号（reject tree）应该由ILS以外的其他过程来解释。

<img src="https://oup.silverchair-cdn.com/oup/backfile/Content_public/Journal/sysbio/71/3/10.1093_sysbio_syab068/1/syab068f3.jpeg?Expires=1681716268&Signature=Et1jsJ2Y~5cswNKSeKVojLZPRna5RGqVhTVmQIBpqmTasmtVY14fXqkH4vPt2QnTsxCr5jH0ifrSPx4Qw9esHFGy1qvwDuNXRP0dRuQLwJkb6G76QReJkP-AfIz8dwcLC5pjtDf4tQ-W7ufYuIzEbZkbM0SVcajZYSGde6vDA8I9knoEsGT53fzflJ~xATaVASqnj4QSkftNjGeYIMSQY~p7aYwNDcS1DMpwQbCoxMxyqXQhdqDkjtT1yLkYtro7NRi~5pqa5KwnpIqMoIuoQvc88Kc7IDhQB~wTDbHtPmV3DvfikaGRRh4nDNA6EzJ7HyQBNOcAinizJiwaJNzmDA__&Key-Pair-Id=APKAIE5G5CRDK6RD3PGA" title="simplex图" width="100%"/>

**<p align="center">图 1. simplex图示例 图源：https://doi.org/10.1093/sysbio/syab068</p>**

# 3. R包MSCquartets常用函数
## 3.1. quartetTable
1. quartetTable函数用来计算基因树的四分体计数一致性系数（quartet count concordance factor, qcCF），并生成四分体计数表。
2. 用法：`quartetTable(trees, taxonnames = NULL, epsilon = 0, random = 0)`
3. 参数
- trees：包含un/rooted metric/topological trees的multiphylo对象，把保存了所有基因树的文件读进来就可以用在这
- taxonnames：选取trees里的部分类群进行分析，n个名字组成的向量，默认是NULL代表trees[[1]]的类群
- epsilon：被处理成非零的最小枝长，一般默认设置成0就ok。
- random：4 taxa的随机子集的数量。如果设置为0，用所有子集。这个参数也是一般默认设置成0就ok。
- 最终会生成n+4列的计数表（n是选中的类群数量）
- 耗时步骤，29个类群3000棵树用时1h。
4. 结果
- 一个(n choose 4)×(n+4)的矩阵，或者(random)×(n+4)的矩阵。编码taxonnames的四个类群子集，以及所有树的四分体种类（种类：12|34，13|24，14|23，1234）的计数。

## 3.2. quartetTableResolved
1. quartetTableResolved函数用于把四分体计数表（常由quartetTable函数生成）的未解决四分体部分转化。转化方式是要么丢弃未解决部分，要么将未解决部分均匀分配在三个已解决的计数中。
2. 用法：`quartetTableResolved(qt,omit=FALSE)`
3. 参数
- qt：quartetTable函数的从n个选中的类群中生成n+4列的计数表
- omit：设置为TRUE代表删除未解决的四分体列；设置为FALSE代表删除未解决的四分体列的同时重新三等分这些未解决的计数分配到解决计数里。默认是FALSE。
4. 结果
- 与qt类似的四分体计数表，但只包含展示被解决四分体计数的列。

## 3.3. quartetTableDominant
1. quartetTableDominant函数用于将n个分类群上已解决的四分体计数表（常由quartetTableResolved函数生成）转换为只显示占支配地位的一个四分体的计数表，并附有MSC下的内部边缘权重的最大似然估计。
2. 用法：`quartetTableDominant(rqt,bigweights="infinite")`
3. 参数
- rqt：quartetTableResolved函数生成的已解决四分体的计数表。列数为(n choose 4)x(n+3)。
- bigweights：默认为infinite，可设置成finite。表示只有一种拓扑结构的四分体的weight（internal edge length）是无限的（infinite）的值，还是有限的（finite）但很大的数值给出。如果设置成finite，，当一组4个类群的四分体计数为(m,0,0)时，那么边缘权重（edge weight）的计算方法是：主导拓扑结构的相对频率为m/(m+1)。
4. 结果
- 一个(n choose 4)×(n+1)的矩阵，包含用1,1,-1,-1为类群列编码的支配地位四分体拓扑。第(n+1)列"weight"包含MSC下的四个类群的树的四分体的中心边缘长度的最大似然估计，溯祖单位。

## 3.4. quartetTreeTest
1. quartetTreeTest函数用于测试在MSC下四分体计数拟合一棵物种树的假设检验
2. 用法：`quartetTreeTest(obs,model="T3",lambda=0,method="MLest",smallcounts="approximate",bootstraps=10^4)`
3. 参数
- obs：已解决四分体频率的3个计数向量（三个数字组成的向量）
- model：假设检验的模型，T1用于指定的一棵物种树的四分体拓扑，T3代表任何物种树四分体拓扑。
- lambda：power-divergence统计参数，0代表似然率统计，1代表卡方检验（Chi-squared）统计
- method：可选MLtest，conservative或者bootstrap
- smallcounts：当有些计数很小时，获取p值的方法，可选bootstrap或者approximate
- bootstraps：bootstrapping采样的数量
4. 其他参数细节描述：
- 这个函数实现了Mitchell等人（2019）给出的两个版本的测试，以及 参数boostrapping，当一些预期计数较小时，还有其他程序。当 拓扑结构和/或内部四分体分支长度不是由无效假设指定的，这些是 比单自由度的Chi-square测试更准确，后者在理论上是不成立的。在模型的奇异点和边界附近是不合理的。
- 如果method="MLtest"，则使用Mitchell等人（2019）第7节中描述的该名称的测试。对于T1和T3模型来说，该测试在四元组物种内部的一个小范围内是略微反保守的。边缘的四分体物种树的小范围内，测试略显反常。虽然该检验在实践中一般表现良好，但它缺乏对 在T1或T3的全部参数空间中缺乏统一的渐进保证。
- 如果method="conservative"，则使用Mitchell等人（2019）所描述的保守检验。对于模型T3，它使用1个自由度的Chi-square分布（"最不利 "的方法），而对于模型T1，它使用最小调整的Bonferroni，基于n=1e+6的模拟中预先计算的值。拒绝无效假设，但代价是第二类错误的增加。
- 如果method="bootstrap"，那么将根据四元组拓扑结构和内部边缘长度的参数估计进行参数化引导。引导样本的大小是由 bootstrap参数。
- 当一些预期的拓扑结构计数较小时，"MLest "和 "conservative"方法并不合适。参数smallcounts决定了是使用bootstrapping还是更快的其他近似方法。这两种方法都涉及对四元组拓扑结构和内部边缘长度的估计。近似法返回一个预先计算好的p值，通过用1e+6替换最大的观察计数，并进行1e+8的计算，找到了p值。用1e+6代替最大的观察数，并对模型T3进行1e+8的引导。当n足够大（至少是30），一些预期计数较小，四叉树的错误概率较小，bootstrap p值大约与T3或T1的选择以及最大的观察计数无关。
- 对于T1模型，obs的第一个条目被视为与物种树一致的基因四分体的计数。种树一致的基因四分体。
- 当样本量较小时，如少于30个基因树，应谨慎对待返回的p值。返回的bl值是一个一致的估计值，但不是凝聚单位的内部边缘长度的MLE。虽然一致，但t的MLE是有偏的。我们的一致估计值仍有偏差，但比MLE的偏差小。关于在参数空间存在边界和/或奇异性的情况下处理参数估计的偏差的更多讨论，见Mitchell等人（2019）。
5. 结果
- 结果是两个值：(p-value,bl)
- p-value是假设检验结果的p值
- bl是以溯祖为单位的内部边缘长度（internal edge length）的一致性估计，可能是Inf。

## 3.5. quartetTreeTestInd
1. quartetTreeTestInd函数用于执行多个独立的MSC下的四分体计数拟合到物种树的假设检验。它是quartetTreeTest函数的多线程版本，quartetTreeTest只做一个假设检验，quartetTreeTestInd则是对所有已解决的四分体都进行假设检验。
2. 用法：`quartetTreeTestInd(rqt,model="T3",lambda=0,method="MLest",smallcounts="approximate",bootstraps=10^4,speciestree=NULL)`
3. 参数
- rqt：已解决四分体频率的计数表（由quartetTableResolved函数或quartetStarTestInd函数生成）
- model：假设检验的模型，T1用于指定的一棵物种树的四分体拓扑，T3代表任何物种树四分体拓扑。
- lambda：power-divergence统计参数，0代表似然率统计，1代表卡方检验（Chi-squared）统计
- method：可选MLtest，conservative或者bootstrap。与quartetTreeTest函数一致。
- smallcounts：当有些计数很小时，获取p值的方法，可选bootstrap或者approximate
- bootstraps：bootstrapping采样的数量
- speciestree：物种树，Newick文本格式，用于T1 test的物种四分体。model="T1"的必选项，model="T3"则会忽略这个参数。
4. 结果
- 如果model="T3"，输出rqt相同的表，并附加一列"p_T3"。附加列代表每个四分体的p-values。
- 如果model="T1"，输出rqt相同的表，并附加两列"p_T1"和"qindex"。"p_T1"代表每个四分体的p-values。"qindex"代表与指定物种树一致的四分体的索引值，1代表物种树上的12|34，2代表13|24，3代表14|23。

## 3.6. quartetStarTestInd
1. quartetStarTestInd函数用于多个独立的MSC下的基因四分体计数拟合一个物种四分体星形树（star tree）的假设检验。它是quartetStarTest的多线程版本。这个函数假设所有四分体都是已解决的。
2. 用法：`quartetStarTestInd(rqt)`
3. 参数
- rqt：已解决的四分体计数表（由quartetTableResolved函数或quartetTreeTestInd函数生成）
4. 结果
- 与输入的rqt相同的表，附加列"p_star"。附加列代表用于判断是否在一个星形树上拟合进MSC的p-values。

## 3.7. quartetTestPlot
1. quartetTestPlot函数用于用四分体假设检验的结果（常由quartetTreeTest函数生成）绘制simplex图，simplex图上的点代表所有四分体计数向量，颜色代表拒绝或失败或特定显著水平的拒绝。
2. 用法：`quartetTestPlot(pTable,test,alpha=0.05,beta=1,cex=1)`
3. 参数
- pTable：四分体和p值表（常由quartetTreeTestInd函数，quartetStarTestInd函数或NANUQ函数生成）
- test：树的零假设使用的模型，选T1或T3
- alpha：test给出的零假设的树测试的显著性水平
- beta：零假设检测星形树（star tree）的显著性水平。beta=1代表只有beta<1和pTable的p_star列存在的测试结果才会绘制点。
- cex：绘制符号尺寸的缩放比例
4. 结果
- 不输出值，会绘制simplex图。

# 4. references
1. R包MSCquartets的manual：https://cran.r-project.org/web/packages/MSCquartets/MSCquartets.pdf
2. R包MSCquartets的paper：https://doi.org/10.1093/bioinformatics/btaa868


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=30% title="wechat_public_QRcode.png" align=center/>