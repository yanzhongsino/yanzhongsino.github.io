---
title: 基因富集分析(gene set enrichment analysis, GSEA)
date: 2021-11-12 16:00:00
categories: 
- bio
- bioinfo
tags: 
- gene set enrichment analysis
- GSEA
- KOBAS-i
- topGO
description: 介绍了基因富集分析和分析软件，包括KOBAS-i，topGO。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=108151&auto=1&height=32"></iframe></div>

# 1. 基因富集分析(gene set enrichment analysis, GSEA)
## 1.1. 定义
基因富集分析(gene set enrichment analysis, GSEA)，或者功能富集分析(functional enrichment analysis)是对大量的序列(基因或者蛋白质)进行按照基因功能或通路等分组，并鉴定哪些组包含比背景比例更多的基因(过度表达，over-represented)或更少的基因(表达不足，under-represented)的一种统计学方法。
## 1.2. 方法
1. 通过将输入基因集和先验基因集(比如GO,KEGG)中的每个类别进行比较，来鉴定先验基因集中的哪一类基因集富含输入基因，即哪一类基因被富集。
2. 通常有三个步骤
- 计算富集分数(enrichment score,ES)
- 估计ES的统计显著性
- 调整多重假设检验，归一化ES，计算错误发现率。
## 1.3. 应用
通常会使用其他分析的结果作为输入基因集，比如比较基因组分析中得到的某物种特有的基因集，比较转录组分析得到的差异表达基因集，基因家族收缩扩张分析得到的基因组中扩张的基因集，基因组共线性分析中在全基因组复制事件附近的Ks值的基因集等各种分析得到的基因集，做基因富集分析来查看这些基因集是否主要集中在某些类别，这些类别代表的功能是否与表型或者进化事件有关联。

# 2. 基因富集分析常用基因集
## 2.1. Gene Ontology(GO)
### 2.1.1. GO
GO是一个国际标准化的基因功能分类体系，由基因本体联合会(Gene Ontology Consortium，GOC) 负责。它提供了一套动态并可控的词汇表（controlled vocabulary）来全面描述生物体中基因和基因产物的属性，它由一组预先定义好的GO术语（GO term）组成，这组术语对基因产物的功能进行限定和描述。

GO由三个ontology（本体）组成，是由独立的术语表示的，分别描述基因的细胞组分（cellular component，CC）、分子功能（molecular function，MF）、参与的生物过程（biological process，BP）。

GO这三个本体的含义：
1. 细胞组成（cellular component，CC）：描述基因产物执行功能的细胞结构相关的位置，比如一个蛋白可能定位在细胞核中，也可能定位在核糖体中；
2. 分子功能（Molecular Function，MF）：描述基因产物发生在分子水平上的活动，例如催化或运输。通常对应于单个基因产物（即蛋白质或RNA）可以进行的活动。常见的宽泛的分子功能描述是催化活性(catalytic activity)和转运活动(transporter activity)。为了避免与基因产物名称混淆，通常分子功能描述后加上"activity"一词。
3. 生物过程（biological process，BP）：描述的是指基因产物所关联的一个大的生物功能，或者说是多个分子活动完成的一个大的生物程序。例如有丝分裂或嘌呤代谢；

### 2.1.2. GO terms
GO terms，它提供生物过程的逻辑结构与相关关系，不同GO terms之间的关系可以通过一个有向无环图来表示。

此处需要注意的是，GO terms是对**基因产物**，而不是基因本身进行描述，因为基因本身的产物有时候不止一种。GO数据库中的GO分类相关信息会得到不断地更新与增加，这个特点要记住，因为不同的GO分析工具使用的数据库版本有可能不一样，造成GO分析结果出现不同。

### 2.1.3. GO annotations
GO注释（GO annotations）是关于特定基因功能的声明，它主要是将GO terms和基因或基因产物相关联来提供注释，也就是描述这个GO terms关联的基因产物是什么（蛋白质，还是非编码RNA，还是大分子等），有什么功能，如何在分子水平发挥作用，在细胞中的哪个位置发挥作用，以及它有助于执行哪些生物过程(途径、程序)。

## 2.2. Kyoto Encyclopedia of Genes and Genomes(KEGG)
### 2.2.1. KEGG
KEGG是处理基因组，生物通路，疾病，药物，化学物质的数据库集合，于1995年由京都大学化学研究所教授Minoru Kanehisa在当时正在进行的日本人类基因组计划下发起。

