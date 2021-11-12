---
title: 基因富集分析(gene set enrichment analysis, GSEA)
date: 2021-11-12 16:00:00
categories: 
- bio
- bioinfo
tags: 
gene set enrichment analysis
GSEA
KOBAS-i
topGO
description: 介绍了基因富集分析和分析软件，包括KOBAS-i，topGO。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=3986241&auto=1&height=32"></iframe></div>

# 1. 基因富集分析(gene set enrichment analysis, GSEA)
1. 定义
- 基因富集分析(gene set enrichment analysis, GSEA)，或者功能富集分析(functional enrichment analysis)是对大量的序列(基因或者蛋白质)进行按照基因功能或通路等分组，并鉴定哪些组包含比预期更多的基因(基因富集)的一种统计学方法。
2. 方法
- 通过将输入基因集和先验基因集(比如GO,KEGG)中的每个类别进行比较，来鉴定先验基因集中的哪一类基因集富含输入基因，即哪一类基因被富集。
- 通常有三个步骤，1.计算富集分数(enrichment score,ES)，2.估计ES的统计显著性，3.调整多重假设检验，归一化ES，计算错误发现率。
3. 应用
- 通常会使用其他分析的结果作为输入基因集，比如比较基因组分析中得到的某物种特有的基因集，比较转录组分析得到的差异表达基因集，基因家族收缩扩张分析得到的基因组中扩张的基因集，基因组共线性分析中在全基因组复制事件附近的Ks值的基因集等各种分析得到的基因集，做基因富集分析来查看这些基因集是否主要集中在某些类别，这些类别代表的功能是否与表型或者进化事件有关联。

# 2. 富集分析程序
目前有许多程序可以用于基因富集分析。包括NASQAR,PlantRegMap,MSigDB,Broad Institute,WebGestalt,Enrichr,GeneSCF,DAVID,Metascape,AmiGO 2,GREAT,FunRich,FuncAssociate,InterMine,ToppGene Suite,QuSAGE,Blast2GO,g:Profiler。

可以根据需要分析的物种类别和流行度选择分析平台，根据需要选择先验基因集。

下面介绍几种常见的。

