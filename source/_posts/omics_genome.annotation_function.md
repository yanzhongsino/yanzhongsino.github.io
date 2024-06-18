---
title: 基因组注释（三）：基因功能注释
date: 2021-05-17 10:36:00
categories: 
- omics
- genome
- annotation
tags: 
- genome
- genome annotation
- functional annotation
- eggnog-mapper
- interproscan
- PANNZER2
- Mercator4
- GFAP

description: 记录对基因组的基因进行功能注释的方法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1842025914&auto=1&height=32"></iframe></div>


基因功能的注释依赖于上一步的基因结构预测，根据预测结果从基因组上提取翻译后的**蛋白序列**和主流的数据库进行比对，完成功能注释。(有些也支持用DNA序列做功能注释的，但比较少。)

比较推荐的方案是：eggNOG-mapper网页注释和PANNZER2网页注释，如果有余力再加上interpro的本地注释。

功能注释的一般流程：1.选择数据库并下载，2.构建blastp索引，3.用blastp比对蛋白序列到数据库，4.结果整理。

有些软件集成了数据库和功能注释的多个流程，甚至提供服务器用于网页版注释，例如eggNOG-mapper和interproscan分别集成了eggNOG和interpro数据库，PANNZER2使用最新的UniProt数据库实现网页注释。

# 1. 数据库概述
根据数据库中已知编码基因的注释信息（包括motif、domain），基于同源比对，对基因中的模序和结构域、新基因编码的蛋白质功能、所参与的信号传导通路和代谢途径等的预测。基因组注释内容还可涉及蛋白激酶、病原与宿主互作、致病毒力因子预测、抗性基因等。
常用数据库：
- Nr：NCBI官方非冗余蛋白数据库，包括PDB, Swiss-Prot, PIR, PRF; 如果要用DNA序列，就是nt库。
- UniProt：分为两部分。Swiss-Prot是UniProt数据库中经过人工校正的高质量蛋白数据库，蛋白序列得到实验的验证。TrEBL是UniProt数据库中未经过人工校正、机器自动注释的蛋白数据库。
- eggNOG：EMBL创建，是对NCBI的COG数据库的拓展，提供了不同分类水平蛋白的直系同源分组（Orthologous Groups，OG）。
- InterPro：EBI开发的一个整合的蛋白家族功能注释数据库，包括Gene3D、CDD、Pfam等10几个数据库。
- Pfam: 蛋白结构域注释的分类系统。

- GO: 基因本体论注释数据库。
- KEGG: 代谢通路注释数据库。

数据库的下载非常耗时，不管是eggnog还是interproscan，linux下几十KB/s，window下几百KB/s(需科学上网)，耗时长度以天计算，可以尝试先在window系统下载，再转移到linux下。

## 1.1. Nr/Nt数据库
[nr数据库](ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nr.gz)；
[nt数据库](ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nt.gz)；

## 1.2. UniProt数据库（Swiss-Prot和TrEMBL）
UniProt数据库包括人工校正的高质量数据库Swiss-Prot和软件自动注释的TrEMBL数据库。

[Swiss-Prot数据库](ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz)：高质量，手工注释，非冗余的蛋白质序列数据库。非常可靠。

[TrEMBL数据库](ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_trembl.fasta.gz)：自动注释的蛋白质序列数据库。

UniProt将完整数据库分类拆分成几个[子库](ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/)。包括古菌，真菌，植物，人，哺乳动物，脊椎动物，无脊椎动物，啮齿动物等。

