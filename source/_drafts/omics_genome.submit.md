---
title: genome submit
date: 2022-01-30 14:30:00
categories:
- bio
- bioinfo
tags:
- genome submit

description: 记录基因组提交到NCBI的genebank的步骤和注意事项。
---

<div align="middle"><music URL></div>



# gff文件准备
## 功能注释
先把所有功能注释转换成单基因单注释的格式(单基因多行注释)，确保数据只有两列(基因ID列和功能注释列)，然后合并merge，排序sort，筛选filter，去重uniq等处理，按照基因ID排序，生成单基因多行注释的merge.anno文件

用`cat merge.anno |awk -F"\t" '{a[$1]=a[$1]$2","}END{for( i in a){print i"\t"a[i]}}' |sed "s/,$/;/g"|sort -gk 1.3n|awk '{print $1"\tproduct="$2}' >product.list`命令从merge.anno生成product.list，每个基因单行的格式，还是两列，第一列基因ID，第二列若有多个注释用逗号隔开，用分号结尾。

sample.gff与product.list合并，把结构注释和功能注释文件合并，功能注释以product属性值的形式加入gff文件的第九列内容。



# 基因组提交
## 注册新的BioProject和BioSample
在网站[submit](https://submit.ncbi.nlm.nih.gov/subs/)提交新的BioProject和BioSample，根据项目和样品信息，计划发表的信息填写，勾选自动生成Locus Tag Prefixes。提交后马上可以获得BioProject ID(PRJNAxxxxxx)和BioSample ID(SAMNxxxxxxxx)，以及Locus Tag Prefixes(xxxxx)三个编号信息用于genome的提交填写。

## table2asn
table2asn软件用于把基因组和其他信息转换成NCBI认可的sqn格式的一个文件。

### 安装table2asn

### 运行table2asn生成sample.sqn

在网站[template](https://submit.ncbi.nlm.nih.gov/genbank/template/submission/)提交基因组信息和BioProject ID(PRJNAxxxxxx)和BioSample ID(SAMNxxxxxxxx)，生成template.sbt文件。



nohup table2asn -t template.sbt -indir ./ -M n -Z -locus-tag-prefix L6164 -gaps-min 1 -gaps-unknown 100 -l paired-ends -j "[organism=Bauhinia variegata][strain=pink_petal]" -logfile bv.log &

-t指定template.sbt文件
-indir指定存放genome.fsa和genome.tbl/genome.gff文件的目录
-M n
-Z
-locus-tag-prefix在向NCBI申请BioProject(以及BioSample)后会分配一个locus-tag-prefix前缀，在这里指定，不指定可能在sample.dr中报错FATAL: INCONSISTENT_PROTEIN_ID。
-gaps-min 1 -gaps-unknown 100指定序列中的N的含义，如未正确指定，可能在submit.stats中报错SEQ_INST.InternalNsInSeqRaw。-gaps-min 指定代表gap的连续Ns的最小数量，1指把大于等于1个的连续Ns碱基都转换成assembly_gaps。-gaps-unknown 指定代表未知长度的gap的连续Ns的
-l paired-ends
-j "[organism=Bauhinia variegata][strain=pink_petal]"用于指定样本名[organism]和品种名[strain]，不指定可能在sample.stats中报错SEQ_DESCR.BioSourceMissing和SEQ_DESCR.NoSourceDescriptor。
-logfile bv.log指定log文件，如果运q行有错误存放在log文件

-i sample.fsa -f sample.gff参数有基因组的最大限制（大概是2G内），如果超出限制可以用-indir ./指定存放目录（需要sample.fsa和sample.gff有相同前缀且存放在同一目录下）来代替-i -f指定两个文件便可以运行了。

可以在[MODULE valid](https://github.com/genome-vendor/sequin/blob/master/errmsg/valid.msg)查看具体报错的含义。

- 当gff的CDS位置有中间终止密码子时，sample.stats种报错**SEQ_FEAT.InternalStop** 和 **SEQ_INST.StopInProtein**。可以修改CDS序列位置，去除中间终止密码子。如果无法去除中间终止密码子，可以在sample.gff文件对应的gene(第三列为gene的那行)的第九列添加`pseudo=true`属性值代表序列无法被翻译。
- - 当gff的CDS位置无终止密码子时，sample.stats种报错**SEQ_FEAT.NoStop**。可以延长CDS序列位置，直至出现终止密码子。如果无法找到终止密码子，可以在sample.gff文件对应的gene(第三列为gene的那行)的第九列添加`pseudo=true`属性值代表序列无法被翻译。

添加属性值的命令行`sed -i '/ID=Ane00639;/ {s/$/pseudo=true;/g}' sample.gff`

- 当gff的第9列的属性值attributes中包含不符合规定的字符，例如"|"符号时，sample.stats中报错**SEQ_FEAT.BadInternalCharacter**。
- 当gff的第9列的属性值attributes中以连字符"-"结尾时，sample.stats中报错**SEQ_FEAT.BadTrailingHyphen**。
- 当gff的第9列的属性值attributes中以连字符":"结尾时，sample.stats中报错**SEQ_FEAT.BadTrailingCharacter**。
- 当gff的product属性不符合蛋白产物名称规定时，会生成Adiantum.nelumboides.fixedproducts文件，记录把不符合规定的product属性值改为**hypothetical protein**。可以提取不符合规定的product属性值用于product的filter。
- 当gff注释的一个基因的exons位置相邻时，sample.stats中报错**SEQ_FEAT.AbuttingIntervals**。可以合并两个相邻的exons，更改位置。
- 当gff的exon位置有重叠时，sample.stats种报错**SEQ_FEAT.SeqLocOrder**。可以参考mRNA和CDS序列的位置更改重叠的exon位置；exons的边界和mRNA一致，数量和CDS一致。
- 当gff注释CDS和exon的位置或数量不一致时，sample.stats中报错**SEQ_FEAT.CDSmRNAXrefLocationProblem**或者**SEQ_FEAT.CDSwithMultipleMRNAs**，可以更改exon或CDS位置和数量。
- 当gff在同样的位置注释了多个gene时，sample.stats中WARNING-level messages报错**SEQ_FEAT.DuplicateFeat**，需要删除一个。
- SEQ_FEAT.ShortIntron表示含有短于11bp的内含子，可以修正intron的位置使它长于11bp，或者在gene那行第九列添加`pseudo=true`属性值，或者在table2asn的命令中中加上参数`-c s`代表标记为"LOW QUALITY PROTEIN"。





### 提交后修改
提交给NCBI后会收到邮件通知处理进度，如果有错误，有可能收到需要更改的邮件。

1. **SEQ_FEAT.SeqFeatXrefNotReciprocal**

在sample.stats的WARNING-level messages中**SEQ_FEAT.SeqFeatXrefNotReciprocal**，在gene行的第九列添加`pseudo=true`属性值；

更改exon或CDS位置和数量有时可行但可能引入**SEQ_FEAT.InternalStop** 和 **SEQ_INST.StopInProtein**错误，引入后还是需要添加`pseudo=true`属性值。

直接添加`pseudo=true`属性值可能转化成**SEQ_FEAT.CDSwithMultipleMRNAs**错误。

其中**SEQ_FEAT.SeqFeatXrefNotReciprocal**和**SEQ_FEAT.DuplicateFeat**虽然是在WARNING-level messages中，但上传给NCBI后会发邮件让修改后重新提交。

同一个mRNA包含的exons的数量和位置与CDSs几乎一致，但同一个mRNA的exons的起始和终止位置(第一个exon的首和最后一个exon的尾)与mRNA的起始和终止相同。


2.**SEQ_FEAT.DuplicateFeat**
这个错误是同一位置注释到多个基因引起的，可以提取邮件中的gene ID信息，保存成gene ID list，然后用命令`grep -v -f geneID.list sample.gff >sample.revised.gff`删除list里的注释信息。


3. **Discrepancy**
Discrepancy.txt文件中保存了一些不符合标准的报错，比如product属性值中有横杠起始的元素，或者括号不完全。

`cat sample.dr |grep "FATAL"`查看FATAL报错信息，如果物种不是细菌则可以忽略BACTERIAL_开头的报错信息；CONTAINED_CDS开头的信息不影响提交，也可暂时忽略。
如果product属性值中有不完整的括号，会显示**FATAL: SUSPECT_PRODUCT_NAMES: 9 features contain unbalanced brackets or parentheses**，需要处理，


### 添加product属性值
最好的办法是添加product属性值的时候做好filter。


去掉|和-等奇怪符号的命令：
sed -i -E -e "s/=[GBEMDJ]+\|[^,]+,/=/g" -e "s/=[GBEMDJ]+\|[^;]+;/=delete;/g" -e "s/,[GBEMDJ]+\|[^,]+,/,/g" -e "s/,[GBEMDJ]+\|[^;]+;/;/g" -e "s/,Protein,/,/ig" -e "s/=protein,/=/ig" -e "s/,protein;/;/ig" -e "s/=protein;/=delete;/ig" -e "s/-,/,/g" product.txt


sed -i -E -e "s/,//g" -e "s/^-//g" -e "s/\/$//g" -e "s/[^a-zA-Z]+//g"

product的一些规则
- 逗号（因为逗号是分隔符）
- -开头/结尾
- /开头/结尾的
- 括号（包括圆括号和中括号）不完整的
- 不包含字母


# references
[genomes submission guide](https://www.ncbi.nlm.nih.gov/genbank/genomesubmit/)
[table2asn](https://www.ncbi.nlm.nih.gov/genbank/table2asn/)
[table2asn documentation](https://ftp.ncbi.nlm.nih.gov/asn1-converters/by_program/table2asn/DOCUMENTATION/)
[Validation Error Explanations](https://www.ncbi.nlm.nih.gov/genbank/genome_validation/#NoStop)
[annotating genomes with gff3 or gtf files](https://www.ncbi.nlm.nih.gov/genbank/genomes_gff/)