KEGG 是一种数据库资源，用于从基因组和分子级信息了解生物系统（例如细胞、生物体和生态系统）的高级功能和效用。它是生物系统的计算机表示，由基因和蛋白质（基因组信息）和化学物质（化学信息）的分子构建块组成，它们与相互作用、反应和关系网络（系统信息）的分子接线图知识相结合。它还包含疾病和药物信息（健康信息）作为对生物系统的扰动。

### 2.2.2. KEGG Database
KEGG 是一个集成的数据库资源，由如下所示的 16 个数据库组成。它们大致分为系统信息、基因组信息、化学信息和健康信息，它们通过网页的颜色编码来区分。

<table>
<caption><h4>KEGG Database</h4></caption>
<thead>
<tr>
<th>Category</th>
<th>Database</th>
<th>Content</th>
<th>Color</th>
</tr>
</thead>
</tbody>
<tr>
<td rowspan="3">Systems information</td>
<td>KEGG PATHWAY</td>
<td>KEGG pathway maps</td>
<td rowspan="3"><font color=green>kegg3</br>green</font></td>
</tr>
<tr>
<td>KEGG BRITE</td>
<td>BRITE hierarchies and tables</td>
</tr>
<tr>
<td>KEGG MODULE</td>
<td>KEGG modules and reaction modules</td>
</tr>
<tr>
<td rowspan="3">Genomic information</td>
<td>KEGG ORTHOLOGY (KO)</td>
<td>Functional orthologs</td>
<td><font color=yellow>kegg4 yellow</font></td>
</tr>
<tr>
<td>KEGG GENES</td>
<td>Genes and proteins</td>
<td rowspan="2"><font color=red>kegg1</br>red</font></td>
</tr>
<tr>
<td>KEGG GENOME</td>
<td>KEGG organisms and viruses</td>
</tr>
<tr>
<td rowspan="4">Chemical information</td>
<td>KEGG COMPOUND</td>
<td>Metabolites and other chemical substances</td>
<td rowspan="4"><font color=blue>kegg2</br>blue</font></td>
</tr>
<tr>
<td>KEGG GLYCAN</td>
<td>Glycans</td>
</tr>
<tr>
<td>KEGG REACTION</br>KEGG RCLASS</td>
<td>Biochemical reactions</br>Reaction class</td>
</tr>
<tr>
<td>KEGG ENZYME</td>
<td>Enzyme nomenclature</td>
</tr>
<tr>
<td rowspan="4">Health information</td>
<td>KEGG NETWORK</td>
<td>Disease-related network variations</td>
<td rowspan="4"><font color=purple>kegg5</br>purle</font></td>
</tr>
<tr>
<td>KEGG VARIANT</td>
<td>Human gene variants</td>
</tr>
<tr>
<td>KEGG DISEASE</td>
<td>Human diseases</td>
</tr>
<tr>
<td>KEGG DRUG</br>KEGG DGROUP</td>
<td>Drugs</br>Drug groups</td>
</tr>
</tbody>
</table>

### 2.2.3. KEGG PATHWAY Database
KEGG PATHWAY Database是KEGG资源的核心，是一组手工绘制的KEGG通路图，代表细胞和生物体的新陈代谢和各种其他功能的实验知识。每个通路图都包含一个分子相互作用和反应网络，旨在将基因组中的基因与通路中的基因产物（主要是蛋白质）联系起来。

# 3. 基因富集分析程序
目前有许多程序可以用于基因富集分析。包括NASQAR,PlantRegMap,MSigDB,Broad Institute,WebGestalt,Enrichr,GeneSCF,DAVID,Metascape,AmiGO 2,GREAT,FunRich,FuncAssociate,InterMine,ToppGene Suite,QuSAGE,Blast2GO,g:Profiler。

可以根据需要分析的物种类别和流行度选择分析平台，根据需要选择先验基因集。

下面介绍几种常见的。