## 1.3. GO数据库
[GO数据库](http://geneontology.org/docs/download-ontology/）

## 1.4. KEGG数据库
KEGG(Kyoto Encyclopedia of Genes and Genomes)京都基因和基因组百科全书，是研究Pathway代谢通路最主要的数据库，整合了基因组、化学、系统及疾病和健康的信息。

## 1.5. COG直系同源蛋白数据库
[COG数据库](ftp://ftp.ncbi.nlm.nih.gov/pub/COG/COG2014/data/):NCBI开发的用于同源蛋白注释的数据库。
fun2003-2014.tab是COG的分类信息，将COG的功能分为26个类别，每个类别用一个字母表示。
cognames2003-2014.tab是COG的详细信息，包括编号，对应的分类，功能描述等。
cog2003-2014.csv是蛋白和COG对应关系。
prot2003-2014.fa.gz是fa格式的蛋白序列和注释信息。

## 1.6. KOG数据库
KOG（Clusters of orthologous groups for eukaryotic complete genomes）真核生物蛋白相邻类的聚簇。构成每个KOG的蛋白都是被假定来自一个祖先蛋白，要么是orthologs，要么是paralogs。orthologs是指来自不同物种的由垂直家系（物种形成）进化而来的蛋白，并且保留与原始蛋白相同的功能；paralogs是指在一定物种中来源于基因复制的蛋白，可能会进化成新的与原来有关的功能。
[KOG数据库](ftp://ftp.ncbi.nlm.nih.gov/pub/COG/KOG/kyva)

## 1.7. eggNOG数据库
eggNOG（evolutionary genealogy of genes：non-supervised orthologous groups）由EMBL创建，是对NCBI的COG数据库的拓展，提供了不同分类水平蛋白的直系同源分组（Orthologous Groups，OG），包括真核生物，原核生物，病毒的信息。拓展了COG的分类，采用无监督聚类算法在全基因组范围内推导基因功能，更适合谱系特征基因的分析。

[eggNOG数据库](http://eggnog5.embl.de/)中，e5.proteomes.faa是所有的蛋白组序列，e5.taxid_info.tsv是taxid对应的物种名及完整的谱系信息，e5.og_annotations.tsv是所有NOG group信息（第一列为Taxid，第二列为NOG groups，第三列为COG归属，第四列为Function）。per_tax_level下不同taxonomy level的members.tsv文件分别是相应level的蛋白序列id和NOG group的对应关系（第一列taxid，第二列NOG groups，第三列为该NOG group所包含的蛋白序列数目，第四列为该NOG group所包含的物种数目，第五列是该NOG group所包含的蛋白序列id，第六列是该NOG group所包含的物种taxid。

## 1.8. pfam数据库
[pfam数据库](http://pfam.xfam.org/)是蛋白质家族数据库，根据多序列比对结果和隐马尔科夫模型，将蛋白分成不同家族。


## 1.9. string数据库
[string数据库](https://string-db.org)。
搜寻已知蛋白质和预测蛋白质之间的相关关系的系统，包括蛋白质之间的物理作用和间接的功能相关性。基于染色体临近，系统进化谱，基因融合和基因芯片数据等计算基因或蛋白的共表达。
包括动物转录因子数据AnimalTFDB3.0，植物转录因子数据[PlnTFDB](http://plntfdb.bio.uni-potsdam.de/v3.0/)，真菌转录因子数据库[Fungal TFDB](http://ftfd.snu.ac.kr/index.php?a=view)。

植物转录因子数据库收录了大部分植物模式物种共20个种的84个转录因子家族，包含28193 protein models，26184 distinct protein sequences，支持在线blast和本地blast。
分析工具：[iTAK](http://itak.feilab.net/cgi-bin/itak/index.cgi)，软件内置了PlantTFDB的数据库，可直接用于预测植物转录因子。
结果在classification.txt文件中。

# 2. 基因功能注释

notes: 注释分析中要保证蛋白序列中没有代表氨基酸字符以外的字符，比如说有些软件会把最后一个终止密码子翻译成"."或者"*"，需要删除。

## 2.1. 下载数据库手动注释--以Uniprot为例

### 2.1.1. 下载数据库
这里以Uniprot为例，下载其他数据库后可用同样方法进行手动注释。

可以uniprot_sprot和uniprot_trembl的植物数据库合并在一起。
`wget ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/uniprot_sprot_plants.dat.gz #`;
`wget ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/uniprot_trembl_plants.dat.gz # 11.5G`;
`zcat uniprot_sprot_plants.dat.gz uniprot_trembl_plants.dat.gz > uniprot_plants.dat`;

转换格式：
用python的SeqIO模块转换
```python
from Bio import SeqIO
count = SeqIO.convert("uniprot_plants.dat","swiss","uniprot_plants.fa","fasta")
print("Converted %i records" % count)
```

### 2.1.2. 构建blastp索引
`makeblastdb -in uniprot_plants.fasta -dbtype prot -title uniprot_plants -parse_seqids`
或者更快的
`diamond makedb --in uniprot_plants.fasta -d uniprot_plants`

### 2.1.3. 用blastp比对蛋白序列protein.fa和数据库uniprot_plants
`blastp -query protein.fa -out uniprot_plants.xml -db uniprot_plants.fasta -evalue 1e-5 -outfmt 6 -num_threads 24 &`
或者更快的
`diamond blastp -d uniprot_plants.dmnd -q proteins.fasta --evalue 1e-5 > blastp.outfmt6`

### 2.1.4. 比对结果中筛选每个query的最佳subject
jcvi可用conda安装
`python -m jcvi.formats.blast best -n 1 blastp.outfmt6`
生成blastp.outfmt6.best

### 2.1.5. 使用add_annotation_from_dat.py根据blastp输出从dat提取GO/KEGG/同源基因
python add_annotation_from_dat.py blastp.outfmt6.best uniprot_plants.dat
生成swiss_annotation.tsv文件；包含以下几列。
- gene id
- uniprot accession
- identity
- homology species
- EnsemblPlants
- GO ID
- GO component, CC/MF/BP
- evidence

参考脚本[add_annotation_from_dat.py](https://github.com/xuzhougeng/myscripts/blob/master/annotation/add_annotation_from_dat.py)

### 2.1.6. blast2go
blast2go的功能是将blast注释结果转换成GO注释，但使用较为复杂，下载数据库耗时长，不推荐使用。
[blast2go教程](https://www.plob.org/article/1299.html)；[blast2go教程](https://www.jianshu.com/p/ad44b2a837c9)
下载数据库
wget ftp://ftp.ncbi.nlm.nih.gov/gene/DATA/gene_info.gz #650M
wget ftp://ftp.ncbi.nlm.nih.gov/gene/DATA/gene2accession.gz #2.3G
wget ftp://ftp.pir.georgetown.edu/databases/idmapping/idmapping.tb.gz #10G
wget http://release.geneontology.org/2017-01-01/mysql_dumps/go_monthly-assocdb-data.gz #6.3G

## 2.2. eggnog-mapper注释
eggNOG(evolutionary genealogy of genes: Non-supervised Orthologous Groups)。
[eggNOG数据库](http://eggnog5.embl.de/download/emapperdb-5.0.0/)是数据库，eggnog-mapper是使用eggnog数据库作为参考的注释软件或网页，[eggnog-mapper网页版](http://eggnog-mapper.embl.de/)。
### 2.2.1. eggNOG本地注释

下载数据库本地注释的方法：
在eggnog网站下载对应使用的数据库。
http://eggnog5.embl.de/download/emapperdb-5.0.0/eggnog.db.gz
http://eggnogdb.embl.de/download/emapperdb-5.0.0/eggnog_proteins.dmnd.gz
注释命令：
python2 /home/leon/software/3.Function/eggnog-mapper.orginal/emapper.py -i Tov.final.all.maker.transcripts.fasta -o run_egg --override  -m diamond  --cpu 80

### 2.2.2. eggNOG-mapper网页版注释

#### 2.2.2.1. 注释原理

注释分为三步：
1. 序列比对
用hmmer3对所有蛋白序列在eggNOG数据库中搜索，找到最佳匹配的HMM；再用phmmer在最佳匹配的HMM对应的一组eggNOG蛋白中进一步搜索，每条蛋白序列的最佳匹配结果以seed orthlog形式存放。
除了HMMER3还可选DIAMOND直接对所有eggNOG蛋白序列进行搜索，速度更快，若数据大或在eggNOG搜集的物种库中有近缘种可选择DIAMOND。
2. 推测直系同源基因
基于预分析的eggNOG进化树数据库，提取最佳匹配seed orthlog的蛋白的一组更精细的直系同源基因，根据bit-score或E-value对结果过滤，剔除同源性不高的结果。
3. 功能注释
用于搜索的蛋白序列对应的直系同源基因的功能描述就是最终的注释结果。比如说GO, KEGG, COG等。

#### 2.2.2.2. 注释
推荐使用[eggnog-mapper网页版](http://eggnog-mapper.embl.de/)注释，避免下载数据库的繁琐。限制10万条蛋白序列，数量超过可以切分后分开注释。

只需要上传氨基酸序列，点击提交；然后**在收到的邮件里点击链接，开始任务**即可。

#### 2.2.2.3. 注释结果
eggnog-mapper会生成五个文件:
- [project_name].emapper.gff: gff注释文件。
- [project_name].emapper.orthologs: 搜索匹配到的物种和orthologs的信息。
- [project_name].emapper.seed_orthologs: 记录每个用于搜索序列对的的最佳的OG，也就是
- [project_name].emapper.annotations.tsv: 该文件提供了最终的注释结果。大部分需要的内容都可以通过写脚本从从提取，一共有13列。
- [project_name].emapper.annotations.xlsx：最终注释结果，另一种格式。

其中[project_name].emapper.annotations.tsv每一列对应的记录如下：
1. query: 检索的基因名或者其他ID
2. seed_ortholog: eggNOG中最佳的蛋白匹配
3. evalue: 最佳匹配的e-value
4. score: 最佳匹配的bit-score
5. eggNOG_OGs：匹配到的eggNOG数据库里的OGs
6. max_annot_lvl：
7. COG_category：COG类别
8. Description：功能描述
9. Preferred_name: 预测的基因名，特别指的是类似AP2有一定含义的基因名，而不是- AT2G17950这类编号
10. GOs: 推测的GO的词条， 未必最新
11. EC：
12. KEGG_ko: 推测的KEGG KO词条， 未必最新
13. KEGG_Pathway：推测的KEGG通路词条
14. KEGG_Module：
15. KEGG_Reaction：
16. KEGG_rclass：
17. BRITE:
18. KEGG_TC:
19. CAZy
20. BiGG_Reaction: BiGG代谢反应的预测结果
21. PFAMs:注释到PFAM数据

#### 2.2.2.4. 结果整理
- `cat eggnog.web/*.emapper.annotations.tsv |sed '/#.*/d' |cut -f1,8,9 |awk -F"\t" '{print $1"\t"$3": "$2}' |sed '/-: -/d'|sort -k 1.3n |uniq |sed '1i\gene\teggnog.preferred.name: description' >eggnog.anno` 提取信息列：1（query），8（Description），9（Preferred name），10（GOs），12（KEGG_ko）列。然后提取1（query），8（Description），9（Preferred name）注释信息。eggnog注释是每个基因一行信息，所以eggnog.anno的行数就是注释到的基因数量。
- `cat eggnog.web/*.emapper.annotations.tsv |sed '/#.*/d' |cut -f1,10 |sed '/\t-$/d' |sort -k 1.3n |uniq |sed '1i\gene\teggnog.go' >eggnog.go.anno` 提取信息列：1（query），10（GOs）的eggnog的GO注释信息。
- `cat eggnog.web/*.emapper.annotations.tsv |sed '/#.*/d' |cut -f1,12 |sed '/\t-$/d' |sort -k 1.3n |uniq |sed '1i\gene\teggnog.kegg' >eggnog.kegg.anno` 提取信息列：1（query），12（KEGG_ko）的eggnog的KEGG注释信息。

## 2.3. InterProScan
### 2.3.1. Interproscan
interpro是数据库，interproscan是可以使用interpro的注释软件。
如果序列少，可以使用[interproscan网页版](http://www.ebi.ac.uk/interpro/search/sequence/)注释，避免下载数据库的繁琐。限制一次注释一条，长度小于40000个蛋白序列，超过可以切分后分开注释。

interproscan可实现多个数据库同时注释，包括：
- InterPro注释
- Pfam数据库注释(可以通过hmmscan搜索pfam数据库完成)
- PANTHER数据库注释
- GO注释(可以基于NR和Pfam等数据库，然后BLAST2GO完成)
- Reactome通路注释，不同于KEGG
还有很多数据库。

### 2.3.2. Interproscan的安装和数据库下载

如果注释的文件比较大或者比较多，可以下载本地版注释，下载过程非常慢。根据你的linux版本和发布日期来选择最适版本，软件很大最近版大约9.1G。建议下载对应的md5文件，用 md5sum -c *.md5来检查下载的是否完全。

#### 2.3.2.1. Interproscan安装
interproscan软件地址：ftp://ftp.ebi.ac.uk/pub/software/unix/iprscan/5/, 选择最新版本，比如5.47-82.0，则`wget ftp://ftp.ebi.ac.uk/pub/software/unix/iprscan/5/5.47-82.0/*`，版本号目录下有两个文件，一个软件的tar.gz压缩文件，一个对应的md5文件，都下下来。

#### 2.3.2.2. 软件自带数据库
软件本身自带了很多数据库，不需要安装，5.47-82.0版本带有CDD-3.17,Coils-2.2.1,Gene3D-4.2.0,Hamap-2020_01,MobiDBLite-2.0,PANTHER-15.0,Pfam-33.1,PIRSF-3.10,PRINTS-42.0,ProSitePatterns-2019_11,ProSiteProfiles-2019_11,SFLD-4,SMART-7.1,SUPERFAMILY-1.75,TIGRFAM-15.0数据库。

#### 2.3.2.3. 关于PANHTER数据库
PANHTER(Protein ANalysis THrough Evolutionary Relationships) 数据库是Gene Ontology Phylogenetic Annotation Project的一部分。
1. 发现interproscan 5.47-82.0软件自带PANHTER数据库，所以不用单独下载啦。
2. 若是没有带PANHTER数据库，则下载构建本地PANHTER数据库，需要下载并解压到软件安装目录下的 /path of interproscan/data/。
- PANHTER下载地址：ftp://ftp.ebi.ac.uk/pub/software/unix/iprscan/5/data/,
- 选择最新版本，比如14.1，则`wget ftp://ftp.ebi.ac.uk/pub/software/unix/iprscan/5/panther-data-14.1.tar.gz*`，下载14.1和对应的md5文件。
- md5验证两个文件下载完全后，tar zxf解压缩，并把PANHTER数据库放进interproscan的data目录下。

#### 2.3.2.4. java和python版本检查
软件的运行依赖java11和python3，`java -version`和`python --version`检查版本是否正确。

#### 2.3.2.5. 测试安装
解压缩后运行 interproscan.sh 测试是否成功安装，弹出help界面就是成功安装，用测试文件运行下，`interproscan.sh -i test_proteins.fasta`，没报错则软件可正常运行。

#### 2.3.2.6. tips
- 如果本地化运行，可以把interproscan.properties文件里的[这行代码](precalculated.match.lookup.service.url=http://www.ebi.ac.uk/interpro/match-lookup)注释掉，就不进行match.lookup检查，这个应该是更新数据库的检查。
- 如果不注释，在运行时存储interproscan数据库端网络慢时（大部分情况下）会输出一些Connection timed out和ERROR - Lookup version check failed的报错信息，但the analysis will continue to run locally.本地运行仍在继续。

### 2.3.3. Interproscan运行
常用运行参数
`interproscan -i pep.fa -b out.iprscan -goterms -iprlookup -pa -dp -cpu 24`

interproscan的参数：
- -i,--input 输入文件，一般要为fasta格式，注意不要带有除氨基酸符号的其他特殊符号（比如代表终止密码子的*），gene/氨基酸的名称不能为空。
- -b,--output-file-base 指定输出文件的路径和文件名，默认是输入文件的路径
- -appl,--applications 指定使用Interpro中哪些数据库，默认使用全部数据库
- -f,--formats 用于指定输出文件的后缀，蛋白序列默认输出TSV, XML and GFF3
- -t,--seqtype 输入文件的序列类型，p为protein，n为dna/rna，默认是p
- -cpu 指定使用线程数
- -goterms 打开GO注释，依赖iprlookup参数，switch on lookup of corresponding Gene Ontology annotation (IMPLIES -iprlookup option)
- -iprlookup Also include lookup of corresponding InterPro annotation in the TSV and GFF3 output formats.
- -pa Optional, switch on lookup of corresponding Pathway annotation (IMPLIES -iprlookup option)

- -dp 关闭precalculated match lookup service，默认的是开启。根据md5值来快速检验这上传的数据是否已经被注释了，如果是已经注释了就直接出结果。

### 2.3.4. Interproscan结果解释

生成的out.tsv和out.gff3等是不同格式的注释文件，这里使用tsv格式。

tsv格式文件每一列含义：
1. 蛋白名字
2. 序列MD5 disest
3. 序列长度
4. 所用的库（Pfam/PANTHER/RPINTS etc）
5. 匹配的数据编号（PF00001/PTHR00001 etc)
6. 功能描述
7. 起始位置
8. 终止位置
9. e-value得分
10. 匹配的状态T: true
11. 日期
12. interPro 注释编号
13. interPro 注释描述
14. GO注释
15. Pathways 注释

每行是一条蛋白序列注释到的每个数据库的每一个匹配，所以每个基因会有多行。
根据第四列-数据库名称来提取不同数据库的注释结果。这里提取pfam和PANTHER两个数据库的结果。

### 2.3.5. Interproscan结果整理
1. 提取interpro数据库注释内容并转化成单基因单行格式
`cat *.iprscan.tsv |cut -f1,12,13|grep "IPR"|awk '{print $1"\t"$2": "$3}'|sort -k 1.3n|uniq |awk -F"\t" '{a[$1]=a[$1]$2"; "}END{for( i in a){print i"\t"a[i]}}'|sed "s/; $//g" |sort -k 1.3n |sed '1i\gene\tinterpro' >iprs.interpro.anno`

notes：
- 注意排序和去重。
- 因为我的geneID是tg00001这种格式，只有两个字母，所以sort排序按数字从第三个`sort -k 1.3n`开始排序，可以根据不同geneID更改。
- 其中`awk -F"\t" '{a[$1]=a[$1]$2"; "}END{for( i in a){print i"\t"a[i]}}'|sed "s/; $//g" |sort -k 1.3n`部分是单基因多行注释转化成单基因单行注释的命令。
- 用cut截取1，12，13三列是因为有些行没有12列及之后数据，直接用awk会失效。
- 用grep搜索IPR是因为直接用awk匹配第12列不能匹配完全，比如SUPERFAMILY数据库的比对就匹配不了。

2. 提取Pfam数据库注释结果并转化成单基因单行格式
`cat *.iprscan.tsv  |awk '$4 == "Pfam" {print $1"\t"$5": "$6}' |sort -k 1.3n |uniq|awk -F"\t" '{a[$1]=a[$1]$2"; "}END{for( i in a){print i"\t"a[i]}}'|sed "s/; $//g" |sort -k 1.3n |sed '1i\gene\tinterproscan.pfam'>iprs.pfam.anno`

3. 提取PANTHER数据库注释结果并转化成单基因单行格式
`cat *.iprscan.tsv |awk '$4 == "PANTHER" {print $1"\t"$5": "$6}'|sort -k 1.3n|uniq |awk -F"\t" '{a[$1]=a[$1]$2"; "}END{for( i in a){print i"\t"a[i]}}'|sed "s/; $//g" |sort -k 1.3n |sed '1i\gene\tinterproscan.panther'>iprs.panther.anno`

## 2.4. PANNZER(Protein ANNotation with Z-scoRE) 
### 2.4.1. PANNZER
网站注释：http://ekhidna2.biocenter.helsinki.fi/sanspanz/

- 用于原核和真核蛋白序列的功能注释。
- 使用的Uniprot数据库，与数据库更新一致。
- 用于预测functional description (DE) 和 GO classes。

注释速度：7万条序列4个小时。

### 2.4.2. 结果文件
notes: 
- 网页版的结果，注意需要把进度条拉到最下面再复制保存文本，否则没有显示完全，ctrl+a全选只会选中已显示的内容，未显示的未被选中。
- (20210813)今天发现不需要拉进度条这么繁琐，在网页右键另存为可以保存文本格式的结果。

1. *.anno.out.txt是蛋白序列的注释信息，包含六列：
- qpid：序列ID；
- type：original_DE,qseq（前两种类型是每个基因都有的）,DE,MF_ARGOT,BP_ARGOT,CC_ARGOT,EC_ARGOT等（后面的类型是注释到的基因才有的）；
- score：匹配分数；
- PPV：不知道是啥指标；
- id：数字；
- desc：功能描述；每个序列都有多行信息，没被注释到的序列只有两行第二列type分别为original_DE和qseq的基本信息。

每个基因有只有一行第二列type为DE的数据，此时第六列的描述是对基因的功能描述，即我们想要的基因功能注释信息。

2. *.DE_prediction.txt是注释到DE（functional description）的序列的注释信息。
重要的是第一列（qpid）：基因id；第九列（desc）：DE描述信息，即注释到uniprot数据库的描述信息；第十列（genename）：基因名字。
3. *.GO_prediction.txt是注释到GO的序列的注释信息。
重要的是第一列（qpid）：基因id；第二列（ontology）：GO类别，BP/MF/CC三个类别；第三列（goid）：GO ID；第四列（desc）：GO描述信息。

### 2.4.3. 结果整理
1. `cat pannzer2/anno.out.txt |awk '$2 == "DE" {print $1"\t"$6}' |sort -k 1.3n |uniq |sed '1i\gene\tpannzer2.uniprot' >pannzer2.uniprot.anno`把Uniprot数据库功能注释信息提取出来。

2. `cat pannzer2/GO.out.txt |awk '{print $1"\tGO:"$3,$4}' |awk -F"\t" '{a[$1]=a[$1]$2"; "}END{for( i in a){print i"\t"a[i]}}'|sed "s/; $//g" |sort -k 1.3n |uniq |sed "1s/.*/gene\tpannzer2.go/" >pannzer2.go.anno`把Uniprot数据库的GO注释提取出来并转化成单基因单行信息。

## 2.5. Mercator4——植物基因组功能注释
网站注释：https://www.plabipd.de/portal/web/guest/mercator4/-/wiki/Mercator4/FrontPage

### 2.5.1. 注释原理
- 针对植物基因组的功能注释，以整个植物界的高质量带注释基因组为种子，找直系同源蛋白。
- 基于隐马尔可夫模型（HMM），用HMMER3软件的hmmscan对每条蛋白做HMM测试，然后被分到Bin-50类别下。
- 其余未被分类的蛋白与Swiss-Prot数据库做blastp比较，然后注释。

注释速度：7万条蛋白3min注释完，神速！

### 2.5.2. 注释结果
两个文件，点击即可下载。
1. Mercator4 annotation results
一共有五列，每列信息都在单引号内：
- BINCODE：有层级的有序数字列表
- NAME：这一层级的功能名字，代表蛋白数据库里的功能层级
- IDENTIFIER：基因ID，在末级层级才会有基因ID
- DESCRIPTION：功能描述
- TYPE：类型

2. Mercator4 annotated fasta file

### 2.5.3. 结果整理
1. 从Mercator4 annotation results提取注释结果出来并转化成单基因单行信息格式，其中geneIDprefix根据基因名的前缀替换：

`cat mercator4v3/*.annotations.results.txt |awk -F"\t" '$3 ~ "geneIDprefix" {print $3"\t"$2": "$4}' |sed "s/'//g" |sort -k 1.3n |uniq |grep -v "no hits"|awk -F"\t" '{a[$1]=a[$1]$2"; "}END{for( i in a){print i"\t"a[i]}}'|sed "s/; $//g" |sort -k 1.3n |sed "s/not assigned.annotated/-/g" |sed "1i\gene\tmercator">mercator.anno`

注意检查是否还有no hits或者not assigned.annotated，有的话都替换掉。

2. 从Mercator4 annotated fasta file中以"not annotated"作为关键词搜索，可以获取未被注释到的基因。

## 2.6. GFAP - functional annotation software for plants
GFAP是windows下的一款基于python开发的软件，用于为植物基因组做功能注释。
2022年初发表在bioRxiv上，特地是运行非常快。
- [GFAP paper](https://www.biorxiv.org/content/10.1101/2022.01.05.475154v1)
- [GFAP software and manual](https://gitee.com/simon198912167815/gfap-database)

# 3. 基因功能注释的整合
提交功能注释文件有两种方案：
1. 整合功能注释和结构注释：把功能注释整理后添加到结构注释文件sample.gff第九列的product属性，与结构注释一同提交给NCBI等公共数据库
2. 单独提供功能注释：通过Dryad【收费】或Figshare【免费】等文件分享网站单独提供功能注释文件

## 3.1. 方案一：整合功能注释和结构注释
功能注释以product属性值的形式提供，添加到结构注释gff文件的第九列。

### 3.1.1. 基因功能注释整理
不同软件的基因功能注释结果都预先进行了整理（参考各个软件后的注释整理部分），整理成两列的格式，首行为表头，第一列为基因ID（gene），第二列为基因功能注释的描述信息（标明注释来源），每个注释软件/数据库的描述信息单独为一列。

目前共有9个文件被整理出来：
1. eggnog.anno
2. eggnog.go.anno
3. eggnog.kegg.anno
4. iprs.interpro.anno
5. iprs.panther.anno
6. iprs.pfam.anno
7. mercator.anno
8. pannzer2.go.anno
9. pannzer2.uniprot.anno

### 3.1.2. 合并功能注释和结构注释
合并步骤：
1. 整理注释结果
先把所有功能注释转换成单基因单注释的格式(单基因多行注释)，确保数据只有两列(基因ID列和功能注释列)，并删除除了product值以外的内容，比如iprs.panther.anno文件中的pantherid号，整理成pannzer2.uniprot.anno文件的形式，两列数据，第一列基因ID，第二列注释到的产物product值。

2. 合并多个注释结果
cat *anno|sort -f|uniq -i|awk '$2 != "" {print $0}' >merge.anno
合并多个注释结果，排序sort，去重uniq，和去除空值行等处理；按照基因ID排序，生成单基因多行注释的merge.anno文件

3. 过滤注释
最好的办法是在这一步做过滤filter。

`sed -i -E -e "s/[,|()[]{}]//g" -e "s/^[-:/]//g" -e "s/[-:/]$//g" -e "s/[^a-zA-Z]+//g" merge.anno`

product值的不规范情况：
- 包含逗号,（因为逗号是分隔符）
- 包含竖杠|
- 短横杠-开头或结尾
- 冒号:开头或结尾
- 斜杠/开头或结尾
- 不包含字母
- 括号（包括圆括号,中括号和大括号）不完整的

4. 单注释转多注释
用`cat merge.anno |awk -F"\t" '{a[$1]=a[$1]$2","}END{for( i in a){print i"\t"a[i]}}' |sed "s/,$/;/g"|sort -gk 1.3n|awk '{print $1"\tproduct="$2}' >product.list`命令从merge.anno生成product.list，每个基因单行的格式，还是两列，第一列基因ID，第二列若有多个注释用逗号隔开，用分号结尾。

5. 添加product属性值
sample.gff与product.list合并，把结构注释和功能注释文件合并，功能注释以product属性值的形式加入gff文件的第九列内容。

## 3.2. 方案二：单独提供功能注释
### 3.2.1. 基因功能注释整理
不同软件的基因功能注释结果都预先进行了整理（参考各个软件后的注释整理部分），整理成两列的格式，首行为表头，第一列为基因ID（gene），第二列为基因功能注释的描述信息（标明注释来源），每个注释软件/数据库的描述信息单独为一列。

目前共有9个文件被整理出来：
1. eggnog.anno
2. eggnog.go.anno
3. eggnog.kegg.anno
4. iprs.interpro.anno
5. iprs.panther.anno
6. iprs.pfam.anno
7. mercator.anno
8. pannzer2.go.anno
9. pannzer2.uniprot.anno

### 3.2.2. 基因功能注释整合

需要整合所有注释软件/数据库的信息，只需根据第一列进行文件的合并。

#### 3.2.2.1. join命令

1. 实现合并a.anno和b.anno的方法：a.anno和b.anno都只有两列的情况
```
join -t $'\t' -j 1 a.anno b.anno >ab.tem #合并两个注释文件都注释到基因的行
join -t $'\t' -j 1 -v 1 a.anno b.anno |awk -F"\t" '{print $1"\t"$2"\t-"}' >a.tem #提取a.anno单独注释到基因的行
join -t $'\t' -j 1 -v 2 a.anno b.anno |awk -F"\t" '{print $1"\t-\t"$2}' >b.tem #提取b.anno单独注释到基因的行
cat ab.tem a.tem b.tem |sort -k 1.3n >ab.anno #合并注释文件
rm ab.tem a.tem b.tem # 删除临时文件
```

2. 实现合并ab.anno和c.anno的方法：ab.anno有三列，c.anno有两列的情况
```
join -t $'\t' -j 1 ab.anno c.anno >abc.tem #合并两个注释文件都注释到基因的行
join -t $'\t' -j 1 -v 1 ab.anno c.anno |awk -F"\t" '{print $1"\t"$2"\t"$3"\t-"}' >ab.tem #提取ab.anno单独注释到基因的行
join -t $'\t' -j 1 -v 2 ab.anno c.anno |awk -F"\t" '{print $1"\t-\t-\t"$2}' >c.tem #提取c.anno单独注释到基因的行
cat abc.tem ab.tem c.tem |sort -k 1.3n >abc.anno #合并注释文件
rm abc.tem ab.tem c.tem # 删除临时文件
```

如果多个文件，则两两依次合并。还没想到什么更简单的方法，就这样用join手动合并所有注释软件/数据库的功能注释到一个文件吧。

#### 3.2.2.2. merge_file_key_property.title.cpp程序
请大佬写了个合并功能注释的程序，merge_file_key_property.title.cpp，可以一次合并任意多个文件。

1. 编译
`g++ -std=c++11 ./merge_file_key_property.cpp -o merge_file_key_property.o`生成执行文件*.o
2. 执行
` ./merge_file_key_property.o -h`查看参数

`./merge_file_key_property.o -a .anno -n 7 -p mc -q gene -o merged` 使用示范：-a指定合并的文件类型（比如.anno）；-n指定第一列字段长度（比如sp00001长度为7）；-p指定第一列关键词的前缀（基因名前缀，比如sp）；-q指定第一列关键词的标题（比如gene），作为表头；-o指定合并文件名的前缀。

#### 3.2.2.3. waiting...
如果文件多且特征复杂，还是琢磨一下python或者R的merge功能吧。


# 4. references
1. https://www.jianshu.com/p/67dbafa86334
2. https://www.jianshu.com/p/e646c0fa6443
3. 文章：http://xuzhougeng.top/archives/Function-anotation-with-swiss-prot-database
4. https://www.jianshu.com/p/4f4819f385d2

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>
