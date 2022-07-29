---
title: 基因组WGD事件的鉴定和时间估算 —— MCScanX,KaKa_Calculator
date: 2021-11-27 17:50:00
categories: 
- bio
- bioinfo
tags:
- synteny
- colinearity
- Ks
- WGD
- biosoft
- MCScanX
- ParaAT.pl
- KaKs_Calculator
- ggplot2
- paml
- divergence time

description: 用MCScanX对基因组进行同线性分析，KaKs_Calculator2.0计算同线性区块的Ks，根据Ks分布和峰值预测全基因组复制事件(Whole genome duplication,WGD)，paml的mcmctree模块估算WGD的发生时间。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=29808885&auto=1&height=32"></iframe></div>

# 1. MCScanX介绍
MCScanX调整[MCScan算法](https://yanzhongsino.github.io/2021/11/05/bioinfo_MCscan/)进行检测基因组间或基因组内的同线性，并附加了14个下游分析和可视化的脚本。

# 2. MCScanX同线性分析
## 2.1. 准备输入文件
MCScanX做同线性分析需要两个输入文件sample.gff(四列数据)和sample.blast。

如果是多个物种，则把多个物种的gff3文件和pep.fa文件合并后再准备sample.gff和sample.blast输入文件。

1. sample.gff
`cat sample.gene.gff3 |awk '{if($3=="gene"){print $1,$9,$4,$5}}'|sed "s/;.*;//g"|sed "s/ID=//g"|sed "s/ /\t/g" >sample.gff #准备gff文件`
2. sample.blast
```shell
makeblastdb -in sample.pep.fa -dbtype prot -out index/sample.pep #给蛋白序列建库
blastp -query sample.pep.fa -db index/sample.pep -out sample.blast -evalue 1e-5 -num_threads 12 -outfmt 6 -num_alignments 5 & #进行自我比对，生成6号格式的比对结果sample.blast
```

## 2.2. 运行MCScanX
在有sample.gff和sample.blast两个文件的目录下，指定前缀sample运行`MCScanX sample`。

重要参数解释：
- -s MATCH_SIZE,default: 5。每个共线性区块包含的基因数量的下限。
- -m MAX_GAPS，default：25。在共线性区块中允许的最大gaps数量。
- -b patterns of collinear blocks。0:intra- and inter-species (default); 1:intra-species; 2:inter-species。

## 2.3. 结果文件
1. sample.collinearity

共线性结果文件，包括三部分内容：

- 参数(parameters)
- 基本统计信息(statistics)：共线性基因的总数，总基因数，共线性基因占比。
- 共线性区块(block)信息：一个Alignment代表一个共线性区块（0起始编号）。后面跟着这个共线性区块的基因对的信息。第一列：block编号；第二列：基因对编号；第三列和第四列：基因对名称；第五列：blast比对的e_value值。

sample.collinearity示例：

```
############### Parameters ###############
# MATCH_SCORE: 50
# MATCH_SIZE: 5
# GAP_PENALTY: -1
# OVERLAP_WINDOW: 5
# E_VALUE: 1e-05
# MAX GAPS: 25
############### Statistics ###############
# Number of collinear genes: 21371, Percentage: 56.25
# Number of all genes: 37996
##########################################
## Alignment 0: score=352.0 e_value=1.2e-17 N=8 scaf014&scaf014 plus
477-  0:	bv37796	bv37889	  3e-06
477-  1:	bv37820	bv37905	  6e-24
477-  2:	bv37822	bv37926	  9e-29
477-  3:	bv37826	bv37930	  4e-62
477-  4:	bv37828	bv37931	  1e-74
477-  5:	bv37829	bv37932	 3e-142
477-  6:	bv37830	bv37933	  6e-78
477-  7:	bv37832	bv37934	  3e-76
## Alignment 1: score=300.0 e_value=6.1e-08 N=6 scaf014&scaf014 minus
478-  0:	bv35514	bv35525	      0
478-  1:	bv35515	bv35524	      0
478-  2:	bv35516	bv35523	  1e-81
478-  3:	bv35517	bv35522	      0
478-  4:	bv35518	bv35521	 1e-115
478-  5:	bv35519	bv35520	      0
... ...
... ...
```

2. sample.html

网页文件所在的文件夹，里面有每条染色体一个html文件。html文件用浏览器打开，包含三列信息。
- 第一列是复制深度。
- 第二列是这条染色体上所有基因的排列顺序，串联重复基因的背景为红色。
- 第三列和之后列是对应的比对上的基因名称。

3. sample.tandem

包含基因组内**串联重复**的基因ID的list。

notes：MCScanX 会根据 gff 文件中染色体号的前缀（前2个字符）将染色体划分为不同的物种，若 MCScanX 识别到输入数据中包含多个物种，则不会生成 tandem 文件。

# 3. MCScanX下游分析
## 3.1. 提取block位置

```
cat sample.collinearity|grep -C 1 "Alignment"|sed '/^--$/d' >block.tem #提取block首尾基因对
tail -1 sample.collinearity >>block.tem #提取block首尾基因对
cat block.tem |sed -e "s/## //g" -e "s/.*gene/gene/g"|tr '\n' ' '|sed "s/Alignment /\nAlignment/g" |sed -e "/^$/d" -e "s/ /\t/g"|sed -e "s/\t\+/\t/g" -e "s/\://g"|awk -v FS="\t" -v OFS="\t" '{print $1,$2,$3,$4,$5,$6,$7,$10,$8,$11}' >block.txt #整理成一个block一行信息的格式，其中"s/.*gene/gene/g"部分的gene修改成geneID的前缀用来去除多余信息。
```

## 3.2. 绘图脚本
MCScanX的downstream_analyses目录包含一些下游脚本。

许多java脚本可以实现绘图功能，包括：
- 共线性点图：dot_plotter
- 双染色体共线性图：dual_synteny_plotter
- 共线性圈图：circle_plotter
- 染色体块图：bar_plotter
- ...

## 3.3. 复制事件分类
### 3.3.1. duplicate_gene_classifier
1. duplicate_gene_classifier

MCScanX的`duplicate_gene_classifier`脚本用来分析与各种复制类别相关的基因数量

2. 输入

`duplicate_gene_classifier sample` # 输入与MCScanX一致，读取当前目录下的sample.gff和sample.blast作为输入

3. 输出示例

```
Type of dup	Code	Number
Singleton	0	88907
Dispersed	1	13573
Proximal	2	463
Tandem	3	847
WGD or segmental	4	1055
```

4. 结果解释

其中 Singleton 表示单拷贝重复，Proximal 表示在相同染色体上相近但不相连的重复，Dispersed 表示除 Tandem、WGD、Proximal 以外的重复。Number代表相应复制类别相关的基因数量。

## 3.4. 解析结果文件sample.collinearity的脚本

推荐这里https://github.com/reubwn/collinearity有许多可以解析果文件sample.collinearity的脚本

## 3.5. 计算共线性区块Ks
tips:需要检查得到的结果中Ks值是否有负值或NA等无效数据，并过滤无效数据。
### 3.5.1. 方案一：add_ka_and_ks_to_collinearity.pl（MCScanX的downstream_analyses目录下自带脚本）
`add_ka_and_ks_to_collinearity.pl -i sample.collinearity -d sample.cds.fa -o sample.kaks > out.log 2>&1` #注意算出的部分kaks值为-2的问题，未找到解决方案

### 3.5.2. 方案二：ParaAT.pl+KaKs_Calculator2.0
- ParaAT.pl用于根据同源基因对list生成比对的gene对cds序列，并可以指定输出格式，如axt格式；
- KaKs_Calculator用于计算基因对的kaks。

用还会用到两个脚本:
- [axt2one-line.py](https://github.com/scbgfengchao/4DTv/blob/master/axt2one-line.py)转换axt格式为单行
- [calculate_4DTV_correction.pl](https://github.com/JinfengChen/Scripts/blob/master/FFgenome/03.evolution/distance_kaks_4dtv/bin/calculate_4DTV_correction.pl)计算4dtv。

1. 安装ParaAT.pl
参考[blog.ParaAT](https://yanzhongsino.github.io/2021/10/29/bioinfo_align.pep2cds/)

```shell
wget ftp://download.big.ac.cn/bigd/tools/ParaAT2.0.tar.gz
tar -zxf ParaAT2.0.tar.gz
cd ParaAT2.0
ParaAT.pl -h
```

2. 安装KaKs_Calculator2.0
[KaKs_Calculator2.0下载地址](https://sourceforge.net/projects/kakscalculator2/?source=typ_redirect)

```shell
wget https://altushost-swe.dl.sourceforge.net/project/kakscalculator2/KaKs_Calculator2.0.tar.gz
tar -zxf KaKs_Calculator2.0.tar.gz
chmod 777 ./KaKs_Calculator2.0/bin/Linux/KaKs_Calculator
chmod 777 ./KaKs_Calculator2.0/src/AXTConvertor
```

然后把./KaKs_Calculator2.0/bin/Linux/和./KaKs_Calculator2.0/src/添加到环境变量即可使用KaKs_Calculator和AXTConvertor命令。

3. 用ParaAT.pl获取共线性基因对比对序列
```shell
cat sample.collinearity |grep -v "^#"|cut -f2,3 >sample.homolog #提取blocks的同源gene对
echo "24" >proc #指定ParaAT.pl使用线程
ParaAT.pl -g -t -h sample.homolog -n sample.cds.fa -a sample.pep.fa -m mafft -p proc -f axt -k -o sample.paraat 2> paraat.log & #用ParaAT.pl调用mafft做每对共线性基因的蛋白比对和蛋白转cds比对，输出axt格式。-g移除比对有gap的密码子，-t移除mismatched codons；；-o指定生成目录；加上-k参数可以在获得axt文件后自动调用KaKs_Calculator计算kaks值，使用MA模型，比YN模型慢，推荐手动用KaKs_Calculator的YN模型，生成sample.axt.kaks文件。
```

建议加上-g和-t，免得后面计算Ks时报错Error. The size of two sequences in 'ctg00816-ctg08844' is not equal。

4. 用KaKs_Calculator手动计算共线性基因对的KaKs和4dtv值
ParaAT.pl的-k参数只能指定KaKs_Calculator的MA模型计算kaks值，如果需要指定其他的模型，则可以手动运行计算。

```shell
cd sample.paraat
for i in `ls |grep "axt"`;do KaKs_Calculator -i $i -o ${i}.kaks -m YN;done #用YN模型计算每个gene对的KaKs，生成四列数据，gene对，Ka，Ks，Ka/Ks
for i in `ls |grep "kaks"`;do awk 'NR>1{print $1"\t"$3"\t"$4"\t"$5}' $i >>../all.kaks;done #合并kaks到all.kaks文件
- 计算4dtv
for i in `ls |grep "axt"`;do axt2one-line.py $i ${i}.one-line;done #多行axt文件转换成单行
ls |grep "axt.one-line"|while read id;do calculate_4DTV_correction.pl $id > ${id%%one-line}4dtv;done #计算4dtv值，生成两列数据，gene对，4dtv
for i in `ls |grep "4dtv"`;do awk 'NR>1{print $1"\t"$3}' $i >>../all.4dtv;done #合并4dtv值到all.4dtv
cd ..
join all.kaks all.4dtv |sed "s/ /\t/g" |awk '$3 != "NA" {print $0}' |sed '1i\genepair\tKa\tKs\tKa/Ks\t4dtv_corrected' >all.results #以gene对为基准，join合并kaks和4dtv值到一个文件，然后过滤Ks值为NA的无效数据，添加标题行。

rm all.kaks* all.4dtv* #删除中间文件
```

## 3.6. ggplot2画密度分布图和峰值
### 3.6.1. ggplot2画密度分布图
```R
library(ggplot2)
library(ggpmisc)
data <- read.table("all.results",header=T)
p <- ggplot(data, aes(Ks)) + geom_density(size=1,color="black")+xlab("Synonymous substitution rate(Ks)")+ylab("Percent of Total Paralogs")+theme_classic()+scale_y_continuous(breaks=seq(0, 2.5, 0.2))+xlim(0,3)
ggsave(file="Ks.pdf",plot=p,width=10,height=5)
```

### 3.6.2. ggplot2峰值标定
```R
pb <- ggplot_build(p)
pic <- p + stat_peaks(data = pb[['data']][[1]], aes(x=x, y=density), geom= 'text', color="red", ,vjust=-0.5)
ggsave(file="Ks_peaks.pdf",plot=pic,width=10,height=5)
```

在密度分布图里得到红色标记的峰值。

### 3.6.3. 多个密度图
同时展示多组Ks数据分布，可以把数据合并，添加一列作为分类标签，通过colour颜色参数指定这个分类标签列，则可以实现一张图上展示多个密度图。

`ggplot(data)+geom_density(aes(x=V3,colour=V5),adjust=2)+xlim(0,0.5)+theme_classic()` #指定第三列V3为数据，第五列V5为分类依据并赋予不同颜色。

### 3.6.4. 通过密度分布图判断WGD
一般认为，Ks密度分布图如果有明显的峰，一个峰代表对应一次WGD事件。
通过峰的数量和对应的Ks大小可以判断WGD事件的次数和发生时间。

## 3.7. WGD发生的时间
通过Ks的密度分布图鉴定得到WGD事件发生的证据之后，有多种方法可以估算WGD发生的时间。
### 3.7.1. 通过Ks=2μT计算时间
依据公式Ks=2μT来计算时间T，依赖准确的进化速率μ，μ通常引用近缘类群的已有权威研究。

进化速率μ：
- 蕨类植物的同义突变率：4.79e-9 subst./syn. site/year :the first estimate of fern nuclear genome evolutionary rates with polypodiaceous nuclear genomes (Barker 2009)
- 水稻：2.5e-8 subst./syn. site/year

### 3.7.2. 通过进化树估算时间
- 用代表WGD的Ks值附近的同线性基因对制作两套样品的单倍型数据，然后选取一到多个近缘种的蛋白序列与其中一套单倍型做直系同源基因分析，用找到的直系同源单拷贝基因建树（把两套单倍型数据当作两个物种来建树），通过化石或者二次标定的方法用paml的mcmctree模块计算两套单倍型的分化时间，即为WGD发生的时间。
- 如果物种分化时间的尺度太大而不能准确估算WGD时间，可以先用大尺度的系统发育树做近缘种和目标物种的分化时间的估算，然后在做WGD时间估算时只用一个近缘种加上近缘种与目标物种的分化时间标定来获得更准确的WGD时间估计结果。

#### 3.7.2.1. 获取sample发生WGD的两套基因
1. 前面分析中计算Ks的分布峰值假设在**0.05**，取Ks值在0.05正负25%的范围，即[0.0375-0.0625]之间的基因对。
`cat all.results |awk '$3>0.0375 && $3>0.0625 {print $1} |sed "s/-/\t/g" >wgd.homologs`
2. 基因对中若有一个基因对多个基因的情况，则选取Ks最接近0.05的那对。
3. 获取两套基因list。
`cut -f1 wgd.homologs >wgd_H1.homologs`;
`cut -f2 wgd.homologs >wgd_H2.homologs`

#### 3.7.2.2. 选近缘种和分化时间
1. 参考近年发表的权威组学系统发育文章选取近缘种和物种分化时间数据。
2. 这里假设选取了两个近缘物种sampleA和sampleB。sample发生WGD形成的两套单倍型H1和H2，目标物种sample与近缘种sampleA的分化时间已知为88Ma。
3. 确定物种树拓扑结构：`(((sampleH1,sampleH2),sampleA)'<0.88>0.88),sampleB;`，以sampleB为外类群。

#### 3.7.2.3. orthofinder找直系同源基因
1. 根据基因对提取蛋白序列，任意用一个（这里用wgd_H1.homologs)提取蛋白序列：`seqkit grep -n -f wgd_H1.homologs sample.pep > wgd_H1.pep`
2. 与其他两个近缘种的蛋白序列一起，用orthofinder找直系同源基因。
- orthofinder使用方法：把所有物种的pep蛋白序列（两个近缘物种加上wgd_H1.pep）放进一个文件夹(directory)，注意蛋白序列中不能有./*等符号（常用来代表终止密码子）。
- 运行`orthofinder -f directory -t 24` # -t指定线程
- 结果文件的Orthogroups目录下，Orthogroups.txt包含所有直系同源基因，Orthogroups_SingleCopyOrthologues.txt包含单拷贝的直系同源基因。
- 根据Orthogroups_SingleCopyOrthologues.txt把Orthogroups.txt里基因信息提取出来,`grep -f Orthogroups_SingleCopyOrthologues.txt Orthogroups.tsv >OG.txt`。OG.txt文件内容是singlecopyorthologues列表，第一列orthogroupID，后面3列每套蛋白序列ID，共4列，这个文件的每行数据为一个homologs对应的每套数据的蛋白ID。
- 合并wgd.homologs和OG.txt文件为OG.list：`join -1 1 -2 2 wgd.homologs OG.txt > OG.list`。OG.list文件是在OG.txt基础上增加了wgd_H2.homologs的一套蛋白ID数据，共有四套蛋白ID数据。

#### 3.7.2.4. 建单基因树
1. 建树脚本
用下面的脚本singlegenetree.sh，提取每个homologs序列，并通过raxml-ng（raxmlHPC建氨基酸树只有bootstrap结果，没有bestTree）为每个homologs基因比对（mafft）和建树。
OG.list文件包含第一列ogID和后4列4套样本ID。

```shell
# singlegenetree.sh
cat ./OG.list | while read line
do
        sample_id=$(echo $line |awk '{print $1}')
        sample_a=$(echo $line |awk '{print $2}')
        sample_b=$(echo $line |awk '{print $3}')
        sample_c=$(echo $line |awk '{print $4}')        
        sample_d=$(echo $line |awk '{print $5}')
        mkdir ./singlegenetree/${sample_id}
        seqkit grep -n ${sample_a} sample.pep >./singlegenetree/${sample_id}/H1_${sample_a}.pep
        seqkit grep -n ${sample_b} sample.pep >./singlegenetree/${sample_id}/H2_${sample_b}.pep
        seqkit grep -n ${sample_c} sampleA.pep >./singlegenetree/${sample_id}/${sample_c}.pep
        seqkit grep -n ${sample_d} sampleB.pep >./singlegenetree/${sample_id}/${sample_d}.pep
        cd ./singlegenetree/${sample_id}/
        cat H1_${sample_a}.pep H2_${sample_b}.pep ${sample_c}.pep ${sample_d}.pep > ${sample_id}.pep
        mafft ${sample_id}.pep >${sample_id}.mafft.pep
        raxml-ng --all --msa ${sample_id}.mafft.pep --data-type AA --threads 8 --bs-trees 100 --model LG+G8+F --tree pars{10} --outgroup ${sample_d} --redo
        sed -E "s/:[0-9]\.[0-9]*//g" ${sample_id}.mafft.pep.raxml.bestTree  >besttree.${sample_id} #删除不必要的枝长等信息
        cd ../../
done
```

2. 筛选homologs
判断生成的每个OG的树文件besttree.${sample_id}，只保留符合物种树拓扑结构`(((sampleH1,sampleH2),sampleA)),sampleB;`的基因，保存成OG.filtered。

#### 3.7.2.5. paml估计分化时间
此节参考博客[估算系统树分歧时间](https://yanzhongsino.github.io/2021/03/25/bioinfo_caculate.divergence.time/)的paml部分。

1. 在OG.filtered最后一列添加每个OG的序列长度信息，保存成OG.filtered.list。
2. 准备输入文件sample.phy
上一步脚本提取的每个homologs的比对后的四套数据蛋白序列${sample_id}.mafft.pep，用脚本fa2phylip.sh换成phylip格式（用python的SeqIO库换的格式mcmctree识别不了）。
并合并提取的所有homologs序列。

```shell
# fa2phylip.sh
cat ./OG.filtered.list | while read line
do
        sample_id=$(echo $line |awk '{print $1}') #获取OG.ID
        sample_a=$(echo $line |awk '{print $6}') #获取序列长度
        sed "s/_.*//g" ../singlegenetree/${sample_id}/${sample_id}.mafft.pep|seqkit seq -w 0|sed -E ":a;N;s/\n/ /g;ta" |sed "s/ >/\n/g" |sed "s/>//g"|sed "s/ /  /g"|sed "1i\5  ${sample_a}" |sed '1i\ ' > ./phy/${sample_id}.phy #把上一步获取的${sample_id}.mafft.pep改为phylip格式，并在首行添加空格行（为了合并后每个OG用空行隔开）

done
cat ./phy/*phy >sample.phy #合并所有phy文件为一个sample.phy文件，假设共有261个子phy文件被合并
```

3. 准备输入文件sample.tre
- sample.tre包含两行，第一行表述树中有5个样本，共计1个树，两个数值之间用空格分割；
第二行则是Newick格式树信息。
- 其中第二行Newick格式树包含有校准点信息，校准点信息一般指95%HPD（Highest Posterior Density）对应的置信区间，这里用的校准点时间只有一个点，所以写成'<0.88>0.88'形式；校准点单位是100MYA（软件说明文档中使用该单位，也推荐使用该单位，若使用其它单位，后续配置文件中的相关参数也需要对应修改）。
- 此外，Newick格式的树尾部一定要有分号，没有的话程序可能不能正常运行。
- sample.tre内容为：

```
4 1
(((sampleH1,sampleH2),sampleA)'<0.88>0.88),sampleB;
```

4. 准备配置文件mcmctree3.ctl
ndata = 261表示有261个数据，与sample.phy包含的子phy数量对应；seqtype = 2表示这里用的氨基酸序列；model = 0暂时选用0，后面要改；

``` #cat mcmctree3.ctl
          seed = -1
       seqfile = sample.phy
      treefile = sample.tre
       outfile = mcmc.out
      mcmcfile = mcmc.txt

       seqtype = 2    *0: DNA data; 1: codon data; 2: AAs data
        ndata = 261
       usedata = 3    * 0: no data; 1:seq like; 2:normal approximation
         clock = 3    * 1: global clock; 2: independent rates; 3: correlated rates
*      TipDate = 1 100
        RootAge = '<10'  * constraint on root age, used if no fossil for root.

         model = 0    * 0:JC69, 1:K80, 2:F81, 3:F84, 4:HKY85
         alpha = 0.5   * alpha for gamma rates at sites
         ncatG = 4    * No. categories in discrete gamma

     cleandata = 0    * remove sites with ambiguity data (1:yes, 0:no)?

       BDparas = 1 1 0.1   * birth, death, sampling
   kappa_gamma = 6 2      * gamma prior for kappa
   alpha_gamma = 1 1      * gamma prior for alpha

   rgene_gamma = 2 20 1   * gamma prior for rate for genes
  sigma2_gamma = 1 10 1    * gamma prior for sigma^2     (for clock=2 or 3)

      finetune = 1: .1 .1 .1 .1 .1 .1  * times, rates, mixing, paras, RateParas

         print = 1
        burnin = 1000000
      sampfreq = 10
       nsample = 500000

*** Note: Make your window wider (100 columns) when running this program.
```

5. 运行`mcmctree mcmctree3.ctl`
- 此步骤的目的是为了获取in.BV文件；
- 用命令`mcmctree mcmctree3.ctl`运行起来后，ctrl+C多次中断运行（共261个序列，会依次调用261次codeml，中断后会自动进入下一次调用，目的是完成261次调用生成261个序列的临时文件）；由于单个序列短，很快会完成分析，也可以不ctrl+C中断，等待完成，只要生成261套tmp*文件即可。
- 删除rst2文件和mcmc.out,mcmc.txt,out.BV文件；把paml软件下的wag.dat文件复制到当前目录下`cp ~/software/anaconda3/dat/wag.dat ./`；
- 用命令`sed -i -e "s/model = 0/model =2\naaRatefile = wag.dat/" -e "s/method = 0/method = 1/" tmp*.ctl`把所有tmp*.ctl文件的model修改成model = 2，method修改成method = 1,并添加一行aaRatefile = wag.dat；
- 用命令`for i in $(ls tmp*.ctl);do codeml ${i} ; mv rst2 ${i}.rst2; done`调用codeml进行261次分析（每次codeml都生成rst2文件，所以每次生成后把rst2改名，否则rst2被覆盖）；
- 然后`cat *rst2 >in.BV`合并所有rst2文件到in.BV文件；

6. 运行`mcmctree mcmctree2.ctl`
- 新建目录并复制四个文件sample.phy,sample.tre,mcmctree3.ctl,in.BV到新目录`mkdir mcmctree2 && cd mcmctree2 && cp ../Ane.* ./ && cp ../mcmctree3.ctl ./ && cp ../in.BV ./`;
- 把mcmctree3.ctl文件内的usedata = 3改为usedata = 2，并重命名成mcmctree2.ctl；
- 然后运行`mcmctree mcmctree2.ctl`；
- 等待两三天获取结果，FigTree.tre文件包含了所有节点的95%HPD的时间，是nexus格式。
- 根据FigTree.tre结果：`((sampleA: 0.408365, (sampleH1: 0.107333, sampleH2: 0.107333) [&95%={0.0749432, 0.126365}]: 0.301032) [&95%={0.285365, 0.478163}]: 0.362000, sampleB: 0.770366) [&95%={0.53864, 0.90012}];`的sampleH1和sampleH2分化的时间，可以看出WGD发生的时间在**10.7Ma[7.5-12.6]**之间。

# 4. references
1. https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3326336/
2. https://github.com/wyp1125/MCScanX
3. https://its201.com/article/sinat_41621566/113359074

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>