## 2.1. NASQAR
NASQAR (Nucleic Acid SeQuence Analysis Resource)是一个开源的网页平台，可以用R包clusterProfiler做GSEA分析，支持[Org.Db数据库](http://bioconductor.org/packages/release/BiocViews.html#___OrgDb)的所有物种的GO Term和KEGG Pathway富集分析。
## 2.2. PlantRegMap
支持165种植物的GO注释和GO富集分析。
## 2.3. Enrichr
Enrichr是针对哺乳动物的基因富集分析工具。可以通过API使用，并提供可视化结果。
## 2.4. GeneSCF
GeneSCF支持多个物种，实时的功能富集分析工具。不需要额外更新数据库，GeneSCF是实时最新的数据库，并且支持多物种富集分析。结果以文本呈现。
## 2.5. DAVID
DAVID是做注释，富集和可视化的整合的数据库。但注释数据库自从2016年10月就没有更新。
## 2.6. Blast2GO
Blast2GO可以做组学数据的功能注释和GSEA分析。
## 2.7. KOBAS-i
### 2.7.1. KOBAS-i简介
1. KOBAS是中科院和北大联合开发的做功能注释和GSEA分析的工具。
2. 目前已经发表了第三个版本，KOBAS-intelligence(KOBAS-i)，KOBAS-i引入了之前发布的基于机器学习的新方法CGPS，并扩展了可视化功能，支持的物种增加到5944个。
3. KOBAS包含注释模块和富集模块。
- 注释模块接受基因列表作为输入，包括 ID 或序列，并根据通路、疾病和GO信息等多个数据库为每个基因生成注释。
- 富集模块给出了关于哪些通路和 GO 术语与输入基因列表或表达在统计上显着相关的结果。有两种不同的富集分析可用，命名为基因列表富集和 exp-data 富集。
4. KOBAS-i有网页版，也有本地版。

### 2.7.2. KOBAS-i网页版
给了两个可用的网址：http://kobas.cbi.pku.edu.cn/，http://bioinfo.org/kobas。
[KOBAS-i](http://kobas.cbi.pku.edu.cn/)。

#### 2.7.2.1. 注释(annotation)和富集(Enrichment)的步骤
1. 输入
注释只有三个输入项：
- 选择物种。目前支持5944个物种。
- 选择输入基因集数据类型。支持核苷酸或者蛋白质的fasta序列，blast的Tabular输出结果，Ensembl Gene ID，Entrez Gene ID，UniProtKB AC，Gene Symbol。
- 输入基因集。可以粘贴文本，也可以上传文件。

富集除了需要上面三个输入项外，还需要选择用哪写先验基因集的数据库来做富集分析。包括PATHWAY,DISEASE,GO。其中KEGG Pathway是所有物种都可以用，其他的只有部份物种可以选。
- PATHWAY。有四个选项，KEGG Pathway(K),Reactome(R),BioCyc(B),PANTHER(p)。
- DISEASE。有三个选项，OMIM(o),KEGG Disease(k),NHGRI GWAS Catalog(N)
- GO(G)。
此外还有高级选项，可以选择统计学方法和纠错方法。

然后点击Run，就可以等待运行结果了。

2. 结果
运行结束后，点击右上角的Download total terms就可以下载到结果。
富集分析还可以点击Visualization得到结果的可视化图。

# 3. 富集分析的R包
常见的有topGO，clusterProfiler，有许多进行富集分析的程序使用了这些包，所以建议直接用富集程序。

## 3.1. topGO
topGO是一个R包，用于半自动的GO terms的基因富集分析。

GO term分为三大类：cellular component(CC)-细胞成分（其中基因产物位于细胞内部）,molecular function(MF)-分子功能（基因产物的功能是什么）和biology process(BP)-生物过程（即基因产物参与的一系列事件）。
三类都可以用topGO做富集分析。

### 3.1.1. 统计检验方法
topGO包默认算法用的是weight01，是elim和权重算法的混合。

| topGO支持的统计方法 | fisher | ks  | t   | globaltest | sum |
| ------------------- | ------ | --- | --- | ---------- | --- |
| classic             | Y      | Y   | Y   | Y          | Y   |
| elim                | Y      | Y   | Y   | Y          | Y   |
| weight              | Y      | N   | N   | N          | N   |
| weight01            | Y      | Y   | Y   | Y          | Y   |
| lea                 | Y      | Y   | Y   | Y          | Y   |
| parentchild         | Y      | N   | N   | N          | N   |

### 3.1.2. 安装topGO

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

### 3.1.3. topGO做基因富集分析(GSEA)
#### 3.1.3.1. 输入文件
两个输入文件
- genes.list：需要做富集分析的geneID的list，一个基因ID一行
- sample.anno：基因及GO注释信息，第一列是geneID，第二列是GO注释，空格分隔，GO注释可以有多个，格式为GO:0000428,GO:0003677,GO:0005506,

#### 3.1.3.2. GSEA分析
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

#### 3.1.3.3. 结果解释
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
topGO有向无环图(Directed acyclic graph, DAG)能直观展示差异表达基因富集的GO节点（Term）及其层级关系，是差异表达基因GO富集分析的结果图形化展示，分支代表包含关系，从上至下所定义的功能描述范围越来越具体。在有向无环图中，箭头代表包含关系，即该节点的所有基因同样注释到其上级节点中。

一个DAG图的示例：

<img src="https://www.google.com/url?sa=i&url=https%3A%2F%2Frpubs.com%2Fthokall%2F78374&psig=AOvVaw3W8VliiD-4n8bfdOGr5U5F&ust=1636793891235000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCMCK1Ma6kvQCFQAAAAAdAAAAABAJ" width=80% height=80% title="DAG图" alt="DAG图" align=center/>

注：对每个GO节点进行富集，在图中用方框表示显著度最高的10个节点，图中还包含其各层对应关系。每个方框（或椭圆）内给出了该GO节点的内容描述和富集显著性值。不同颜色代表不同的富集显著性，颜色越深，显著性越高。

# 4. references
[GSEA wiki](https://en.wikipedia.org/wiki/Gene_set_enrichment_analysis)
[topGO tutorial](https://bioconductor.org/packages/release/bioc/vignettes/topGO/inst/doc/topGO.pdf)
[topGO blog](https://datacatz.wordpress.com/2018/01/19/gene-set-enrichment-analysis-with-topgo-part-1/)
[R topGO](https://www.codenong.com/cs105162324/)
[enrichment](https://www.jianshu.com/p/47b5ea646932)