## 3.1. NASQAR
NASQAR (Nucleic Acid SeQuence Analysis Resource)是一个开源的网页平台，可以用R包clusterProfiler做GSEA分析，支持[Org.Db数据库](http://bioconductor.org/packages/release/BiocViews.html#___OrgDb)的所有物种的GO Term和KEGG Pathway富集分析。
## 3.2. PlantRegMap
支持165种植物的GO注释和GO富集分析。
## 3.3. Enrichr
Enrichr是针对哺乳动物的基因富集分析工具。可以通过API使用，并提供可视化结果。
## 3.4. GeneSCF
GeneSCF支持多个物种，实时的功能富集分析工具。不需要额外更新数据库，GeneSCF是实时最新的数据库，并且支持多物种富集分析。结果以文本呈现。
## 3.5. DAVID
DAVID是做注释，富集和可视化的整合的数据库。但注释数据库自从2016年10月就没有更新。
## 3.6. Blast2GO
Blast2GO可以做组学数据的功能注释和GSEA分析。
## 3.7. KOBAS-i
### 3.7.1. KOBAS-i简介
1. KOBAS是中科院和北大联合开发的做功能注释和GSEA分析的工具。
2. 目前已经发表了第三个版本，KOBAS-intelligence(KOBAS-i)，KOBAS-i引入了之前发布的基于机器学习的新方法CGPS，并扩展了可视化功能，支持的物种增加到5944个。
3. KOBAS包含注释模块和富集模块。
- 注释模块接受基因列表作为输入，包括 ID 或序列，并根据通路、疾病和GO信息等多个数据库为每个基因生成注释。
- 富集模块给出了关于哪些通路和 GO 术语与输入基因列表或表达在统计上显着相关的结果。有两种不同的富集分析可用，命名为基因列表富集和 exp-data 富集。
4. KOBAS-i有网页版，也有本地版。

### 3.7.2. KOBAS-i网页版
给了两个可用的网址：http://kobas.cbi.pku.edu.cn/，http://bioinfo.org/kobas。
[KOBAS-i](http://kobas.cbi.pku.edu.cn/)。

#### 3.7.2.1. 注释(annotation)和富集(Enrichment)的步骤
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

# 4. 富集分析的R包
常见的有topGO，clusterProfiler，有许多进行富集分析的程序使用了这些包，所以建议直接用富集程序。

## 4.1. topGO
topGO是一个R包，用于半自动的GO terms的基因富集分析。

GO term分为三大类：cellular component(CC)-细胞成分（其中基因产物位于细胞内部）,molecular function(MF)-分子功能（基因产物的功能是什么）和biology process(BP)-生物过程（即基因产物参与的一系列事件）。
三类都可以用topGO做富集分析。

### 4.1.1. 统计检验方法
topGO包默认算法用的是weight01，是elim和权重算法的混合。

| topGO支持的统计方法 | fisher | ks  | t   | globaltest | sum |
| ------------------- | ------ | --- | --- | ---------- | --- |
| classic             | Y      | Y   | Y   | Y          | Y   |
| elim                | Y      | Y   | Y   | Y          | Y   |
| weight              | Y      | N   | N   | N          | N   |
| weight01            | Y      | Y   | Y   | Y          | Y   |
| lea                 | Y      | Y   | Y   | Y          | Y   |
| parentchild         | Y      | N   | N   | N          | N   |

### 4.1.2. 安装topGO

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

### 4.1.3. topGO做基因富集分析(GSEA)
#### 4.1.3.1. 输入文件
两个输入文件
- genes.list：需要做富集分析的geneID的list，一个基因ID一行
- sample.anno：基因及GO注释信息，第一列是geneID，第二列是GO注释，空格分隔，GO注释可以有多个，格式为GO:0000428,GO:0003677,GO:0005506,

#### 4.1.3.2. GSEA分析
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

#### 4.1.3.3. 结果解释
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

# 5. references
[GSEA wiki](https://en.wikipedia.org/wiki/Gene_set_enrichment_analysis)
[topGO tutorial](https://bioconductor.org/packages/release/bioc/vignettes/topGO/inst/doc/topGO.pdf)
[topGO blog](https://datacatz.wordpress.com/2018/01/19/gene-set-enrichment-analysis-with-topgo-part-1/)
[R topGO](https://www.codenong.com/cs105162324/)
[enrichment](https://www.jianshu.com/p/47b5ea646932)
[GO explanation](https://www.jianshu.com/p/7177c372243f)
[GO overview](http://geneontology.org/docs/ontology-documentation/)
[KEGG](https://en.wikipedia.org/wiki/KEGG)