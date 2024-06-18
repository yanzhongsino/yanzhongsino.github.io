---
title: 富集分析：（四）clusterProfiler：不同物种的GO+KEGG富集分析
date: 2022-04-26
categories: 
- bioinfo
- enrichment
tags: 
- enrichment analysis
- over representation analysis
- ORA
- gene set enrichment analysis
- GSEA
- clusterProfiler
- GO
- KEGG
- AnnotationHub
- AnnotationForge
- enricher
description: 记录使用clusterProfiler进行GO/KEGG富集分析时，根据分析的物种来选择和准备背景数据集，包括支持的模式物种的获取，非模式物种Orgdb包的构建，通用富集分析等。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283092&auto=1&height=32"></iframe></div>

clusterProfiler相关的博客共有三篇，共同食用，效果更好 :wink: ：
- 博客[富集分析：（三）clusterProfiler概述](https://yanzhongsino.github.io/2021/12/13/bioinfo_enrichment_clusterProfiler.intro/)
- 博客[富集分析：（四） clusterProfiler：不同物种的GO+KEGG富集分析](https://yanzhongsino.github.io/2022/04/26/bioinfo_enrichment_clusterProfiler.species/)
- 博客[富集分析：（五）clusterProfiler：Visualization](https://yanzhongsino.github.io/2022/04/28/bioinfo_enrichment_clusterProfiler.visualization/)

**总结**

用clusterProfiler进行GO/KEGG富集分析中的过表达分析(ORA)时，根据分析的物种来选择和准备背景数据集。
1. GO数据库过表达分析(ORA)根据物种选择背景数据集：
- 先在Bioconductor库查询是否已有OrgDb的物种(大部分是模式物种)
- 如果没有，再在AnnotationHub上查询是否有在线注释可以创建OrgDb对象
- 如果没有，就准备分析物种基因组的功能注释集，通过AnnotationForge创建OrgDb包；或者通过通用富集分析函数enricher进行富集分析

2. KEGG数据库过表达分析(ORA)根据物种选择背景数据集：
- 先在KEGG数据库查询是否已有基因组物种
- 如果没有，就准备分析物种基因组的功能注释集，通过AnnotationForge创建OrgDb包；或者通过通用富集分析函数enricher进行富集分析

# 1. GO数据库过表达分析(ORA)的背景数据集选择
GO富集分析需要物种的注释数据库作为背景，根据分析物种的具体情况，选择不同的数据库作为背景数据库。

groupGO(),enrichGO(),gseGO()函数支持具有OrgDb数据库的生物物种，enricher()和gseGO()函数支持用户自己提供的GO注释文件。

## 1.1. 已有OrgDb的物种 —— Bioconductor
1. 查询
[OrgDb数据库](http://bioconductor.org/packages/release/BiocViews.html#___OrgDb)查询，目前Bioconductor大概有20个，都是模式物种，植物只有拟南芥**org.At.tair.db**一个数据库。

2. 安装和加载
OrgDb数据库本质是一个R包，需要`BiocManager::install("org.At.tair.db")`安装和`library(org.At.tair.db)`加载后才能使用。

3. 使用
如果有的话，加载后在`groupGO(OrgDb = org.At.eg.db)`,`enrichGO(OrgDb = org.At.eg.db)`,`gseGO(OrgDb = org.At.eg.db)`等函数里直接赋值给`OrgDb`参数即可使用。

## 1.2. 获取在线注释创建OrgDb对象 —— AnnotationHub
如果物种不在Bioconductor的OrgDb列表里，可以用[AnnotationHub](http://bioconductor.org/packages/release/bioc/html/AnnotationHub.html)包在线查询和获取其他可用的物种注释信息，并制作OrgDb包使用。
AnnotationHub包连接的Bioconductor数据库是实时更新的，所以需要用到的时候再在线查询和使用。

1. 查询和制作OrgDb库
```R
BiocManager::install("AnnotationHub") #安装AnnotationHub
BiocManager::install("AnnotationDbi") #安装AnnotationDbi
BiocManager::install("rtracklayer") #安装rtracklayer
library(AnnotationHub) #加载AnnotationHub
hub <- AnnotationHub() #建立AnnotationHub对象保存到hub

# 查看AnnotationHub内容
display(hub) #会调出一个网页，以交互式表格的形式查看，但有些服务器不支持
unique(hub$species) #查看hub里包含的所有物种，目前有2700+个物种。
unique(hub$rdataclass) #查看hub里的数据类型，我们这里需要的是最后一种OrgDb类型数据
hub[hub$rdataclass == "OrgDb"] #查看hub里OrgDb类型的数据

query(hub, "Cricetulus") #查询包含Cricetulus的物种信息，一共查询到229条信息。输出如下：
------
AnnotationHub with 229 records
# snapshotDate(): 2021-10-20
# $dataprovider: Ensembl, ftp://ftp.ncbi.nlm.nih.gov/gene/DATA/, UCSC, FANTO...
# $species: cricetulus griseus crigri, cricetulus griseus chok1gshd, cricetu...
# $rdataclass: TwoBitFile, GRanges, SQLiteFile, OrgDb, Inparanoid8Db, ChainFile
# additional mcols(): taxonomyid, genome, description,
#   coordinate_1_based, maintainer, rdatadateadded, preparerclass, tags,
#   rdatapath, sourceurl, sourcetype
# retrieve records with, e.g., 'object[["AH10393"]]'

            title
  AH10393 | hom.Cricetulus_griseus.inp8.sqlite
  AH13980 | criGri1.2bit
  AH14346 | criGri1ToHg19.over.chain.gz
  AH57104 | Cricetulus_griseus_chok1gshd.CHOK1GS_HDv1.90.abinitio.gtf
  AH57105 | Cricetulus_griseus_chok1gshd.CHOK1GS_HDv1.90.gtf
  ...       ...
  AH99348 | Cricetulus_griseus_crigri.CriGri_1.0.ncrna.2bit
  AH99349 | Cricetulus_griseus_picr.CriGri-PICR.cdna.all.2bit
  AH99350 | Cricetulus_griseus_picr.CriGri-PICR.dna_rm.toplevel.2bit
  AH99351 | Cricetulus_griseus_picr.CriGri-PICR.dna_sm.toplevel.2bit
  AH99352 | Cricetulus_griseus_picr.CriGri-PICR.ncrna.2bi
-------

query(hub[hub$rdataclass == "OrgDb"] , "Cricetulus") #query(hub, "Cricetulus")的查询结果中有各种类型的数据，如果我们需要OrgDb类型，可以这样筛选后查询，也可以这样`query(hub,'org.Cricetulus')`搜索。输出如下：
-------
AnnotationHub with 2 records
# snapshotDate(): 2021-10-20
# $dataprovider: ftp://ftp.ncbi.nlm.nih.gov/gene/DATA/
# $species: Cricetulus griseus, Cricetulus barabensis_griseus
# $rdataclass: OrgDb
# additional mcols(): taxonomyid, genome, description,
#   coordinate_1_based, maintainer, rdatadateadded, preparerclass, tags,
#   rdatapath, sourceurl, sourcetype
# retrieve records with, e.g., 'object[["AH96227"]]'

            title
  AH96227 | org.Cricetulus_barabensis_griseus.eg.sqlite
  AH96228 | org.Cricetulus_griseus.eg.sqlite
-------

Cgriseus <- hub[["AH96228"]]#制作Cricetulus_griseus的OrgDb库。AH96228是Cricetulus_griseus对应的编号。下载和检索后即保存在缓存目录下（我这是/home/user/.cache/R/AnnotationHub/5f01212f7210_102705）。
Cgriseus #查看Cgriseus，输出如下：
-----
OrgDb object:
| DBSCHEMAVERSION: 2.1
| DBSCHEMA: NOSCHEMA_DB
| ORGANISM: Cricetulus barabensis_griseus
| SPECIES: Cricetulus barabensis_griseus
| CENTRALID: GID
| Taxonomy ID: 10029
| Db type: OrgDb
| Supporting package: AnnotationDbi

Please see: help('select') for usage information
-----

library(AnnotationDbi)
saveDb(Cgriseus,file="Cgriseus.OrgDb") #把Cgriseus对象保存成Cgriseus.OrgDb文件
```

2. 载入和查看
保存为文件后使用时加载Cgriseus.OrgDb文件即可使用。常用的函数包括columns()，keys()，keytypes()，select()等。
```R
library(AnnotationDbi)
Cgriseus<-loadDb(file="Cgriseus.OrgDb") #载入Cgriseus.OrgDb文件，保存到Cgriseus
length(keys(Cgriseus)) #查看包含的基因数量
columns(Cgriseus) #查看OrgDb对象的数据类型
keys(Cgriseus, keytype = "GO") #查看GO数据集下的ID
```

3. 使用
载入后在`groupGO(OrgDb = Cgriseus)`,`enrichGO(OrgDb = Cgriseus)`,`gseGO(OrgDb = Cgriseus)`等函数里直接赋值给`OrgDb`参数即可。

## 1.3. 通用富集分析（非模式物种）—— AnnotationForge
- 如果以上两个方法都没有找到你分析的物种，可以选择近缘种来代替。
- 如果有分析物种的基因组注释数据，更好的方案是使用通用富集分析。
- AnnotationForge是用来创建OrgDb包的R包。

### 1.3.1. 安装

```R
install.packages('GO.db')
library(GO.db)
BiocManager::install("AnnotationForge") #安装
library(AnnotationForge) #加载
```

### 1.3.2. 准备
- 这里使用的基因组注释数据是eggNOG注释结果emapper.annotations.tsv。
- 准备步骤之后的步骤大多在R环境里操作，有些需要在shell环境处理，建议开两个shell窗口操作。
1. 删除冗余
把emapper.annotations.tsv的##开头的行和标题行前的#删除，保存到文件eggnog.anno。
`sed '/^##/d' emapper.annotations.tsv |sed 's/#//' >eggnog.anno`

2. 读取数据
```R
egg<-read.csv("eggnog.anno",header=T,sep="\t") #如果用read.csv报错，尝试read.table("eggnog.anno",header=T,sep="\t")，或者试试egg <- rio::import('emapper.annotations.tsv')。
egg[egg==""] <- NA #把空行标记上NA

colnames(egg) #列出标题行
[1] "query"          "seed_ortholog"  "evalue"         "score"
 [5] "eggNOG_OGs"     "max_annot_lvl"  "COG_category"   "Description"
 [9] "Preferred_name" "GOs"            "EC"             "KEGG_ko"
[13] "KEGG_Pathway"   "KEGG_Module"    "KEGG_Reaction"  "KEGG_rclass"
[17] "BRITE"          "KEGG_TC"        "CAZy"           "BiGG_Reaction"
[21] "PFAMs"
```

### 1.3.3. 提取注释的GO信息
从egg中提取GO注释并整理成gene2go.txt文件
```R
library(dplyr)
gene_info <- egg %>%dplyr::select(GID = query, GENENAME = seed_ortholog) %>% na.omit() #根据egg第一和第二列的标题提取前两列。
goterms <- egg %>%dplyr::select(query, GOs) %>% na.omit() %>% filter(str_detect(GO,"GO")) #根据egg第一列和GO列的标题提取基因的GO注释。后面的`%>% filter(str_detect(GO,"GO"))`是筛选GO列值包含"GO"的行，删除空值。

library(stringr)
all_go_list=str_split(goterms$GOs,",") #分隔goterms的GOs值，逗号是分隔符
gene2go <- data.frame(GID = rep(goterms$query, times = sapply(all_go_list, length)), GO = unlist(all_go_list), EVIDENCE = "IEA") %>% filter(str_detect(GOs,"GO")) #注释换成单个GO编号一行的格式。eggnog里每个基因一行注释，GOs注释里可能有多个GO编号信息，要转换成每个GO编号一行的格式（看下面）。
write.table(gene2go,file="gene2go.txt",sep="\t",row.names=F,quote=F) #保存成文件gene2go.txt

head(gene2go) #查看gene2go前六行
      GID         GO EVIDENCE
1 mc00002 GO:0000003      IEA
2 mc00002 GO:0000323      IEA
3 mc00002 GO:0000325      IEA
4 mc00002 GO:0002376      IEA
5 mc00002 GO:0003006      IEA
6 mc00002 GO:0003674      IEA
```

### 1.3.4. 提取注释的KEGG信息
从egg中提取KEGG注释并整理成gene2pathway.txt文件
1. 提取KEGG信息
```R
koterms <- egg %>%dplyr::select(GID = query, KO=KEGG_ko)%>%na.omit()%>% filter(str_detect(KO,"ko")) #根据egg第一列和KEGG_ko列的标题提取基因的KEGG注释。
head(koterms) #查看koterms前六行
      GID                  KO
1 mc00001 ko:K14524,ko:K22674
2 mc00002           ko:K16292
3 mc00003           ko:K16392
4 mc00004           ko:K16167
5 mc00006           ko:K16253
6 mc00007 ko:K14664,ko:K21604
```

2. 下载ko00001.json
在网页https://www.genome.jp/kegg-bin/get_htext?ko00001点击**Download json**即可下载ko00001.json，是KEGG的K与ko，ko与通路的对应关系。

3. 运行R脚本ko_json2data.R
`Rscript ko_json2data.R`运行R脚本ko_json2data.R，用ko00001.json生成ko与通路和ko与K的对应关系文件kegg_info.RData。

- ko_json2data.R脚本如下：
```R
if(!file.exists('kegg_info.RData')){
  
  library(jsonlite)
  library(purrr)
  library(RCurl)
  
  update_kegg <- function(json = "ko00001.json",file=NULL) {
    pathway2name <- tibble(Pathway = character(), Name = character())
    ko2pathway <- tibble(Ko = character(), Pathway = character())
    
    kegg <- fromJSON(json)
    
    for (a in seq_along(kegg[["children"]][["children"]])) {
      A <- kegg[["children"]][["name"]][[a]]
      
      for (b in seq_along(kegg[["children"]][["children"]][[a]][["children"]])) {
        B <- kegg[["children"]][["children"]][[a]][["name"]][[b]] 
        
        for (c in seq_along(kegg[["children"]][["children"]][[a]][["children"]][[b]][["children"]])) {
          pathway_info <- kegg[["children"]][["children"]][[a]][["children"]][[b]][["name"]][[c]]
          
          pathway_id <- str_match(pathway_info, "ko[0-9]{5}")[1]
          pathway_name <- str_replace(pathway_info, " \\[PATH:ko[0-9]{5}\\]", "") %>% str_replace("[0-9]{5} ", "")
          pathway2name <- rbind(pathway2name, tibble(Pathway = pathway_id, Name = pathway_name))
          
          kos_info <- kegg[["children"]][["children"]][[a]][["children"]][[b]][["children"]][[c]][["name"]]
          
          kos <- str_match(kos_info, "K[0-9]*")[,1]
          
          ko2pathway <- rbind(ko2pathway, tibble(Ko = kos, Pathway = rep(pathway_id, length(kos))))
        }
      }
    }
    
    save(pathway2name, ko2pathway, file = file)
  }
  
  update_kegg(json = "ko00001.json",file="kegg_info.RData")
  
}
```

4. 读取kegg_info.RData文件
在R里用`load("kegg_info.RData")`读取kegg_info.RData文件，会得到ko2pathway和pathway2name两个对象。
```R
head(ko2pathway) #查看前六行
# A tibble: 6 × 2
  Ko     Pathway
  <chr>  <chr>
1 K00844 ko00010
2 K12407 ko00010
3 K00845 ko00010
4 K25026 ko00010
5 K01810 ko00010
6 K06859 ko00010

head(pathway2name) #查看前六行
# A tibble: 6 × 2
  Pathway Name
  <chr>   <chr>
1 ko00010 Glycolysis / Gluconeogenesis
2 ko00020 Citrate cycle (TCA cycle)
3 ko00030 Pentose phosphate pathway
4 ko00040 Pentose and glucuronate interconversions
5 ko00051 Fructose and mannose metabolism
6 ko00052 Galactose metabolism

write.table(pathway2name,file="pathway2name.txt",sep="\t",row.names=F,quote=F) #【optional】保存成文件pathway2name.txt，可以给通用KEGG富集分析的结果注释KEGG Pathway的描述(Name)用。
```

5. 获得gene2pathway
```R
library(stringr)
colnames(ko2pathway)=c("KO",'Pathway') #把ko2pathway的列名改为KO和Pathway，与koterms一致。
koterms$KO=str_replace_all(koterms$KO,"ko:","") #把koterms的KO值的前缀ko:去掉，与ko2pathway格式一致。
gene2pathway <- koterms %>% left_join(ko2pathway, by = "KO") %>%dplyr::select(GID, Pathway) %>%na.omit() #合并koterms和ko2pathway到gene2pathway，将基因与pathway的对应关系整理出来

##【optional】下面为可选步骤
gene2pathway_name<-left_join(gene2pathway,pathway2name,by="Pathway") #合并gene2pathway和pathway2name
write.table(gene2pathway_name,file="gene2pathway_name.txt",sep="\t",row.names=F,quote=F) #【optional】保存成文件gene2pathway_name.txt，可以给通用KEGG富集分析用。

head(gene2pathway)  #查看前六行
       GID Pathway
3  mc00003 ko01002
8  mc00009 ko01002
13 mc00016 ko03012
14 mc00016 ko03029
16 mc00018 ko03037
17 mc00018 ko04812

head(gene2pathway_name) #查看前六行
       GID Pathway                                                Name
1 mc01138 ko00511                            Other glycan degradation
2 mc00017 ko04142                                            Lysosome
3 mc01022 ko00930                             Caprolactam degradation
4 mc00072 ko01002              Peptidases and inhibitors [BR:ko01002]
5 mc01058 ko00130 Ubiquinone and other terpenoid-quinone biosynthesis
6 mc00288 ko03440                            Homologous recombination
```

### 1.3.5. 创建OrgDb包
1. 参数准备
- Taxonomy ID(tax_id)
在[NCBI的Taxonomy网站](https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi)，通过搜索物种的学名，点击结果中的学名就可以链接到Taxonomy ID信息，比如Melastoma candidum的Taxonomy ID为119954。
- gene_info, gene2go, koterms, gene2pathway是前面步骤生成的四个对象
- version：格式是0.0.1
- maintainer和author的格式包括名字和邮箱
- outputDir是创建的OrgDb包的输出路径
- genus是属名，species是种加词。
- goTable：生成GO的详细注释信息。需要go参数值的标题行是"GID, GO and EVIDENCE"。

2. 创建命令
```R
makeOrgPackage(gene_info=gene_info, go=gene2go, ko=koterms,  pathway=gene2pathway, version="0.0.1", maintainer='yanzhong <yan.zhong.sino@gmail.com>', author='yanzhong <yan.zhong.sino@gmail.com>',outputDir=".", tax_id="119954", genus="Melastoma", species="candidum",goTable="go")
```

生成org.Mcandidum.eg.db文件夹，即为制作的Orgdb包。

- notes:如果报错`GO Ids must be formatted like 'GO:XXXXXXX'`则需要检查gene2go数据的GO值是否有冗余或污染。

### 1.3.6. 使用OrgDb包
1. 安装加载
- 创建的org.Mcandidum.eg.db文件夹本质上是个R包，需要安装和加载才能使用。

```R
install.packages('org.Mcandidum.eg.db',repos = NULL, type="source") #安装包
library(org.Mcandidum.eg.db) #加载包
```

- 安装后会在R包存放位置生成org.Mcandidum.eg.db文件夹，这个位置下面`C:\Users\name\Documents\R\win-library\4.1`，以后使用即可直接加载包

2. 用于GO分析

`enrichGO(gene = genes, keyType="GID", OrgDb = org.Mcandidum.eg.db, ont = "ALL", pAdjustMethod = "BH", pvalueCutoff  = 0.05, qvalueCutoff  = 0.2)`

加载后在`groupGO(keyType="GID", OrgDb = org.Mcandidum.eg.db)`,`enrichGO(keyType="GID", OrgDb = org.Mcandidum.eg.db)`,`gseGO(keyType="GID", OrgDb = org.Mcandidum.eg.db)`等函数里把包赋值给`OrgDb`参数，`keyType`参数指定GID即可使用。

3. 用于KEGG分析
waiting...

## 1.4. GO富集分析
上面三种方法获取了背景数据集，接下来就可以使用enrichGO()函数进行GO富集分析。

下面是用enrichGO做go富集分析的例子。
```R
library(clusterProfiler)
data <- read.table("gene.list",header=F) #读取gene ID list
genes <- as.character(data$V1) #转换成字符格式
ego <- enrichGO(gene          = genes, # list of entrez gene id
                OrgDb         = org.Mcandidum.eg.db, # 背景使用分析物种的org包
                keyType       = 'GID', # 输入基因的类型，命令keytypes(org.Mcandidum.eg.db)会列出可用的所有类型，如果按上面步骤制备的数据库，则使用GID；
                ont           = "ALL", #  "BP", "MF", "CC", "ALL"。GO三个子类。如果选ALL，会同时获得三个子类的结果(相当于单独做三个子类的结果合并在一起)，结果中增加一列ONTOLOGY为子类。
                pAdjustMethod = "BH", # 指定多重假设检验矫正的方法，选项包含 "holm", "hochberg", "hommel", "bonferroni", "BH", "BY", "fdr", "none"
                pvalueCutoff  = 0.05, # 富集分析的pvalue，默认是pvalueCutoff = 0.05，更严格可选择0.01
                qvalueCutoff  = 0.2, # 富集分析显著性的qvalue，默认是qvalueCutoff = 0.2，更严格可选择0.05
                readable      = TRUE ) # 是否将gene ID转换到 gene symbol
write.table(as.data.frame(ego),"go_enrich.tsv",sep="\t",row.names =F,quote=F) #保存到文件go_enrich.csv。其中as.data.frame(ego)把ego对象转换成数据框dataframe
```

# 2. KEGG数据库过表达分析(ORA)的背景数据集选择
## 2.1. 支持的物种
有些物种在KEGG数据库里有参考基因组数据库的支持，可以直接用于KEGG分析。
### 2.1.1. 查询
1. 网页查看
- 通过[KEGG Organisms](http://www.genome.jp/kegg/catalog/org_list.html)访问 KEGG 支持的生物的完整列表，目前包含700+真核生物。
- ctrl+F搜索学名来查看是不是已有参考基因组
- 如果在其中，就记录下物种简写(kegg_code,比如hsa)
2. 代码搜索
- clusterProfiler包提供search_kegg_organism()可以搜索支持的物种。
```R
library(clusterProfiler)
search_kegg_organism('Eucalyptus', by='scientific_name')
    kegg_code    scientific_name common_name
313       egr Eucalyptus grandis    rose gum
```
- 记录物种对应的物种简写(kegg_code,这里是egr)

### 2.1.2. 使用
查询到有分析物种时，在`enrichKEGG(organism='egr')`,`gseKEGG(organism='egr')`,`enrichMKEGG(organism='egr')`,`gseMKEGG(organism='egr')`函数里直接把物种简写(kegg_code)赋值给`organism`参数即可使用。

## 2.2. 其他物种
如果分析物种不在上面支持的物种里，那么就选择下面的通用富集分析 —— enricher()和GSEA()啦~

# 3. 通用富集分析 —— enricher()和GSEA()
- 以上介绍的方法都是寻找/创建OrgDb对象或已有数据集，并用enrichGO(),enrichKEGG(),gseGO(),gseKEGG()函数实现GO和KEGG分析的。
- clusterProfiler开发者还专门开发了通用的富集分析函数enricher()和GSEA()，可以实现适用性更广的分析。

基因组功能注释的软件和输出参考这篇博客:[基因组功能注释](https://yanzhongsino.github.io/2021/05/17/omics_genome.annotation_function/)

## 3.1. enricher的背景数据集
整理分析物种基因组的功能注释成enricher接受的格式。
1. 注释文件annotation.txt的格式
- 数据表格data.frame格式，包含三列。
- 第一列为Gene ID，第二列为 GO ID(单个GO ID)，第三列为GO_Description(如果没有可以用任意单词替代，富集分析后再做GO到GO_Description的注释)，行间的顺序无要求。
- enricher同样适用KEGG的富集分析，注释文件格式是一样的，第一列Gene ID，第二列为单个KEGG Pathway ID(ko01001)，第三列为KEGG_Pathway_Description(如果没有也可以用任意单词替代)。

| GeneID | GO         | GO_Description  |
| ------ | ---------- | --------------- |
| gene00001 | GO:0005819 | spindle         |
| gene00002 | GO:0072686 | mitotic spindle |
| gene00003 | GO:0000776 | kinetochore     |

2. PANNZER功能注释整理
- pannzer2_GO.out.txt是pannzer2的GO注释结果，提取第1(Gene ID)，3(GO ID)，4(description)列，保存为pannzer_go_annotation.txt。
`cat pannzer2_GO.out.txt |awk '{print $1"\tGO:"$3"\t"$4}' >pannzer_go_annotation.txt`
- PANNZER的注释是一个GO ID一行的格式，所以不需更多整理了。

3. Interproscan功能注释整理
- iprscan.tsv是interproscan的注释结果，提取第1(Gene ID)和第14(GO IDs)列，保存为iprscan_gos.txt。
`cat interproscan/mc.iprscan.tsv |cut -f1,14 |grep "GO"|sed "s/|/,/g" >iprscan_gos.txt`
- Interproscan的注释是一个基因的所有GO IDs都在第14列，所以需要处理成单个GO ID一行的格式。下面的R脚本示例处理iprscan_gos.txt成iprscan_go_annotation.txt文件。
```R
library(stringr)
iprscan_go<-read.table("iprscan_gos.txt",sep="\t",header=F)
all_go_list=str_split(iprscan_go$V2,",") #分隔iprscan_go的GOs值，逗号是分隔符
ip_gene2go <- data.frame(GID = rep(iprscan_go$V1, times = sapply(all_go_list, length)), GO = unlist(all_go_list), EVIDENCE = "IEA") #把多个GO编号一行的格式转换成每个GO编号一行的格式，并添加标题行，添加EVIDENCE列作为GO_Description。
write.table(ip_gene2go,file="iprscan_go_annotation.txt",sep="\t",row.names=F,quote=F) #保存成文件iprscan_go_annotation.txt
head(ip_gene2go) #查看ip_gene2go前六行
      GID         GO EVIDENCE
1 mc16959 GO:0003779      IEA
2 mc16959 GO:0003779      IEA
3 mc32011 GO:0000148      IEA
4 mc32011 GO:0003843      IEA
5 mc32011 GO:0006075      IEA
6 mc32011 GO:0016020      IEA
```

4. 其他功能注释
- 功能注释软件有很多，结果文件的格式也各有差别，如果有注释到GO，都可以整理成这种格式，作为背景数据集，用作enricher的富集分析。
- 在前面节**通用富集分析 —— AnnotationForge**中从eggNOG注释结果中提取的gene2go.txt文件格式与这里也一致，适用于enricher分析。

## 3.2. enricher富集分析
可以使用enricher()和gseGO()函数进行ORA和GSEA分析。

下面是用enricher做go富集分析的例子，KEGG的enricher富集分析也类似。
```R
data <- read.table("gene.list",header=F) #读取gene ID list
genes <- as.character(data$V1) #转换成字符格式
go_anno <- read.table("go_annotation.txt",header = T,sep = "\t") #读取go_annotation.txt，可以是pannzer_go_annotation.txt或者iprscan_go_annotation.txt
go2gene <- go_anno[, c(2, 1)] #读取第1，2列
go2name <- go_anno[, c(2, 3)] #读取第2，3列
ego <- enricher(genes, TERM2GENE = go2gene, TERM2NAME = go2name, pAdjustMethod = "BH",pvalueCutoff  = 0.05, qvalueCutoff  = 0.2) #enricher分析，并保存在ego。
write.table(as.data.frame(ego),"go_enrich.csv",sep="\t",row.names =F,quote=F) #保存到文件go_enrich.csv。其中as.data.frame(ego)把ego对象转换成数据框dataframe
```

# 4. GO和KEGG描述查询
如果在通用富集分析中准备的背景数据集第三列描述栏没有信息，可以在富集结果出来后根据GO ID或者KEGG Pathway ID查询。

## 4.1. GO term 查询
1. AmiGO2查询
- 在[GO官网](http://geneontology.org/))使用的[AmiGO2网站](http://amigo.geneontology.org/amigo/landing)查询GO ID和GO term信息。
- [WEGO 2.0](https://wego.genomics.cn/)也可以查询。
- **查询方法**：在[AmiGO2网站](http://amigo.geneontology.org/amigo/landing)的搜索框输入GO ID(格式是 GO:0000000)即可查到GO term information。

## 4.2. KEGG Pathway 查询
1. KEGG官网查询
- [KEGG网站](https://www.kegg.jp/kegg/)提供了KEGG信息查询入口，包括[KEGG Pathway](https://www.kegg.jp/kegg/pathway.html)中查询KEGG Pathway ID(格式是 ko00000)的详细信息。
- **查询方法**：把[KEGG Pathway](https://www.kegg.jp/kegg/pathway.html)的搜索前缀(Select prefix)改成ko，在搜索框(Enter keywords)输入KEGG Pathway ID(格式是5个数字 00000)，点击查询Go即可得到KEGG Pathway information。
- 官网查询只发现了单个查询的方法，批量查询还是推荐使用文件查询方法。
2. 文件查询【推荐】
使用前面**提取注释的KEGG信息**第4步骤得到的pathway2name.txt文件，获取KEGG Pathway ID对应的描述信息(Name列)。

# 5. references
1. clusterProfiler github：https://github.com/YuLab-SMU/clusterProfiler
2. clusterProfiler paper：https://www.cell.com/the-innovation/fulltext/S2666-6758(21)00066-7?_returnURL=https%3A%2F%2Flinkinghub.elsevier.com%2Fretrieve%2Fpii%2FS2666675821000667%3Fshowall%3Dtrue
3. clusterProfiler book：http://yulab-smu.top/biomedical-knowledge-mining-book/index.html
4. clusterProfiler manual：https://bioconductor.org/packages/devel/bioc/manuals/clusterProfiler/man/clusterProfiler.pdf 
5. clusterProfiler ducumentation：https://guangchuangyu.github.io/software/clusterProfiler/documentation/
6. 用AnnotationForge进行非模式物种注释构建：https://www.jieandze1314.com/post/cnposts/208/
7. 用AnnotationHub获取非模式物种注释信息：https://www.bioinfo-scrounger.com/archives/512/


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>