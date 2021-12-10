---
title: 基因富集分析(gene set enrichment analysis, GSEA)—— topGO
date: 2021-11-12 16:00:00
categories: 
- bio
- bioinfo
tags: 
- gene set enrichment analysis
- GSEA
- topGO
description: 介绍了基因富集分析R包topGO。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=108151&auto=1&height=32"></iframe></div>


## 2.2. GO term
GO term分为三大类：
- cellular component(CC)-细胞成分（其中基因产物位于细胞内部）
- molecular function(MF)-分子功能（基因产物的功能是什么）
- biology process(BP)-生物过程（即基因产物参与的一系列事件）
三类都可以用topGO做富集分析。
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

# 3. references
[topGO tutorial](https://bioconductor.org/packages/release/bioc/vignettes/topGO/inst/doc/topGO.pdf)
[topGO blog](https://datacatz.wordpress.com/2018/01/19/gene-set-enrichment-analysis-with-topgo-part-1/)
[R topGO](https://www.codenong.com/cs105162324/)