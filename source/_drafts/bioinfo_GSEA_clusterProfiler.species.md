---
title: 基因富集分析(gene set enrichment analysis, GSEA) —— clusterProfiler准备：使用R包AnnotationForge构建非模式物种的Orgdb包
date: 2022-04-24
categories: 
- bio
- bioinfo
tags: 
- gene set enrichment analysis
- GSEA
- topGO
description: 使用clusterProfiler进行基因富集分析时，非模式物种Orgdb包的构建。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

模式物种已有Orgdb包可以直接使用，如果分析的是非模式物种，则需要自行构建Orgdb包。


构建方法

1. 删除冗余
准备egomapper的结果emapper.annotations.tsv

把##开头的行和标题行前的#删除
sed '/^##/d' emapper.annotations.tsv |sed 's/#//' >eggmapper.anno


R读取数据

egg<-read.csv("eggnog.anno.tsv",header=T,sep="\t") #如果用read.csv报错，尝试read.table("eggnog.anno.tsv",header=T,sep="\t")，或者试试egg <- rio::import('emapper.annotations.tsv')。

egg[egg==""] <- NA

colnames(egg) #列出标题行
[1] "query"          "seed_ortholog"  "evalue"         "score"
 [5] "eggNOG_OGs"     "max_annot_lvl"  "COG_category"   "Description"
 [9] "Preferred_name" "GOs"            "EC"             "KEGG_ko"
[13] "KEGG_Pathway"   "KEGG_Module"    "KEGG_Reaction"  "KEGG_rclass"
[17] "BRITE"          "KEGG_TC"        "CAZy"           "BiGG_Reaction"
[21] "PFAMs"

gene_info <- egg %>%dplyr::select(GID = query, GENENAME = seed_ortholog) %>% na.omit() #根据第一和第二列的标题提取前两列。

gterms <- egg %>%dplyr::select(query, GOs) %>% na.omit() #根据第一列和GO列的标题提取基因的GO注释。

GO注释多行转换成单行。因为eggnog里每个基因一行注释，可能GOs注释里有多个GO编号信息，要转换成每个GO编号一行的格式。后面的`%>% filter(str_detect(GO,"GO"))`是筛选GO列值包含"GO"的行，删除空值。
library(stringr)
all_go_list=str_split(gterms$GOs,",")
gene2go <- data.frame(GID = rep(gterms$query, times = sapply(all_go_list, length)), GO = unlist(all_go_list), EVIDENCE = "IEA") %>% filter(str_detect(GO,"GO"))

> head(gene2go)
      GID         GO EVIDENCE
1 mc00002 GO:0000003      IEA
2 mc00002 GO:0000323      IEA
3 mc00002 GO:0000325      IEA
4 mc00002 GO:0002376      IEA
5 mc00002 GO:0003006      IEA
6 mc00002 GO:0003674      IEA


gene2ko <- egg %>%dplyr::select(GID = query, KO=KEGG_ko)%>%na.omit() #根据第一列和KEGG_ko列的标题提取基因的KEGG注释。

下载ko00001.json
在网页https://www.genome.jp/kegg-bin/get_htext?ko00001点击Download json即可下载ko00001.json。

`Rscript ko_json2data.R`运行R脚本ko_json2data.R，用ko00001.json生成ko与通路和ko与K的对应关系文件kegg_info.RData

```ko_json2data.R
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

在R里用`load("kegg_info.RData")`读取kegg_info.RData文件

head(gene2ko),head(ko2pathway)和head(pathway2name)查看前六行

```
> head(gene2ko)
      GID                  KO
1 mc00001                   -
2 mc00002                   -
3 mc00003           ko:K16292
4 mc00004                   -
5 mc00006                   -
6 mc00007 ko:K14664,ko:K21604

> head(ko2pathway)
# A tibble: 6 × 2
  Ko     Pathway
  <chr>  <chr>
1 K00844 ko00010
2 K12407 ko00010
3 K00845 ko00010
4 K25026 ko00010
5 K01810 ko00010
6 K06859 ko00010

> head(pathway2name)
# A tibble: 6 × 2
  Pathway Name
  <chr>   <chr>
1 ko00010 Glycolysis / Gluconeogenesis
2 ko00020 Citrate cycle (TCA cycle)
3 ko00030 Pentose phosphate pathway
4 ko00040 Pentose and glucuronate interconversions
5 ko00051 Fructose and mannose metabolism
6 ko00052 Galactose metabolism
```

colnames(ko2pathway)=c("KO",'Pathway') #把ko2pathway的列名改为KO和Pathway，与gene2ko一致。

library(stringr)
gene2ko$KO=str_replace(gene2ko$KO,"ko:","") #把gene2ko的KO值的前缀ko:去掉，与ko2pathway格式一致。

gene2pathway <- gene2ko %>% left_join(ko2pathway, by = "KO") %>%dplyr::select(GID, Pathway) %>%na.omit() #合并gene2ko和ko2pathway到gene2pathway，将基因与pathway的对应关系整理出来

> head(gene2pathway)
       GID Pathway
3  mc00003 ko01002
8  mc00009 ko01002
13 mc00016 ko03012
14 mc00016 ko03029
16 mc00018 ko03037
17 mc00018 ko04812


makeOrgPackage(gene_info=gene_info, go=gene2go, ko=gene2ko,  pathway=gene2pathway, version="0.0.1", maintainer='yanzhong <yan.zhong.sino@gmail.com>', author='yanzhong <yan.zhong.sino@gmail.com>',outputDir=".", tax_id="119954", genus="Melastoma", species="candidum",goTable="go")

生成org.Mcandidum.eg.db文件夹，即为制作的Orgdb包。

如果报错`GO Ids must be formatted like 'GO:XXXXXXX'`则需要检查gene2go数据的GO值是否有冗余或污染。

# 使用
install.packages('org.Mcandidum.eg.db',repos = NULL, type="source") #安装包
library(org.Mcandidum.eg.db) #加载包

ego_all <- enrichGO(gene = glist,
                keyType="GID",
                #模式物种
                #OrgDb = org.Mm.eg.db,
                #非模式物种
                OrgDb = org.Mcandidum.eg.db,
                ont = "ALL", #ALL或BP或MF或CC
                pAdjustMethod = "BH",
                #pvalueCutoff  = 0.01,
                qvalueCutoff  = 0.05) 

使用包


references
1. https://www.jieandze1314.com/post/cnposts/208/

