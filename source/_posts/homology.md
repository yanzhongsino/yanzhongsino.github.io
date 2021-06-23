---
title: homology
date: 2021-06-23 16:50:00
categories: 
- bio
  - concept
tags: 
- homolog
- ortholog
- paralog
- xenolog
- analog
- orthology
- orthogroup
- gene family
description: 解释了同源性(homology)在进化生物学应用的相关概念，包括homolog,ortholog,paralog,xenolog,analog,orthology,orthogroup,gene family。
---

{% img http://www.discoveryandinnovation.com/bioinformatics/glossary_detail/images/orthologs.jpg 200 400 vi-vim-cheat-sheet %}

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=3986241&auto=1&height=32"></iframe></div>


**homology**在生物学上被翻译成“同源性”，是指在不同生物类群中，由于共同的祖先，在一对结构或基因间表现出来的相似性。在基因序列(DNA,RNA or Protein)上体现出的相似性被称作序列同源性(sequence homology)，具有序列同源性的基因被称做同源基因(homolog=homologous gene)，常用于进化生物学的研究。

1. 同源基因(homolog)
按不同成因分为两类（也有把homolog分为ortholog，paralog和xenolog三类的）：
- 直系同源基因（ortholog）：物种形成(speciation)导致，一个共同祖先的基因由于物种形成分离到不同物种间；orthologs通常（不是绝对）保留相同的功能。
- 旁系同源基因(paralog)：重复(duplication)导致，一个物种的基因组内由于基因复制产生的同源性；paralogs通常具有不同的功能，或者成为假基因(pseudogene)；不同物种间不是由于物种形成导致的同源基因也被称为paralogs。在旁系同源基因中，起源于一次全基因组重复(whole-genome duplication, WGD)的那些被称为ohnolog，由Ken Wolfe在2000年第一次给出定义。


2. 扩展概念
- 直系同源基因：单数形式ortholog=orthologous gene；复数形式orthologs=orthologous genes。
- 直系同源(orthology)：一个名词，代表直系同源基因的直系同源属性；相应地，有旁系同源(paralogy)。
- 纯正并交群(orthogroup)：在一群物种中，起源于这群物种的最近共同祖先(last common ancestor, LCA)的一个基因的一组基因，是这群物种中具有直系同源性的基因家族(gene family)的合集。

按照逻辑，应该有一套完整的单词：

|\*log 特指基因|\*logy 属性|\*logous 形容词|\*group|
|---|---|---|---|
|ortholog|orthology|orthologous|orthogroup|
|paralog|paralogy|paralogous|paragroup|
|homolog|homology|homologous|homogroup|

但我查了下，paragroup,homogroup这两个词是不存在的。
按照orthogrop的定义，个人觉得用homogroup代替orthogroup好像更好，因为纯正并交群内包括orthologs和paralogs，即包括所有homologs。

3. 相关概念
- 异同源基因(xenolog)：水平基因转移(horizontal/lateral gene transfer)导致，来源于寄生，腐生，共生或病毒侵染等途径转移产生的异同源基因；一般具有相似功能，但如果新环境差异巨大，功能可能出现分化。xenolog一词最早在1970年由Walter Fitch创造。由于异同源基因不是来自共同祖先，而是拥有独立起源，
- analog（不知道咋翻译，同功基因？）：在不同物种中，不相关的基因有着相似的功能，但是独立的进化起源。
按照定义，xenolog应该是一种analog。
- 基因家族(gene family)：是指在一个物种中，起源于一个基因的复制，通常具有相似生化功能的一套基因。

{% img https://upload.wikimedia.org/wikipedia/commons/thumb/4/4b/Ortholog_paralog_analog_examples.svg/400px-Ortholog_paralog_analog_examples.svg.png 200 400 vi-vim-cheat-sheet %}


**reference**
[wiki-homology](https://en.wikipedia.org/wiki/Homology_(biology))
[wiki-sequence homology](https://en.wikipedia.org/wiki/Sequence_homology#Homoeology)
[Paper-Orthologs, Paralogs, and Evolutionary Genomics](https://www.annualreviews.org/doi/abs/10.1146/annurev.genet.39.073003.114725)
[Paper-analog和xenolog的由来和定义](https://academic.oup.com/sysbio/article-abstract/19/2/99/1655771?redirectedFrom=fulltext)
[Paper-ohnolog的由来和定义](https://www.nature.com/articles/ng0500_3)