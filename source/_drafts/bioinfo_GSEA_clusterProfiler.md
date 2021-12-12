---
title: 基因富集分析(gene set enrichment analysis, GSEA) —— clusterProfiler
date: 2021-12-07 16:00:00
categories: 
- bio
- bioinfo
tags: 
- gene set enrichment analysis
- GSEA
- clusterProfiler
description: 介绍了基因富集分析R包clusterProfiler。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=108151&auto=1&height=32"></iframe></div>

# 1. 基因富集分析(gene set enrichment analysis, GSEA)
基因富集分析(gene set enrichment analysis, GSEA)的介绍参考[博文基因富集分析](https://yanzhongsino.github.io/2021/11/12/bioinfo_GSEA/)。

# clusterProfiler介绍
clusterProfiler是一个R包，是一个解释组学数据的通用富集工具许多基因集的功能注释和富集分析，以及富集分析结果的可视化。

2021年07月发布了clusterProfiler 4.0版本。

<img src="https://els-jbs-prod-cdn.jbs.elsevierhealth.com/cms/attachment/86a1af7b-6e9c-4c63-a02b-83fe7593f157/gr6_lrg.jpg" width=80% title="clusterProfiler function and workflow" alt="clusterProfiler function and workflow" align=center/>

# clusterProfiler支持的基因集(gene sets)或基因通路数据库
1. Gene Ontology(GO)
2. Kyoto Encyclopedia of Genes and Genomes(KEGG)
3. Disease Ontology(DO)
4. Disease Gene Network(DisGeNET)
5. Molecular Signatures Database(MSigDb)
6. wikiPathways
... ...

# clusterProfiler功能 —— 富集分析(enrichment analysis)
## 过表达分析(Over Representation Analysis, ORA)
过表达分析是用于采用超几何分布检验判断已知的生物功能或过程(例如GO/KEGG)在实验产生的基因列表(例如差异表达基因列表: differentially expressed genes, DEGs)中是否过表达(over-represented=enriched)的常用方法。

所有基因（或全基因组）作为背景总数N，被注释到已知基因集(例如GO/KEGG)某一子集的基因总数是M，差异表达的基因数量是n，差异表达基因中属于M的数量是k。
过表达分析就是用统计学的方法判断GO/KEGG的每一个子类中是否k/n显著高于M/N，显著高于就是这个子类过表达。

差异表达基因通常是前期通过比较转录组分析等手段获取的不同转录组间有表达差异的基因列表。

## 基因集富集分析(Gene Set Enrichment Analysis, GSEA)
对于差异很大的基因，可以使用ORA分析，但差异较小的基因也可能具有意义，很多相关表型差异是通过一组基因中微小但一致的变化(small but consistent changes)来表现的。

基因集富集分析汇总一个基因集中每个基因的统计数据，可以检测预先定义的基因集(例如GO/KEGG)中所有基因以一种较小但协调的(small but coordinated)方式发生变化的情况。

基因根据表型进行排序，给定先验定义的基因集(例如共享相同GO/KEGG类别的基因)，GSEA的目标是确定S的成员是随机分布在排序的基因列表L中，还是主要分布在顶部或者底部。

## leading edge分析和核心富集基因(Leading edge analysis and core enriched genes)
包（clusterProfiler,DOSE,meshes,ReactomePA）支持Leading edge分析并可以提供GSEA分析中的核心富集基因core enriched genes结果。

1. Leading edge分析可以获取：
- Tags显示对富集分数有贡献的基因的百分比
- List显示获得富集分数在列表中的位置
- Signal显示富集信号强度

# clusterProfiler安装
启动R后输入下面命令安装
```R
if (!requireNamespace("BiocManager", quiet = TRUE))
    install.packages("BiocManager")

BiocManager::install("clusterProfiler")
```

# clusterProfiler富集分析
clusterProfiler支持对许多ontology/pathway的hypergeometric test和gene set enrichment analyses，包括数据库GO,KEGG,DO,DisGeNET,MSigDb,wikiPathways。

使用支持的数据库时，需要检查分析样本是否在已有物种列表中。不在物种列表的物种数据，以及其他不在数据库列表的数据库，做分析时使用通用的富集分析(Universal enrichment analysis)模块。

分析前，先用命令`library(clusterProfiler)`载入clusterProfiler包。

## GO富集分析

先准备好需要做富集分析的基因list，保存为内容为一列数据的文本文件gene.list，数据内容可以是OrgDb支持的任意ID类型（常用的都支持，ENSEMBL，ENTREZID，GENETYPE，GO，PFAM，具体参考[ID](http://yulab-smu.top/biomedical-knowledge-mining-book/useful-utilities.html#id-convert)）。

GO分析(包括groupGO(),enrichGO(),gseGO())支持具有OrgDb数据库的生物物种。

Bioconductor自带20个物种的OrgDb数据库。
还可以通过AnnotationHub包在线获取OrgDb

### GO分类
groupGO()函数可以基于GO在指定水平范围内的分布进行基因分类。

```
genes <- read.table("gene.list",header=F)
ggo <- groupGO(gene = genes, OrgDb = org.Hs.eg.db, ont = "CC", level = 3, readable = TURE)
head(ggo,3)
                   ID       Description Count GeneRatio geneID
GO:0000003 GO:0000003      reproduction     0     0/929       
GO:0008152 GO:0008152 metabolic process     0     0/929       
GO:0001906 GO:0001906      cell killing     0     0/929
```

### GO的ORA分析
#### 输入文件
ORA分析需要一个基因列表作为输入，比如差异表达分析(DESeq2)获得的差异表达基因列表，分析的是列表中基因在GO/KEGG各个子分类单元是否被过度代表还是代表不足。

enrichGO函数做ORA分析支持许多类型的gene ID list，用keyType参数指定输入的gene ID list类型，可以在网站查看支持的ID类型。

#### bitr格式转换
如果需要，也可以用bitr功能函数实现各种ID格式的转换。
```R
data <- read.table("gene",header=FALSE) #单列基因名文件
data$V1 <- as.character(data$V1) #需要character格式，然后进行ID转化
#将SYMBOL格式转为ENSEMBL和ENTERZID格式 
test1 = bitr(data$V1, fromType="SYMBOL", toType=c("ENSEMBL", "ENTREZID"), OrgDb="org.Hs.eg.db")
head(test1,2)

    SYMBOL         ENSEMBL ENTREZID
1    AASDH ENSG00000157426   132949
2   ABCB11 ENSG00000073734     8647
```

#### 分析
```
data <- read.table("gene.list",header=F) #读取gene ID list，内容为一列ENSEMBL格式的基因ID名称
genes <- as.character(data$V1) #转换成字符格式
ego <- enrichGO(gene          = genes, # list of entrez gene id
                OrgDb         = org.At.tair.db, # 背景使用拟南芥的数据库
                keyType       = 'ENSEMBL', # 输入基因的类型
                ont           = "CC", # "BP", "MF", "CC", "ALL"。GO三个子类里选
                pAdjustMethod = "BH", # 选项包含 "holm", "hochberg", "hommel", "bonferroni", "BH", "BY", "fdr", "none"
                pvalueCutoff  = 0.01, # 富集分析的pvalue
                qvalueCutoff  = 0.05, # 富集分析显著性的qvalue
                readable      = TRUE ) # 是否mapping gene ID到 gene Name
head(ego,2)
     ONTOLOGY         ID                                                Description
GO:0002887       BP GO:0002887 negative regulation of myeloid leukocyte mediated immunity
GO:0033004       BP GO:0033004                negative regulation of mast cell activation
           GeneRatio  BgRatio      pvalue  p.adjust    qvalue                   geneID Count
GO:0002887     2/121 10/11461 0.004706555 0.7796682 0.7796682              CD300A/CD84     2
GO:0033004     2/121 10/11461 0.004706555 0.7796682 0.7796682              CD300A/CD84     2
```

#### 结果文件解释
ego结果文件中；
- ONTOLOGY：CC  BP  MF 
- GO ID: Gene Ontology数据库中唯一的标号信息
-Description ：Gene Ontology功能的描述信息
- GeneRatio：差异基因中与该Term相关的基因数与整个差异基因总数的比值
- BgRation：所有（ bg）基因中与该Term相关的基因数与所有（ bg）基因的比值
- pvalue: 富集分析统计学显著水平，一般情况下， P-value < 0.05 该功能为富集项
- p.adjust 矫正后的P-Value
- qvalue：对p值进行统计学检验的q值
- geneID：与该Term相关的基因
- Count：与该Term相关的基因数

#### 结果可视化
1. 输出结果到enrich.csv
`write.csv(summary(ego),"enrich.csv",row.names =FALSE)`
2. 可视化——点图
`dotplot(ego,title="EnrichmentGO_MF_dot")#点图，按富集的数从大到小的`
3. 可视化——条形图
`barplot(ego, showCategory=20,title="EnrichmentGO_MF")#条状图，按p从小到大排，绘制前20个Term`
4. 可视化——有向无环图
`plotGOgraph(ego)`


### GO的GSEA分析
#### GSEA的输入文件
1. GSEA分析的输入文件是一个基因排序列表，有三个要点：
- numeric vector：倍数变化或者其他类型的数字变量，比如差异表达分析里的logFC值
- named vector：每个数字倍对应的gene ID命名
- sorted vector；数字应该以降序排序
即包含两列，一列基因ID名称，一列数据，并以数据降序排序。

2. 获取输入文件的示例：

```R
d <- read.csv(your_csv_file)
## assume that 1st column is ID (no duplicated allowed)
## 2nd column is fold change
## feature 1: numeric vector
geneList <- d[,2]
## feature 2: named vector
names(geneList) <- as.character(d[,1])
## feature 3: decreasing order
geneList <- sort(geneList, decreasing = TRUE)
```

```R
> head(mydata,3)
  gene_name     female     male    logFC
1   CG32548 0.02310383 72.43205 11.61428
2   CG15892 0.02624160 57.22716 11.09063
3   CR43803 0.02474626 34.09726 10.42823
> #GSEA的基因排序列表
> FCgenelist <- mydata$logFC #numeric vector
> names(FCgenelist) <- as.character(mydata$gene_name) #named vector
> FCgenelist <- sort(FCgenelist,decreasing=T) #decreasing order
> head(FCgenelist)
 CG32548  CG15892  CR43803  CG15136   CG4983  CG13989 
11.61428 11.09063 10.42823 10.34305 10.29130 10.00569
```


```
ego2 <- gseGO(geneList     = geneList,
              OrgDb        = org.At.tair.db,
              ont          = "CC",
              minGSSize    = 100,
              maxGSSize    = 500,
              pvalueCutoff = 0.05,
              verbose      = FALSE)
```

### 非模式生物的GO分析
enrichGO函数做ORA和gseGO做GSEA分析需要[OrgDb数据库](http://bioconductor.org/packages/release/BiocViews.html#___OrgDb)中已有物种作为背景注释，目前只有20个，植物只有拟南芥org.At.tair.db一个数据库。如果OrgDb数据库中没有待分析的物种，可以用其他来源的GO注释(比如biomaRt或者Blast2GO)，然后可以使用enricher()或GSEA()函数对非模式物种进行GO分析。

### GO分析结果画有向无环图(directed acyclic graph, DAG)
goplot()函数可以用enrichGO()的结果画DAG图。

运行`goplot(ego)`就可以出图。

## KEGG富集分析
### KEGG支持的物种
clusterProfiler包支持KEGG数据库提供KEGG注释的所有生物，可以通过[org_list网站](https://www.genome.jp/kegg/catalog/org_list.html)查看支持的物种列表，或者用clusterProfiler包提供search_kegg_organism()函数来帮助搜索支持的生物
。

```
search_kegg_organism('osa', by='kegg_code') #查询kegg_code为osa的记录，水稻
search_kegg_organism('Escherichia coli', by='scientific_name') #查询学名为Escherichia coli的记录
```

### KEGG pathway的ORA分析

```
genes <- read.table("gene.list",header=F)

kk <- enrichKEGG(gene         = genes,
                 organism     = 'osa',
                 pvalueCutoff = 0.05)
head(kk)
```
输入的ID类型可以是kegg或ncbi-geneid或ncbi-proteinid或者uniprot；
不像enrichGO()函数，enrichKEGG()函数没有readable()参数，当待分析物种在OrgDb数据库中可用时，可以用setReadable()函数把gene ID转换成gene Name。

### KEGG pathway的GSEA分析

```
kk2 <- gseKEGG(geneList     = geneList,
               organism     = 'osa',
               minGSSize    = 120,
               pvalueCutoff = 0.05,
               verbose      = FALSE)
head(kk2)
```

### KEGG module的ORA分析

```
mkk <- enrichMKEGG(gene = gene,
                   organism = 'hsa',
                   pvalueCutoff = 1,
                   qvalueCutoff = 1)
head(mkk)                   
```

### KEGG module的GSEA分析

```
mkk2 <- gseMKEGG(geneList = geneList,
                 organism = 'hsa',
                 pvalueCutoff = 1)
head(mkk2)
```

### KEGG pathways富集结果的可视化
使用browseKEGG()函数来可视化富集分析中被证明富集的基因的KEGG通路，用hsa04110基因举例
`browseKEGG(kk, 'hsa04110')`

```
library("pathview")
hsa04110 <- pathview(gene.data  = geneList,
                     pathway.id = "hsa04110",
                     species    = "hsa",
                     limit      = list(gene=max(abs(geneList)), cpd=1))
```


## 通用的富集分析(Universal enrichment analysis)
clusterProfiler支持对许多ontology/pathway的hypergeometric test和gene set enrichment analyses，但是还有很多用户想分析他们自己的数据，包括不支持的物种、不支持的ontologies/pathways或自定义注释等。clusterProfiler提供了用于hypergeometric test的enricher函数和用于基因集富集分析的GSEA函数，用于接受用户定义的注释。它们接受另外两个参数TERM2GENE和TERM2NAME。从参数名可以看出，TERM2GENE是一个data.frame，第一列为term ID，第二列为对应映射基因；TERM2NAME是一个data.frame，第一列为term ID，第二列为对应term name。TERM2NAME是可选的。

# 功能富集结果可视化



# 5. references
[GSEA wiki](https://en.wikipedia.org/wiki/Gene_set_enrichment_analysis)
[enrichment](https://www.jianshu.com/p/47b5ea646932)
[clusterProfiler github](https://github.com/YuLab-SMU/clusterProfiler)
[universal enrichment analysis using clusterProfiler](http://yulab-smu.top/biomedical-knowledge-mining-book/universal-api.html)
[clusterProfiler paper](https://www.cell.com/the-innovation/fulltext/S2666-6758(21)00066-7?_returnURL=https%3A%2F%2Flinkinghub.elsevier.com%2Fretrieve%2Fpii%2FS2666675821000667%3Fshowall%3Dtrue)
