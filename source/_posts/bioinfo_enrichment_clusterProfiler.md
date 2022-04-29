---
title: 富集分析 —— clusterProfiler
date: 2021-12-13
categories: 
- bio
- bioinfo
- enrichment
tags: 
- enrichment analysis
- over representation analysis
- ORA
- gene set enrichment analysis
- GSEA
- clusterProfiler
description: 介绍了基因富集分析R包clusterProfiler。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1336867002&auto=1&height=32"></iframe></div>

clusterProfiler相关的博客共有三篇，共同食用，效果更好 :wink: ：
- 博客[富集分析 —— clusterProfiler](https://yanzhongsino.github.io/2021/12/13/bioinfo_enrichment_clusterProfiler/)
- 博客[富集分析 —— clusterProfiler：不同物种的GO+KEGG富集分析](https://yanzhongsino.github.io/2022/04/26/bioinfo_enrichment_clusterProfiler.species/)
- 博客[富集分析 —— clusterProfiler：Visualization](https://yanzhongsino.github.io/2022/04/28/bioinfo_enrichment_clusterProfiler.visualization/)

# 1. 基因富集分析(gene set enrichment analysis, GSEA)
富集分析概述参考[博客：富集分析概述](https://yanzhongsino.github.io/2021/11/12/bioinfo_enrichment/)。

# 2. clusterProfiler介绍
clusterProfiler是一个R包，是一个解释组学数据的通用富集工具许多基因集的功能注释和富集分析，以及富集分析结果的可视化。

2021年07月发布了clusterProfiler 4.0版本。

<img src="https://yulab-smu.top/biomedical-knowledge-mining-book/figures/clusterProfiler-diagram.png" width=100% title="clusterProfiler function and workflow" alt="clusterProfiler function and workflow" align=center/>

**<p align="center">Figure 1. clusterProfiler function and workflow**
from [yulab's book](https://yulab-smu.top/biomedical-knowledge-mining-book/)</p>

# 3. clusterProfiler支持的基因集(gene sets)或基因通路数据库
1. Gene Ontology(GO)
2. Kyoto Encyclopedia of Genes and Genomes(KEGG)
3. Disease Ontology(DO)
4. Disease Gene Network(DisGeNET)
5. Molecular Signatures Database(MSigDb)
6. wikiPathways
... ...

# 4. clusterProfiler功能 —— 富集分析(enrichment analysis)
## 4.1. 过表达分析(Over Representation Analysis, ORA)
过表达分析是用于采用超几何分布检验判断已知的生物功能或过程(例如GO/KEGG)在实验产生的基因列表(例如差异表达基因列表: differentially expressed genes, DEGs)中是否过表达(over-represented=enriched)的常用方法。

所有基因（或全基因组）作为背景总数N，被注释到已知基因集(例如GO/KEGG)某一子集的基因总数是M，差异表达的基因数量是n，差异表达基因中属于M的数量是k。
过表达分析就是用统计学的方法判断GO/KEGG的每一个子类中是否k/n显著高于M/N，显著高于就是这个子类过表达。

差异表达基因通常是前期通过比较转录组分析等手段获取的不同转录组间有表达差异的基因列表。

## 4.2. 基因集富集分析(Gene Set Enrichment Analysis, GSEA)
对于差异很大的基因，可以使用ORA分析，但差异较小的基因也可能具有意义，很多相关表型差异是通过一组基因中微小但一致的变化(small but consistent changes)来表现的。

基因集富集分析汇总一个基因集中每个基因的统计数据，可以检测预先定义的基因集(例如GO/KEGG)中所有基因以一种较小但协调的(small but coordinated)方式发生变化的情况。

基因根据表型进行排序，给定先验定义的基因集(例如共享相同GO/KEGG类别的基因)，GSEA的目标是确定S的成员是随机分布在排序的基因列表L中，还是主要分布在顶部或者底部。

## 4.3. leading edge分析和核心富集基因(Leading edge analysis and core enriched genes)
包（clusterProfiler,DOSE,meshes,ReactomePA）支持Leading edge分析并可以提供GSEA分析中的核心富集基因core enriched genes结果。

1. Leading edge分析可以获取：
- Tags显示对富集分数有贡献的基因的百分比
- List显示获得富集分数在列表中的位置
- Signal显示富集信号强度

# 5. clusterProfiler安装
启动R后输入下面命令安装
```R
if (!requireNamespace("BiocManager", quiet = TRUE))
    install.packages("BiocManager")

BiocManager::install("clusterProfiler")
```

# 6. clusterProfiler的GO和KEGG富集分析
clusterProfiler支持对许多ontology/pathway的hypergeometric test和gene set enrichment analyses，包括数据库GO,KEGG,DO,DisGeNET,MSigDb,wikiPathways。

使用支持的数据库时，需要检查分析样本是否在已有物种列表中。不在物种列表的物种数据，以及其他不在数据库列表的数据库，做分析时使用通用的富集分析(Universal enrichment analysis)模块。

分析前，先用命令`library(clusterProfiler)`载入clusterProfiler包。

## 6.1. 背景数据库选择
参考博客[基因富集分析(gene set enrichment analysis, GSEA) —— clusterProfiler：不同物种的GO+KEGG富集分析](https://yanzhongsino.github.io/2022/04/26/bioinfo_GSEA_clusterProfiler.species/)。

## 6.2. GO富集分析
### 6.2.1. GO的ORA分析
#### 6.2.1.1. 输入文件
ORA分析的输入文件是gene ID list，比如差异表达分析(DESeq2)获得的差异表达基因列表，保存为内容为一列数据的文本文件gene.list，数据内容可以是OrgDb支持的任意ID类型（常用的都支持，ENSEMBL，ENTREZID，GENETYPE，GO，PFAM），具体参考[ID](http://yulab-smu.top/biomedical-knowledge-mining-book/useful-utilities.html#id-convert)。
分析的是列表中基因在GO/KEGG各个子分类单元是否被过度代表还是代表不足。

#### 6.2.1.2. bitr格式转换
如果需要，也可以用bitr功能函数实现各种ID格式的转换。
```R
data <- read.table("gene.list",header=FALSE) #单列基因名文件
data$V1 <- as.character(data$V1) #需要character格式，然后进行ID转化
test = bitr(data$V1, fromType="SYMBOL", toType=c("ENSEMBL", "ENTREZID"), OrgDb= org.At.tair.db) #将SYMBOL格式转为ENSEMBL和ENTERZID格式 
head(test,2)

    SYMBOL         ENSEMBL ENTREZID
1    AASDH ENSG00000157426   132949
2   ABCB11 ENSG00000073734     8647
```

#### 6.2.1.3. GO分类
groupGO()函数可以基于GO在指定水平范围内的分布进行基因分类。

```R
data <- read.table("gene.list",header=F) #读取gene ID list，内容为一列ENSEMBL格式的基因ID名称
genes <- as.character(data$V1) #转换成字符格式
ggo <- groupGO(gene = genes, OrgDb = org.Hs.eg.db, ont = "CC", level = 3, readable = TURE)
head(ggo,3)
                   ID       Description Count GeneRatio geneID
GO:0000003 GO:0000003      reproduction     0     0/929       
GO:0008152 GO:0008152 metabolic process     0     0/929       
GO:0001906 GO:0001906      cell killing     0     0/929
```

#### 6.2.1.4. ORA分析
1. 读取输入文件
```R
data <- read.table("gene.list",header=F) #读取gene ID list，内容为一列ENSEMBL格式的基因ID名称
genes <- as.character(data$V1) #转换成字符格式
```
2. 支持物种的标准分析 enrichGO()

```R
ego <- enrichGO(gene          = genes, # list of entrez gene id
                OrgDb         = org.At.tair.db, # 背景使用分析物种的org包，这里示例使用拟南芥的数据库
                keyType       = 'ENSEMBL', # 输入基因的类型，命令keytypes(org.Hs.eg.db)会列出可用的所有类型；
                ont           = "ALL", # "BP", "MF", "CC", "ALL"。GO三个子类。如果选ALL，会同时获得三个子类的结果(相当于单独做三个子类的结果合并在一起)，结果中增加一列ONTOLOGY为子类。
                pAdjustMethod = "BH", # 指定多重假设检验矫正的方法，选项包含 "holm", "hochberg", "hommel", "bonferroni", "BH", "BY", "fdr", "none"
                pvalueCutoff  = 0.05, # 富集分析的pvalue，默认是pvalueCutoff = 0.05，更严格可选择0.01
                qvalueCutoff  = 0.2, # 富集分析显著性的qvalue，默认是qvalueCutoff = 0.2，更严格可选择0.05
                readable      = TRUE ) # 是否将gene ID转换到 gene symbol
head(ego,2)
     ONTOLOGY         ID                                                Description
GO:0002887       BP GO:0002887 negative regulation of myeloid leukocyte mediated immunity
GO:0033004       BP GO:0033004                negative regulation of mast cell activation
           GeneRatio  BgRatio      pvalue  p.adjust    qvalue                   geneID Count
GO:0002887     2/121 10/11461 0.004706555 0.7796682 0.7796682              CD300A/CD84     2
GO:0033004     2/121 10/11461 0.004706555 0.7796682 0.7796682              CD300A/CD84     2
```

3. 提供注释文件go_annotation.txt的非模式物种分析 enricher()
```R
go_anno <- read.table("go_annotation.txt",header = T,sep = "\t")
go2gene <- go_anno[, c(2, 1)]
go2name <- go_anno[, c(2, 3)]
ego <- enricher(genes, TERM2GENE = go2gene, TERM2NAME = go2name, pAdjustMethod = "BH",pvalueCutoff  = 0.05, qvalueCutoff  = 0.2) 
```

4. ego结果文件解释
- ONTOLOGY：CC  BP  MF 
- ID: Gene Ontology数据库中唯一的标号信息
- Description ：Gene Ontology功能的描述信息
- GeneRatio：差异基因中与该Term相关的基因数与整个差异基因总数的比值
- BgRatio：所有（ bg）基因中与该Term相关的基因数与所有（ bg）基因的比值
- pvalue: 富集分析统计学显著水平，一般情况下， P-value < 0.05 该功能为富集项
- p.adjust 矫正后的P-Value
- qvalue：对p值进行统计学检验的q值
- geneID：与该Term相关的基因
- Count：与该Term相关的基因数

5. 输出结果保存为csv表格
```R
write.table(as.data.frame(ego),"go_enrich.csv",sep="\t",row.names =F,quote=F) #保存到文件go_enrich.csv。其中as.data.frame(ego)把ego对象转换成数据框dataframe
```

#### 6.2.1.5. 结果整理和筛选
1. 如果没使用keyType参数，可以在得到结果后使用
2. 如果没使用readable=True参数，可以在得到结果后用函数setReadable()将GeneID转换为symbol。`ego <- setReadable(ego, OrgDb = org.At.tair.db)`
3. 函数dropGO可以移除enrichGO结果中特定的GO term或GO level
4. 函数gofilter()可以将结果限定在特定的GO level
5. 函数simplify可以去除冗余，`ego_rm <- simplify(ego, cutoff=0.7, by="p.adjust", select_fun=min)`

### 6.2.2. GO的GSEA分析
GSEA分析通过置换检验来计算p值
#### 6.2.2.1. GSEA的输入文件
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
> genelist <- mydata$logFC #numeric vector
> names(genelist) <- as.character(mydata$gene_name) #named vector
> genelist <- sort(genelist,decreasing=T) #decreasing order
> head(genelist)
 CG32548  CG15892  CR43803  CG15136   CG4983  CG13989 
11.61428 11.09063 10.42823 10.34305 10.29130 10.00569
```

#### 6.2.2.2. GSEA分析
1. 读取输入文件
```R
data <- read.csv("geneList.csv")
geneList <- d[,2]
names(geneList) <- as.character(d[,1])
geneList <- sort(geneList, decreasing = TRUE)
```

2. 支持物种的标准分析 gseGO()
```R
gsego <- gseGO(geneList     = geneList,
              OrgDb        = org.At.tair.db,
              ont           = "ALL", # "BP", "MF", "CC", "ALL"。GO三个子类。如果选ALL，会同时获得三个子类的结果(相当于单独做三个子类的结果合并在一起)，结果中增加一列ONTOLOGY为子类。
              keyType      = "SYMBOL",  # 输入基因的类型，命令keytypes(org.Hs.eg.db)会列出可用的所有类型；
              nPerm        = 1000, # permutation numbers 置换次数
              minGSSize    = 100, # 富集基因数量的下限
              maxGSSize    = 500, # 富集基因数量的上限
              pvalueCutoff = 0.05,
              verbose      = FALSE, # 不输出结果
              by           = "fgsea") # 选项fgsea或DOSE

head(gsego,1);dim(gsego)
                   ID        Description setSize enrichmentScore
GO:0003674 GO:0003674 molecular_function     323       0.9462411
                NES      pvalue    p.adjust qvalues rank
GO:0003674 1.716837 0.000999001 0.000999001      NA  579
                              leading_edge
GO:0003674 tags=100%, list=11%, signal=95%
core_enrichment
GO:0003674 CG15892/CG15136/CG4983/CG43851/…
[1] 53 11

head(data.frame(gsego$ID,gsego$Description))
   gsego.ID gsego.Description
1 GO:0003674 molecular_function
2 GO:0005488            binding
```

3. 提供注释文件go_annotation.txt的非模式物种分析 GSEA()
```R
data <- read.table("go_annotation.txt",header = T,sep = "\t",quote="")
go2gene <- data[, c(2, 1)]
go2name <- data[, c(2, 3)]
gsego <- GSEA(genes, TERM2GENE = go2gene, TERM2NAME = go2name)
```

4. gsego结果文件解释
- ID: Gene Ontology数据库中唯一的标号信息
- Description：Gene Ontology功能的描述信息
- setSize
- enrichmentScore：富集分数
- NES
- pvalue: 富集分析统计学显著水平，一般情况下， P-value < 0.05 该功能为富集项
- p.adjust：矫正后的P-Value
- qvalues：对p值进行统计学检验的q值
- rank
- leading_edge
- core_enrichment

### 6.2.3. GO富集结果可视化
`goplot(ego)`简单可视化结果为有向无环图。

## 6.3. KEGG富集分析
clusterProfiler通过[KEGG数据库的API](https://www.kegg.jp/kegg/rest/keggapi.html)来获取KEGG的注释信息，包括一个物种所有基因对应的pathway注释文件，比如人的：http://rest.kegg.jp/link/hsa/pathway；和pathway对应的描述信息，比如人的：http://rest.kegg.jp/list/pathway/hsa。

### 6.3.1. 支持的物种
1. clusterProfiler包支持的物种
只需要将物种缩写输入给clusterProfiler，clusterProfiler包支持自动联网调取[kegg注释物种](https://www.genome.jp/kegg/catalog/org_list.html)列出物种的pathway注释信息，网站可以查看物种列表和缩写，或者用clusterProfiler包提供search_kegg_organism()函数来帮助搜索支持的生物。

```R
search_kegg_organism('osa', by='kegg_code') #查询kegg_code为osa的记录，水稻
search_kegg_organism('Escherichia coli', by='scientific_name') #查询学名为Escherichia coli的记录
```

2. 用户提供KEGG pathway注释数据
如果分析的物种不支持自动调取，可以自己做KEGG pathway注释后提供注释文件，比如interproscan、eggnog-mapper等软件的注释结果，整理成clusterProfiler支持的输入格式即可。

clusterProfiler需要导入的KEGG pathway注释文件pathway_annotation.txt的格式如下：

| GeneID | Pathway  | Path_Description |
| ------ | -------- | ---------------- |
| 1      | ko:00001 | spindle          |
| 2      | ko:00002 | mitotic spindle  |
| 3      | ko:00003 | kinetochore      |

data.frame格式，包含三列，第一列为Gene ID，第二列为 KEGG Pathway ID，第三列为Path_Description，顺序无要求。

### 6.3.2. KEGG pathway的ORA分析
#### 6.3.2.1. KEGG输入ID的格式转换
ID转换函数

```R
library(clusterProfiler)
bitr_kegg("1",fromType = "kegg",toType = 'ncbi-proteinid',organism='hsa')

library(org.Hs.eg.db)
keytypes(org.Hs.eg.db) #查看支持的ID类型
bitr(gene, fromType = "ENTREZID", toType = c("ENSEMBL", "SYMBOL"), OrgDb = org.Hs.eg.db)

#以上看出ID转换输入时，可以向量的形式，也可以单列基因名list导入
gene <- c("AASDH","ABCB11","ADAM12","ADAMTS16","ADAMTS18")
genes  <-  gene$V1 #字符串
#也可以是内置数据
data(geneList, package="DOSE") #富集分析的背景基因集
genes <- names(geneList)[abs(geneList) > 2]
```

#### 6.3.2.2. KEGG pathway的ORA分析
输入文件与GO的ORA分析输入文件一样。
1. 导入输入文件

```R
data <- read.table("gene.list",header=F) #读取gene ID list，内容为一列ENSEMBL格式的基因ID名称
genes <- as.character(data$V1) #转换成字符格式
```

2. 支持物种的标准分析 enrichKEGG()

```R
kk <- enrichKEGG(gene         = genes,
                 keyType      = "kegg", # 输入基因的类型，命令keytypes(org.Hs.eg.db)会列出可用的所有类型；
                 organism     = 'hsa',
                 pvalueCutoff = 0.05
                 pAdjustMethod= "BH",
                 qvalueCutoff = 0.2)
head(kk,2)
              ID                                      Description GeneRatio  BgRatio
hsa04750 hsa04750 Inflammatory mediator regulation of TRP channels      5/53  97/7387
hsa04020 hsa04020                        Calcium signaling pathway      6/53 182/7387
               pvalue   p.adjust    qvalue                              geneID Count
hsa04750 0.0006135305 0.08589427 0.0807277             40/3556/3708/5608/79054     5
hsa04020 0.0018078040 0.12654628 0.1189345        493/1129/2066/3707/3708/4842     6
```

输入的ID类型默认是kegg gene id，也可以是ncbi-geneid或ncbi-proteinid或者uniprot等，可以通过上一步格式转换转换ID类型。
organism对应物种的三字母缩写，其他参数与GO的ORA分析参数一致。

不像enrichGO()函数有readable()参数，enrichKEGG()函数没有这个参数，当待分析物种在OrgDb数据库中可用时，可以用setReadable()函数把gene ID转换成gene Name。

3. 提供注释文件pathway_annotation.txt的非模式物种分析 enricher()

```R
pathway_anno <- read.table("pathway_annotation.txt",header = T,sep = "\t")
go2gene <- pathway_anno[, c(2, 1)]
go2name <- pathway_anno[, c(2, 3)]
kk <- enricher(gene, TERM2GENE = go2gene,TERM2NAME = go2name)
```

4. 结果输出到csv文件
`write.table(as.data.frame(kk),"KEGG_enrich.csv",sep="\t",row.names =F,quote=F)` #保存到文件KEGG_enrich.csv。其中as.data.frame(kk)把kk对象转换成数据框dataframe

### 6.3.3. KEGG pathway的GSEA分析

1. 读取输入文件
```R
data <- read.csv("geneList.csv")
geneList <- d[,2]
names(geneList) <- as.character(d[,1])
geneList <- sort(geneList, decreasing = TRUE)
```

2. 支持物种的标准分析 gseKEGG()

```R
kks <- gseKEGG(geneList     = geneList,
               keyType      = "kegg", # 输入基因的类型，命令keytypes(org.Hs.eg.db)会列出可用的所有类型；
               organism     = 'hsa',
               nPerm        = 1000,
               minGSSize    = 10,
               maxGSSize    = 500,
               pvalueCutoff = 0.05,
               pAdjustMethod= "BH",
               verbose      = FALSE)
head(kk2)
```

3. 提供注释文件pathway_annotation.txt的非模式物种分析 GSEA()
```R
data <- read.table("pathway_annotation.txt",header = T,sep = "\t")
go2gene <- data[, c(2, 1)]
go2name <- data[, c(2, 3)]
kks <- GSEA(gene, TERM2GENE = go2gene, TERM2NAME = go2name)
```

### 6.3.4. KEGG module的ORA分析
[KEGG Module](https://www.genome.jp/kegg/module.html)是手动定义的功能单元的集合。在某些情况下，KEGG 模块有更直接的解释。

```R
mkk <- enrichMKEGG(gene = gene,
                   organism = 'hsa',
                   pvalueCutoff = 0.05,
                   qvalueCutoff = 0.2)
head(mkk)
##            ID                                             Description GeneRatio
## M00912 M00912      NAD biosynthesis, tryptophan => quinolinate => NAD       2/9
## M00095 M00095          C5 isoprenoid biosynthesis, mevalonate pathway       1/9
## M00053 M00053 Pyrimidine deoxyribonuleotide biosynthesis, CDP => dCTP       1/9
## M00938 M00938 Pyrimidine deoxyribonuleotide biosynthesis, UDP => dTTP       1/9
## M00003 M00003            Gluconeogenesis, oxaloacetate => fructose-6P       1/9
## M00049 M00049     Adenine ribonucleotide biosynthesis, IMP => ADP,ATP       1/9
##        BgRatio      pvalue   p.adjust     qvalue     geneID Count
## M00912  12/829 0.006541756 0.03925053 0.03443029 23475/3620     2
## M00095  10/829 0.103949536 0.20710097 0.18166751       3158     1
## M00053  11/829 0.113796244 0.20710097 0.18166751       6241     1
## M00938  14/829 0.142761862 0.20710097 0.18166751       6241     1
## M00003  18/829 0.180072577 0.20710097 0.18166751       5105     1
## M00049  21/829 0.207100966 0.20710097 0.18166751      26289     1             
```

### 6.3.5. KEGG module的GSEA分析

```R
mkk2 <- gseMKEGG(geneList = geneList,
                 organism = 'hsa',
                 pvalueCutoff = 0.05)
head(mkk2)
##            ID                                                      Description
## M00001 M00001        Glycolysis (Embden-Meyerhof pathway), glucose => pyruvate
## M00035 M00035                                           Methionine degradation
## M00002 M00002         Glycolysis, core module involving three-carbon compounds
## M00938 M00938          Pyrimidine deoxyribonuleotide biosynthesis, UDP => dTTP
## M00104 M00104 Bile acid biosynthesis, cholesterol => cholate/chenodeoxycholate
## M00009 M00009                           Citrate cycle (TCA cycle, Krebs cycle)
##        setSize enrichmentScore       NES      pvalue  p.adjust   qvalues rank
## M00001      24       0.5739036  1.683936 0.005050296 0.1616095 0.1488508 2886
## M00035      10       0.6784636  1.637462 0.025966367 0.2710124 0.2496167 1555
## M00002      11       0.6421781  1.585841 0.032455383 0.2710124 0.2496167 1381
## M00938      10       0.6648004  1.604486 0.033876549 0.2710124 0.2496167  648
## M00104      10      -0.5876900 -1.338409 0.128747795 0.6564103 0.6045884  961
## M00009      22       0.4504911  1.315039 0.138271605 0.6564103 0.6045884 3514
##                          leading_edge
## M00001 tags=54%, list=23%, signal=42%
## M00035 tags=50%, list=12%, signal=44%
## M00002 tags=55%, list=11%, signal=49%
## M00938  tags=40%, list=5%, signal=38%
## M00104  tags=50%, list=8%, signal=46%
## M00009 tags=50%, list=28%, signal=36%
##                                                         core_enrichment
## M00001 5214/3101/2821/7167/2597/5230/2023/5223/5315/3099/5232/2027/5211
## M00035                                           875/1789/191/1788/1786
## M00002                                    7167/2597/5230/2023/5223/5315
## M00938                                              6241/7298/4830/1841
## M00104                                        6342/10998/1581/3295/8309
## M00009            3418/50/4190/3419/2271/3421/55753/3417/1431/6389/4191
```

### 6.3.6. KEGG pathways富集结果的可视化
enrichplot包可以实现几种方法，可以用在GO,KEGG,MSigDb等基因集注释上。
1. browseKEGG()函数会打开网络浏览器突出显示富集的基因的KEGG通路，用hsa04110基因举例：
`browseKEGG(kk, 'hsa04110')`

2. pathview::pathview()可视化富集的KEGG通路
```
library("pathview")
hsa04110 <- pathview(gene.data  = geneList,
                     pathway.id = "hsa04110",
                     species    = "hsa",
                     limit      = list(gene=max(abs(geneList)), cpd=1))
```

## 6.4. 通用的富集分析(Universal enrichment analysis)
除GO,KEGG基因集外，clusterProfiler还支持WikiPathways，Reactome，Disease，MeSH等数据库的富集分析。

虽然clusterProfiler支持对许多ontology/pathway的hypergeometric test和gene set enrichment analyses，但是还有很多数据，包括不支持的物种、不支持的ontologies/pathways或自定义注释等。
clusterProfiler提供了用于hypergeometric test的enricher()函数和用于基因集富集分析的GSEA()函数，用于接受用户定义的注释。

另外两个参数TERM2GENE和TERM2NAME:
- TERM2GENE是一个必需的data.frame，第一列为term ID，第二列为对应映射基因；
- TERM2NAME是一个可选的data.frame，第一列为term ID，第二列为对应term name。

在GO和KEGG富集分析的用户自行提供注释文件的分析部分，使用的enricher()和GSEA()函数。


# 7. references
1. [GSEA wiki](https://en.wikipedia.org/wiki/Gene_set_enrichment_analysis)
2. [enrichment](https://www.jianshu.com/p/47b5ea646932)
3. [clusterProfiler github](https://github.com/YuLab-SMU/clusterProfiler)
4. [clusterProfiler paper](https://www.cell.com/the-innovation/fulltext/S2666-6758(21)00066-7?_returnURL=https%3A%2F%2Flinkinghub.elsevier.com%2Fretrieve%2Fpii%2FS2666675821000667%3Fshowall%3Dtrue)
5. [clusterProfiler book](http://yulab-smu.top/biomedical-knowledge-mining-book/index.html)
6. [clusterProfiler manual](https://bioconductor.org/packages/devel/bioc/manuals/clusterProfiler/man/clusterProfiler.pdf)7. 
7. [clusterProfiler ducumentation](https://guangchuangyu.github.io/software/clusterProfiler/documentation/)
8. [clusterProfiler blog](https://guangchuangyu.github.io/2016/01/go-analysis-using-clusterprofiler/)
9. [tutorial](https://www.cnblogs.com/jessepeng/p/12159139.html)
10. [函数simplify](http://guangchuangyu.github.io/2015/10/use-simplify-to-remove-redundancy-of-enriched-go-terms/)