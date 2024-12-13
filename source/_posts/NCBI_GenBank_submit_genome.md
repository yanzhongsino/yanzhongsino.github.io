---
title: 提交基因组到公共数据库 —— 以GenBank为例
date: 2022-03-22 21:30:00
categories:
- NCBI
- GenBank
- submit
- genome
tags:
- genome
- genome submit
- GenBank
- table2asn
- tbl2asn

description: 此文记录提交基因组到NCBI的GenBank数据库的步骤，常见错误和修改，以及注意事项。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=31721697&auto=1&height=32"></iframe></div>

# 1. 基因组数据库
提交基因组组装的数据到公共数据库以实现共享，在发表提供基因组数据文章时在文章内提供基因组的accession number。

常用的公共数据库包括：
- [NCBI-GenBank](https://submit.ncbi.nlm.nih.gov/)：美国生物信息学国家中心
- [EMBL-EBI](https://www.ebi.ac.uk/)：欧洲生物信息学研究所
- [DDBJ](https://www.ddbj.nig.ac.jp/index-e.html)：日本DNA数据银行
- [中国的国家基因组科学数据中心, National Genomics Data Center, NGDC](https://ngdc.cncb.ac.cn/?lang=zh): 中国

# 2. 基因组提交
可以单独提交基因组，也可以基因组和基因组注释一同提交。

单独提交基因组会比加上注释信息简单很多，因为加上注释信息一般都会面临多种注释错误(我的例子是4万个基因注释中包含了将近一千个错误需要手动修改)，但是现在基因组注释是非常基础的操作，所以还是建议加上注释文件更完整。如果发表时间紧张，可以考虑先提交基因组获得accession number(在文章里提供)，再提交注释(accession number不变)。

- 单独提交基因组只需要准备基因组文件(fasta格式)，在提交网站填写基因组相关信息，上传基因组文件即可。
- 基因组与注释文件共同提交则需要对注释文件(gff或者bed格式)做检查和过滤后，用table2asn软件从基因组文件(fasta格式)和注释文件(gff或者bed格式)生成一个sqn文件，再提交sqn文件到提交网站上。当然，单独的基因组文件也可以转化成sqn格式再上传。

在[NCBI的提交网站](https://submit.ncbi.nlm.nih.gov/)可以选择各种数据类型，提交基因组就输入**genome**。

# 3. 文件准备
在整理基因组文件，基因组结构注释和功能注释文件时，可以先用table2asn检测错误，以指导基因组改善(例如可能删除污染片段)，以及合并注释文件。

## 3.1. 基因组文件
基因组文件用fa格式储存，最好做一做污染过滤。

## 3.2. 注释文件
- 注释文件可以是tbl格式或者gff格式，我的习惯是用gff格式(许多注释软件结果也是gff格式)。
- 提交的注释文件可以只有结构注释(不提供功能注释，或者通过Dryad【收费】或Figshare【免费20GB容量】等文件分享网站提供功能注释文件)，也可以把功能注释整理后添加到结构注释文件的product属性值。

### 3.2.1. 功能注释
功能注释以product属性值的形式提供，添加到结构注释gff文件的第九列。

合并功能注释和结构注释可以参考博客[基因组注释(三)：基因功能注释](https://yanzhongsino.github.io/2021/05/17/omics_genome.functional.annotation/)

product值的不规范情况：
- 包含逗号,（因为逗号是分隔符）
- 包含竖杠|
- 短横杠-开头或结尾
- 冒号:开头或结尾
- 斜杠/开头或结尾
- 不包含字母
- 括号（包括圆括号,中括号和大括号）不完整的

# 4. 提交基因组
## 4.1. 提交基因组到NCBI的GenBank
单独提交基因组只需要基因组(fasta格式)文件，在基因组提交网站[genome Submit Portal](https://submit.ncbi.nlm.nih.gov/subs/genome/)操作。

1. 注册和登录NCBI账号
2. 选择提交类型Submission Type：单个基因组Single genome或者批量多个基因组Batch/multiple genomes

然后进入信息填写页面：
1. 提交者信息Submitter
   - 包括姓名First and Last name，邮箱Email，单位Submitting organization(学校)，部门Department(学院)，街道Street，城市City，省/州State/Province，邮编Postal code，国家Country。
   - 在最后可以选择**Update my contact information in profile**会更新填写的信息到账号，下一次新填写就默认填入提交者信息了。

2. 一般性的信息General info
   - BioProject和BioSample可以先注册在这里提供accession number；也可以选择还未注册提交之后会增加填写BioProject和BioSample信息的步骤然后自动注册新的；
   - 释放日期Release date可以指定日期，也可以提交通过后立即释放；释放日期可以提交后发邮件更改的；
   - 基因组信息Genome info：如果保存到了sqn文件也可以选择**Genome Assembly structured comment is in the contig .sqn file(s)**就不需要填写(但我选了之后不识别还是不选填入)。包括组装日期Assembly date，组装方法Assembly method和版本，组装名称Assembly name，基因组覆盖度Genome coverage(这里其实是depth)，测序技术Sequencing technology；
   - 确认信息：是否完整基因组，是否最终版，是否de novo从头组装即无参考组装，是否已有提交的更新，是否自动移除被系统认为是污染的序列(建议自动移除，否则可能发邮件让手动移除污染序列)；
   - Submission title可选填，建议填上以区别于其他提交。

3. BioProject General Info
   - 提供创建新的BioProject的信息，项目题目Project title，项目描述Project description，和领域Relevance
   - 可选项：相关的外部链接External links，基金grants

4. Publications【only batch/multiple genomes】：选填，发表文章的PubMed ID或者DOI号

5. BioSample Type：样品类型，选择Human/Plant等

6. BioSample Attributes
   - 如果是批量提交基因组，可以用excel或csv表格提交多个BioSamples信息
   - 样品名称Sample Name：区别于其他样品的唯一名称
   - 生物名称Organism，可以填物种学名；个体描述isolate/栽培名称cultivar/生态型ecotype三选一填，年龄age/发育阶段development stage二选一填
   - 地理位置geographic location：可以填国家China
   - 组织tissue：填写取样部位，eg leaf，flower，root，stem

7. Genome Info【only batch/multiple genomes】
   - 批量提交的基因组应该有相同的类型：要么只有染色体序列，要么都包含非染色体的序列(会被处理到WGD genome程序)
   - 确认信息：是否完整基因组，是否最终版，是否de novo从头组装即无参考组装，是否已有提交的更新，是否自动移除被系统认为是污染的序列(建议自动移除，否则可能发邮件让手动移除污染序列)；

8. Files
    - 提交基因组文件，单个基因组提交fasta格式或者sqn格式，批量基因组提交fasta格式或者asn.1格式
    - FTP或Aspera命令行上传文件夹(>10GB)，或者网页提交(<2GB)或Aspera Connect plugin(2-10GB)上传文件
    - 提交之后会检查信息是否错误，提示修改

9. 空隙Gaps【only batch/multiple genomes】
   - 基因组序列中的N的含义，是否随机合并序列(不用组装软件)，感觉一般都不会随机合并吧
   - 指定代表gap的N的最小数量，是否有代表未知长度gap的N的数量(常用大于100个N代表未知长度的gap)，组装gap的证据的数据源

10. Assignment【only single genomes】：确认是否有染色体序列，填写染色体序列信息，是否有质体或线粒体或质粒序列    
11. References【only single genomes】：序列作者和相关文章信息
12. Review&Submit：最后检查一遍信息没错误就确认提交

## 4.2. 提交基因组和注释
提交带有注释的基因组的大致步骤：
1. 注册新的BioProject和BioSample
2. 生成template.sbt文件
3. 用table2asn把基因组(fsa)和注释文件(gff/tbl)生成用于提交的sqn文件
4. 根据生成的验证结果文件查看table2asn检查出来的错误，并修正基因组和注释文件中包含的错误
5. 重新运行table2asn和修正错误(重复步骤3和4)直至没有错误
6. 根据上一个部分**提交基因组到NCBI的GenBank**的步骤提交sqn文件到NCBI
7. 提交后可能会收到邮件需要再次更改，更改后重新运行table2asn和修正错误(重复步骤3和4)直至没有错误在NCBI修正错误(提交新的sqn文件)

### 4.2.1. 准备工作
#### 4.2.1.1. 注册新的BioProject和BioSample
- 在网站[submit](https://submit.ncbi.nlm.nih.gov/subs/)提交新的BioProject和BioSample
- 根据项目和样品信息，计划发表的信息填写，勾选自动生成Locus Tag Prefixes。
- 提交后马上可以获得BioProject ID(PRJNAxxxxxx)和BioSample ID(SAMNxxxxxxxx)，以及Locus Tag Prefixes(xxxxx)三个编号信息用于genome的提交填写。

#### 4.2.1.2. 生成template.sbt文件
在网站[template](https://submit.ncbi.nlm.nih.gov/genbank/template/submission/)提交基因组信息和BioProject ID(PRJNAxxxxxx)和BioSample ID(SAMNxxxxxxxx)，生成template.sbt文件。

### 4.2.2. 生成提交文件
用table2asn把基因组(fsa)和注释文件(gff/tbl)生成用于提交的sqn文件。

table2asn还有一个功能，是在整理基因组文件，基因组结构注释和功能注释文件时，可以先用table2asn检测错误，以指导基因组(例如可能删除污染片段)，以及合并注释文件。

#### 4.2.2.1. table2asn简介
table2asn用于把基因组和其他信息转换成NCBI的GenBank认可的ASN.1 (Abstract Syntax Notation 1)文本格式文件(sqn后缀)和其他用于协助修改的文件，是命令行格式的软件，取代了原来的老款软件tbl2asn。

另一个相同功能的图形界面软件是Genome Workbench(尝试了一下，数据量大的情况下Window版本会卡顿，还是推荐table2asn)。

#### 4.2.2.2. 安装table2asn
在网址下载对应系统(linux64,mac,win64)版本的table2asn安装包，解压缩，重命名为table2asn并设置权限(linux系统下运行`chmod 711 table2asn`)。

- [table2asn下载网址](https://ftp.ncbi.nlm.nih.gov/asn1-converters/by_program/table2asn/)
- [table2asn 用法简介](https://www.ncbi.nlm.nih.gov/genbank/table2asn/)
- [table2asn read me](https://ftp.ncbi.nlm.nih.gov/asn1-converters/by_program/table2asn/DOCUMENTATION/table2asn_readme.txt)

#### 4.2.2.3. 运行table2asn
1. 用法
- 最基本的用法：`table2asn -t template.sbt -indir ./`
- 注释的基因组提交的基础用法：`table2asn -t template.sbt -indir ./ -outdir ./ -M n -Z -locus-tag-prefix L6164`
- 注释的基因组提交推荐用法：`table2asn -t template.sbt -indir ./ -outdir ./ -M n -Z -a s -V vb -c fx -euk -augustus-fix -l paired-ends -gaps-min 1 -gaps-unknown 100 -locus-tag-prefix L6164 -j "[organism=Bauhinia variegata][isolate=pink_petal]" -logfile table2asn.log`

2. 参数解释
- -indir ./: 指定存放sample.fsa和sample.gff/.tbl文件的目录。自动识别目录下所有*.fsa和*.gff/tbl文件，需要sample.fsa和sample.gff有相同前缀
- -outdir ./: 指定结果文件目录，默认与输入一致
- -t: 指定template.sbt文件
- -i sample.fsa：指定目录下的单个sample.fsa文件(单个基因组提交)，有基因组的最大限制（大概是2G内），超出限制可以只用-indir不用-i -f指定
- -f sample.gff：指定目录下的sample.gff/tbl文件，不能和-indir同用
- -M n：用作原核或真核基因组提交，代替(-a s -V v -c f)，加上-Z就包含基因组特定的差异验证; -M t则是用作TSA提交，代替(-a s -V v -c f，加上TSA特定的验证)
- -Z: 生成差异报告(Discrepancy report)，保存到sample.dr中；只推荐注释基因组或者转录组提交使用
- -a s: 指定文件类型；-a a指任意格式，包括单个fasta或ASN.1(default)； -a s指一套不相关的序列，例如一个基因组组装；-a s1指物种内的居群集；-a s2指不同物种的系统集
- -V: 验证数据，后面可以跟vbr的任意组合；v验证数据记录，结果保存在sample.val(含有错误的类型和严重程度)，错误总结保存在sample.stats；b生成sample.gbf的GenBank文件；r在不做国家检查的情况下验证(Validates without Country Check)
- -c: 清理数据，后面可以跟fxdD的任意组合；f修正差异报告中特定类别的产物名称，被修正的产物名称保存在sample.fixedproducts；x扩展注释的features的1-2个核苷酸边界，使得连接gaps或序列末端；D/d修正采样日期
- -euk：为差异报告的测试指定是真核生物(eukaryotic lineage)
- -augustus-fix：针对augustus的错误进行修正
- -locus-tag-prefix L6164: 在向NCBI申请BioProject(以及BioSample)后会分配一个locus-tag-prefix前缀，在这里指定，不指定可能在sample.dr中报错FATAL: INCONSISTENT_PROTEIN_ID。
- -gaps-min 1 -gaps-unknown 100: 指定序列中的N的含义，如未正确指定，可能在submit.stats中报错SEQ_INST.InternalNsInSeqRaw。-gaps-min 指定代表gap的连续Ns的最小数量，1指把大于等于1个的连续Ns碱基都转换成assembly_gaps。-gaps-unknown 指定代表未知长度的gap的连续Ns的
- -l paired-ends: 
- -j "[organism=Bauhinia variegata][isolate=pink_petal]": 用于指定样本名[organism]和个体名[isolate]，会添加进每一条序列的ID里；不指定可能在sample.stats中报错SEQ_DESCR.BioSourceMissing和SEQ_DESCR.NoSourceDescriptor。
- -logfile bv.log:指定log文件，如果运q行有错误存放在log文件
- -y：添加评论到每一条提交记录，例如-y "Contigs larger than 2kb have been annotated, representing approximately 87% of the total genome"

### 4.2.3. 修正错误
根据生成的验证结果修正注释错误，重复table2asn生成sqn文件和修正错误的步骤直至没有错误。
#### 4.2.3.1. 查看错误
查看运行table2asn生成的sample.stats，如果有ERROR-level的错误和部分特定的WARNING-level的错误就需要修正，通过sample.val查看错误的具体描述。
另外差异报告文件sample.dr中标注为FATAL的错误也需要修正。

下面是一个sample.stats文件的例子：

```
Total messages:		86696

=================================================================
86259 WARNING-level messages exist

SEQ_FEAT.CDSmRNArange:	219
SEQ_FEAT.NotSpliceConsensusDonor:	300
SEQ_FEAT.NotSpliceConsensusAcceptor:	585
SEQ_FEAT.SeqFeatXrefFeatureMissing:	84578
SEQ_FEAT.ShortExon:	577

=================================================================
437 ERROR-level messages exist

SEQ_INST.StopInProtein:	109
SEQ_INST.InternalNsInSeqRaw:	169
SEQ_DESCR.BioSourceMissing:	47
SEQ_DESCR.NoSourceDescriptor:	1
SEQ_FEAT.InternalStop:	109
SEQ_FEAT.NoStop:	2
```

#### 4.2.3.2. 注释规范
报错常常是注释文件的注释不规范导致的，根据报错我总结出几点注释规范：
1. 同一个mRNA包含的exons与CDSs的数量和位置几乎一致；区别在于同一个mRNA的exons的起始和终止位置(第一个exon的首和最后一个exon的尾)与mRNA的起始和终止相同，包含5'UTR和3'UTR，CDSs则不包含5'UTR和3'UTR，所以CDSs的起始和终止位置不一定和exon相同。
2. 第9列的产物属性product值不能包括逗号,(因为逗号是多个属性值的分隔符)，竖杠|，不能不包含字母，不能以短横杠-或冒号:结尾。最好先做好product值的过滤filter，再添加到gff注释文件。

product值的不规范情况：
- 包含逗号,（因为逗号是分隔符）
- 包含竖杠|
- 短横杠-开头或结尾
- 冒号:开头或结尾
- 斜杠/开头或结尾
- 不包含字母
- 括号（包括圆括号和中括号）不完整的

在整理基因组结构注释和功能注释时，可以先用table2asn检测错误，以指导注释合并。

#### 4.2.3.3. 报错和修正方案
- table2asn运行结束后要根据输出文件的报错进行调整，并重新运行直至没有报错再提交给NCBI。
- 实践中发现修正一个错误后产生新的错误的情况是很常见的，所以多次运行table2asn也是很有必要的。
- NCBI给出了[Validation and Discrepancy Report Error Explanations](https://www.ncbi.nlm.nih.gov/genbank/validation/)和[Validation Error Explanations for Genomes](https://www.ncbi.nlm.nih.gov/genbank/genome_validation/)可以参考报错含义和解决方案。

下面我总结了一下我遇到的报错类型和修正方案：
##### 4.2.3.3.1. sample.stats/sample.val文件中报错ERROR
添加`pseudo=true`属性值的命令举例：`sed -i '/ID=Bv00639;/ {s/$/pseudo=true;/g}' sample.gff`
1. **SEQ_FEAT.InternalStop** 和 **SEQ_INST.StopInProtein**
   - 当gff的CDS位置有中间终止密码子时报错
   - 可以修改CDS序列位置，去除中间终止密码子。如果无法去除中间终止密码子，可以在sample.gff文件对应的gene行(第三列为gene的那行)的第九列添加`pseudo=true`属性值代表序列无法被翻译。
2. **SEQ_FEAT.NoStop**
   - 当gff的CDS位置无终止密码子时
   - 可以延长CDS序列位置，直至出现终止密码子。如果无法找到终止密码子，可以在sample.gff文件对应的gene行(第三列为gene的那行)的第九列添加`pseudo=true`属性值代表序列无法被翻译。
3. **SEQ_FEAT.AbuttingIntervals**
   - 当gff注释的一个基因的exons位置相邻时。
   - 可以合并两个相邻的exons，更改位置。
4. **SEQ_FEAT.SeqLocOrder**
   - 当gff的exon/CDS间的位置有重叠时。
   - 可以参考mRNA和CDS序列的位置更改重叠的exon/CDS位置，删除多余的exon/CDS；exons的边界和mRNA一致，数量和CDS一致。
5. **SEQ_FEAT.CDSmRNAXrefLocationProblem**
   - 当gff注释CDS和exon的位置或数量不一致时
   - 可以更改exon或CDS位置和数量。
6. **SEQ_FEAT.CDSwithMultipleMRNAs**
   - 当gff注释CDS和exon的位置不一致时，这个报错添加`pseudo=true`属性值之后还是存在。实践发现有时与**SEQ_FEAT.CDSmRNAmismatchCount**一同出现且数量一致。
   - 需要更改exon或CDS位置和数量。
7. **SEQ_FEAT.ShortIntron**
   - 含有短于11bp的内含子时
   - 可以修正intron的位置使它长于11bp，或者在gene那行第九列添加`pseudo=true`属性值，或者在table2asn的命令中中加上参数`-c s`代表标记为"LOW QUALITY PROTEIN"。
8. **SEQ_FEAT.TransLen**
   - 表示蛋白质长度与预测的蛋白质长度不匹配，运行错误
   - 建议重跑table2asn，报错持续存在就写邮件把sample.sqn和运行的命令行发给NCBI(genomes@ncbi.nlm.nih.gov)让帮忙修改这个错误。
9. **SEQ_FEAT.BadInternalCharacter**
   - 表示gff的第9列的属性值attributes中包含不符合规定的字符，例如"|"符号。
   - 删除第9列属性值包含的"|"符号
10. **SEQ_FEAT.BadTrailingHyphen**
   - 当gff的第9列的属性值attributes中以连字符"-"结尾时
   - 删除第9列属性值包含的"-"符号
11. **SEQ_FEAT.BadTrailingCharacter**
    - 当gff的第9列的属性值attributes中以连字符":"结尾时。
    - 删除第9列属性值包含的":"结尾符号
12. 当gff的product属性不符合蛋白产物名称规定时，会生成Adiantum.nelumboides.fixedproducts文件，记录把不符合规定的product属性值改为**hypothetical protein**。可以提取不符合规定的product属性值用于product的filter。

##### 4.2.3.3.2. sample.stats/sample.val文件中的报警**WARNING**
部分WARNING也需要修复；比如**SEQ_FEAT.SeqFeatXrefNotReciprocal**和**SEQ_FEAT.DuplicateFeat**虽然是在WARNING-level messages中，但上传给NCBI后收到邮件让修改后重新提交。

1. **SEQ_FEAT.SeqFeatXrefNotReciprocal**
   - exon和CDS的位置或数量不一致且有中间终止密码子导致
   - 可以调整exon和CDS的位置和数量；可能引入**SEQ_FEAT.InternalStop** 和 **SEQ_INST.StopInProtein** 或 **SEQ_FEAT.ShortIntron** 错误，若引入则在gene行的第九列添加`pseudo=true`属性值；也可以直接添加`pseudo=true`属性值。
   - 实践发现一个例子，**SEQ_FEAT.SeqFeatXrefNotReciprocal**的数量与**SEQ_FEAT.CDSmRNAmismatchCount**和**SEQ_FEAT.CDSmRNAMismatchLocation**一样，上传给NCBI后收到邮件让修改**SEQ_FEAT.CDSmRNAmismatchCount**和**SEQ_FEAT.CDSmRNAMismatchLocation**。这种情况修改exon和CDS的位置和数量很可能导致**SEQ_FEAT.InternalStop** 和 **SEQ_INST.StopInProtein**错误，所以建议直接在gene行的第九列添加`pseudo=true`属性值。
2. **SEQ_FEAT.DuplicateFeat**
   - 当gff在同样的位置注释了多个gene时。
   - 需要删除其中一个，或者修改重复gene注释为一个gene的多个可变剪切的注释。
3. **SEQ_FEAT.GeneXrefStrandProblem**
   - 基因的CDS或exon位置信息不一致时，可能报错
   - 修改位置信息为一致，或者在gene行的第九列添加`pseudo=true`属性值；

##### 4.2.3.3.3. sample.dr文件中的报错FATAL
1. **FATAL**
`cat sample.dr |grep "FATAL"`查看FATAL报错信息，如果物种不是细菌则可以忽略BACTERIAL_开头的报错信息；CONTAINED_CDS开头的信息不影响提交，也可暂时忽略。

2. **FATAL: SUSPECT_PRODUCT_NAMES**
如果product属性值中有错误格式会报错，比如含有不完整的括号，会显示**FATAL: SUSPECT_PRODUCT_NAMES: 9 features contain unbalanced brackets or parentheses**，需要处理(添加括号或者删除括号)。

3. **GENES_OPPOSITE_STRANDS**
- 在sample.dr文件里搜索“DUP_GENES”，如果得到下面的信息，代表有24个基因重复注释到了相同位置的正反链。
- 类似**SEQ_FEAT.DuplicateFeat**报错，但是是重复注释到正反链。
> DUP_GENES_OPPOSITE_STRANDS: 24 genes match other genes in the same location, but on the opposite strand
- 这条信息后面会给出24个基因的位置，删除重复的12条注释，或者修改重复gene注释为一个gene的多个可变剪切的注释。

### 4.2.4. 在线提交
根据上一个部分**提交基因组到NCBI的GenBank**的步骤提交sqn文件到NCBI

### 4.2.5. 提交后修改
提交给NCBI后会收到邮件通知处理进度，如果有错误，会在提交网站显示Error提交状态，并收到需要更改的邮件。根据邮件提示修正注释错误，重复table2asn生成sqn文件和修正错误的步骤直至没有错误在NCBI修正错误(提交新的sqn文件)。

下面记录了我遇到的邮件中提到的错误和修改方案：

1. **SEQ_FEAT.SeqFeatXrefNotReciprocal**
实践经验1
- exon和CDS的位置或数量不一致且有中间终止密码子导致。
- 可以调整exon和CDS的位置和数量；可能引入**SEQ_FEAT.InternalStop** 和 **SEQ_INST.StopInProtein**错误；若引入则在gene行的第九列添加`pseudo=true`属性值；

实践经验2
- exon和CDS的位置或数量不一致且有中间终止密码子导致。同时存在数量一致的**SEQ_FEAT.CDSmRNAmismatchCount**和**SEQ_FEAT.CDSmRNAMismatchLocation**。
- 在gene行的第九列添加`pseudo=true`属性值后，**SEQ_FEAT.SeqFeatXrefNotReciprocal**和**CDSmRNAMismatchLocation**消失并转化成ERROR水平信息**SEQ_FEAT.CDSwithMultipleMRNAs**，数量与**SEQ_FEAT.CDSmRNAmismatchCount**一致。
- 调整exon和CDS的位置和数量，并且在gene行的第九列添加`pseudo=true`属性值。

2. **SEQ_FEAT.DuplicateFeat**
- 这个错误是同一位置注释到多个基因引起的，可以提取邮件中的gene ID(重复基因的第二个)信息，保存成gene ID list，然后用命令`grep -v -f geneID.list sample.gff >sample.revised.gff`删除list里的注释信息。
- 如果邮件里没有gene ID信息，可以用`cat sample.val |grep DuplicateFeat`里查看错误的位置信息，根据位置信息查找gff的重复注释位置，并删除其中一个基因注释。
- 除了直接删除重复的整个基因注释外，还可以把重复基因的注释变成一个基因的多个可变剪切注释(删除重复基因的gene注释，把重复基因的mRNA注释的parent属性改为另一个基因)。

3. **Discrepancy**
Discrepancy.txt文件中保存了一些不符合标准的报错，比如product属性值中有横杠起始的元素，或者括号不完全，根据Discrepancy.txt文件指出的错误一条条修改。

`cat sample.dr |grep "FATAL"`查看FATAL报错信息，如果物种不是细菌则可以忽略BACTERIAL_开头的报错信息；CONTAINED_CDS开头的信息不影响提交，也可暂时忽略。
如果product属性值中有不完整的括号，会显示**FATAL: SUSPECT_PRODUCT_NAMES: 9 features contain unbalanced brackets or parentheses**，需要处理需要处理(添加括号或者删除括号)。

## 4.3. 提交状态
在提交网页，提交后会显示提交状态，经过一轮机器自动验证和两轮人工检查后可公开数据。

提交状态含义：
1. Unfinished
- 提交未完成
- 提交如果没有最后确认，会保存已填写内容，随时可以登陆账号点击未完成的提交号继续提交。
   
2. Queued
- 提交在等待自动验证
- 刚提交的任务会进入自动验证环节

3. Error
- 错误
- 有文件有错误需要修正或者再次提交；会提供错误具体信息的文件和邮件通知。

4. Processing
- 处理中
- 提交通过自动验证，等待NCBI员工手动检查

5. Processing and accession number present
- 处理中，并且显示了序列号
- 提交通过了NCBI员工的首次检查，等待NCBI员工的最终检查
- 如果提交有问题最终检查员工会发邮件告知
- 如果指定了释放日期，提交会在释放日期前再做最终检查；也就是说释放日期前都保持在这个提交状态

6. Processed
- 已处理
- 数据已被公开释放，在这之前做的任何更改都会包含在释放数据中，但在提交网站不会显示修改。

# 5. references
1. genomes submission guide：https://www.ncbi.nlm.nih.gov/genbank/genomesubmit/
2. table2asn：https://www.ncbi.nlm.nih.gov/genbank/table2asn/
3. table2asn documentation：https://ftp.ncbi.nlm.nih.gov/asn1-converters/by_program/table2asn/DOCUMENTATION/
4. Validation and Discrepancy Report Error Explanations：https://www.ncbi.nlm.nih.gov/genbank/validation/
5. Validation Error Explanations for Genomes：https://www.ncbi.nlm.nih.gov/genbank/genome_validation/
6. Discrepancy Report：https://www.ncbi.nlm.nih.gov/genbank/asndisc/
7. MODULE valid：https://github.com/genome-vendor/sequin/blob/master/errmsg/valid.msg
8. annotating genomes with gff3：https://www.ncbi.nlm.nih.gov/genbank/genomes_gff/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>