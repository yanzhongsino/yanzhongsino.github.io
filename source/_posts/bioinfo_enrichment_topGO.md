---
title: 富集分析：（二）topGO
date: 2021-11-13
categories: 
- bio
- bioinfo
- enrichment
tags: 
- enrichment analysis
- over representation analysis
- ORA
- topGO
description: 介绍了富集分析的R包topGO。用topGO做富集分析中的过表达分析(ORA)。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=116744&auto=1&height=32"></iframe></div>

# 1. 富集分析(enrichment analysis)
富集分析(enrichment analysis)的概述参考[博客：富集分析概述](https://yanzhongsino.github.io/2021/11/12/bioinfo_enrichment/)。

# 2. topGO
## 2.1. topGO简介
topGO是一个R包，用于半自动的GO terms的富集分析。

因为topGO只用于GO的富集分析，且是半自动化的，推荐使用更方便的在线工具[KOBAS-i](http://kobas.cbi.pku.edu.cn/)；[KOBAS-i 备用](http://bioinfo.org/kobas)；[GOEAST](http://omicslab.genetics.ac.cn/GOEAST/index.php); 或者功能更完善的clusterProfiler包，参考[博客：clusterProfiler包](https://yanzhongsino.github.io/2021/12/13/bioinfo_enrichment_clusterProfiler/)。

## 2.2. GO term
GO term分为三大类：
- cellular component(CC)-细胞成分（其中基因产物位于细胞内部）
- molecular function(MF)-分子功能（基因产物的功能是什么）
- biology process(BP)-生物过程（即基因产物参与的一系列事件）
三类都可以用topGO做富集分析。

## 2.3. topGO支持的统计检验方法
topGO包默认算法用的是weight01，是elim和权重算法的混合。

| topGO支持的统计方法 | fisher | ks  | t   | globaltest | sum |
| ------------------- | ------ | --- | --- | ---------- | --- |
| classic             | Y      | Y   | Y   | Y          | Y   |
| elim                | Y      | Y   | Y   | Y          | Y   |
| weight              | Y      | N   | N   | N          | N   |
| weight01            | Y      | Y   | Y   | Y          | Y   |
| lea                 | Y      | Y   | Y   | Y          | Y   |
| parentchild         | Y      | N   | N   | N          | N   |

## 2.4. 安装topGO

```R
# 安装topGO软件包
if (!requireNamespace("BiocManager", quietly = TRUE))
  install.packages("BiocManager")

BiocManager::install("topGO", version = "3.14")
BiocManager::install("Rgraphviz", version = "3.8")
BiocManager::install("GO.db")
BiocManager::install("biomaRt")
```

或者

```R
source("https://bioconductor.org/biocLite.R")
biocLite("topGO")
biocLite("GO.db")
biocLite("biomaRt")
biocLite("Rgraphviz")
 
# Load the required R packages
library(topGO)
library(GO.db)
library(biomaRt)
library(Rgraphviz)
```

## 2.5. topGO做富集分析(ORA)
### 2.5.1. 输入文件
两个输入文件
- genes.list：需要做富集分析的geneID的list，一个基因ID一行
- sample.anno：基因及GO注释信息，第一列是geneID，第二列是GO注释，空格分隔，GO注释可以有多个，格式为GO:0000428,GO:0003677,GO:0005506,

### 2.5.2. 富集分析
```R
# 设置工作目录，后面读取文件什么的就可以直接读取不需要那么长的路径
setwd('D:/test_data')

# 加载包
rm(list=ls())
library(topGO)
library(Rgraphviz)

# 设置输入文件
input="genes.list"  #待分析基因名称的列表
mapfile="sample.anno"    #所有基因GO注释结果

# 开始分析
gene_id = readMappings(file = mapfile) #如果是读取其他文件格式，后面参数还需要修改
gene_names = names(gene_id)
my_genes = read.table(input)[,1]

gene_list = rep(1,length(gene_id))
names(gene_list) = names(gene_id)

gene_list[match(my_genes,names(gene_list))] = 0
top_diff_genes = function(allScore){return(allScore<0.01)}


# 1. BP 富集分析
## new() 创建一个 topGO 的对象，然后对这个对象做检验
bp_go = new("topGOdata",
                   nodeSize = 6,
                   ontology="BP",
                   allGenes = gene_list,
                   annot = annFUN.gene2GO,
                   gene2GO = gene_id,
                   geneSel=top_diff_genes)

## 做显著性检验，使用的是elim 的算法，使用 ks 的统计量。可以理解为 p 值
result_KS.elim = runTest(bp_go,
                         algorithm = "elim",
                         statistic = "ks")

## 提取基因 table
allres = GenTable(bp_go,
                  KS = result_KS.elim,
                  ranksOf = "classic",
                  topNodes = attributes(result_KS.elim)$geneData[4])

## 生成文件，后面画图都可以用这个表
write.table(allres,
            file = paste(input,".BP.xls",sep=""),
            sep="\t", quote=FALSE, col.names=TRUE, row.names=FALSE)

## 输出矢量图
pdf(paste(input,".BP.pdf",sep=""))
showSigOfNodes(bp_go,
               score(result_KS.elim),
               firstSigNodes = 10,
               useInfo = "all") #设置节点数量，10个或者20个更多都可以
dev.off()

## 输出像素图
png(paste(input,".BP.png",sep=""))
showSigOfNodes(bp_go, score(result_KS.elim), firstSigNodes = 10, useInfo = "all")
dev.off()

# 2. MF 富集分析（同理）
## 创建一个 topGO 的对象
mf_go = new("topGOdata",
                    nodeSize = 6,
                    ontology="MF",
                    allGenes = gene_list,
                    annot = annFUN.gene2GO,
                    gene2GO = gene_id,
                    geneSel=top_diff_genes)

## 做检验，使用的是elim 的算法，使用 ks 的统计量。可以理解为 p 值
result_KS.elim = runTest(mf_go,
                         algorithm = "elim",
                         statistic = "ks")

## 提取基因 table
allres = GenTable(mf_go ,
                  KS = result_KS.elim,
                  ranksOf = "classic",
                  topNodes = attributes(result_KS.elim)$geneData[4])

## 生成文件，后面画图都可以用这个表
write.table(allres,
            file = paste(input,".MF.xls",sep=""),
            sep="\t", quote=FALSE, col.names=TRUE, row.names=FALSE)

## 输出矢量图
pdf(paste(input,".MF.pdf",sep=""))
showSigOfNodes(mf_go,
               score(result_KS.elim),
               firstSigNodes = 10,
               useInfo = "all") #设置节点数量，10个或者20个更多都可以
dev.off()

## 输出像素图
png(paste(input,".MF.png",sep=""))
showSigOfNodes(mf_go, score(result_KS.elim), firstSigNodes = 10, useInfo = "all")
dev.off()


# 3. CC节点的富集分析(同理)
## 创建一个 topGO 的对象
cc_go = new("topGOdata",
                    nodeSize = 6,
                    ontology="CC",
                    allGenes = gene_list,
                    annot = annFUN.gene2GO,
                    gene2GO = gene_id,
                    geneSel=top_diff_genes)

## 做检验，使用的是elim 的算法，使用 ks 的统计量。可以理解为 p 值
result_KS.elim = runTest(cc_go,
                         algorithm = "elim",
                         statistic = "ks")

## 提取基因 table
allres = GenTable(cc_go,
                  KS = result_KS.elim,
                  ranksOf = "classic",
                  topNodes = attributes(result_KS.elim)$geneData[4])

## 生成文件，后面画图都可以用这个表
write.table(allres,
            file = paste(input,".CC.xls",sep=""),
            sep="\t", quote=FALSE, col.names=TRUE, row.names=FALSE)

## 输出矢量图
pdf(paste(input,".CC.pdf",sep=""))
showSigOfNodes(cc_go,
               score(result_KS.elim),
               firstSigNodes = 10,
               useInfo = "all") #设置节点数量，10个或者20个更多都可以
dev.off()

## 输出像素图
png(paste(input,".CC.png",sep=""))
showSigOfNodes(cc_go, score(result_KS.elim), firstSigNodes = 10, useInfo = "all")
dev.off()
```

### 2.5.3. 结果解释
1. .xls结果文件中每一列的含义
- GO.ID：富集的GO ID
- Term：GO ID的描述
- Annotated : number of genes in go.db which are annotated with the GO-term.在go.db中被注释到GO-term的基因数量。
- Significant : number of genes belonging to your input which are annotated with the GO-term. GO-term被注释到的基因中包含输入的基因的数量
- Expected : show an estimate of the number of genes a node of size Annotated would have if the significant genes were to be randomly selected from the gene universe. 对节点基因数量的预期
- KS：用KS（Kolmogorov-Smirnov）算法计算得到的p-value值。
  
  KS全称是：Kolmogorov-Smirnov，KS值是通过KS检验所得，KS检验是一种算法。统计方法如下：
  - 首先计算每个go节点对应的gene个数，
  - 如果某个节点的子节点也有gene比对上，那么父节点对应的gene个数也要加上子节点的基因数
  - 使用KS统计检验进行p值的计算，文件中的KS值，就是常说的p value，叫KS值的原因，是体现使用的KS检验方法。

2. DAG图（矢量图/像素图）
有向无环图(Directed acyclic graph, DAG)中，从上至下依次有包含关系，只有从上往下的箭头，这个箭头代表包含关系，即**有向**。只有从上往下的箭头，方向是确定的，所以不会形成闭环，即**无环**。

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/fe/Tred-G.svg/800px-Tred-G.svg.png" width=40% title="DAG图示例" alt="DAG图" align=center/>

**<p align="center">Figure 1. DAG图示例**
from [wikipedia:directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph)</p>

- 有向无环图能直观展示基因富集的GO节点（Term）及其层级关系。
- 在有向无环图中，矩形代表富集到的top10个GO Terms，颜色从黄到红，对应p值从大到小。
- 分支代表包含关系，从上至下所定义的功能描述范围越来越具体。
- 箭头代表包含关系，即该节点的所有基因同样注释到其上级节点中。

<img src="http://guangchuangyu.github.io/blog_images/Bioconductor/clusterProfiler/2016_GO_analysis_using_clusterProfiler_files/figure-markdown_strict/unnamed-chunk-7-4.png" title=" topGO结果DAG图" width="90%"/>

**<p align="center">Figure 2. topGO结果DAG图**
from [clusterProfiler blog](https://guangchuangyu.github.io/2016/01/go-analysis-using-clusterprofiler/)</p>

注：对每个GO节点进行富集，在图中用方框表示显著度最高的10个节点，图中还包含其各层对应关系。每个方框（或椭圆）内给出了该GO节点的内容描述和富集显著性值。不同颜色代表不同的富集显著性，颜色越深，显著性越高。

# 3. references
1. topGO tutorial：https://bioconductor.org/packages/release/bioc/vignettes/topGO/inst/doc/topGO.pdf
2. topGO blog：https://datacatz.wordpress.com/2018/01/19/gene-set-enrichment-analysis-with-topgo-part-1/
3. R topGO：https://www.codenong.com/cs105162324/