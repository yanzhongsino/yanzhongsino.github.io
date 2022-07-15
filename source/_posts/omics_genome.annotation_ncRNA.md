---
title: 基因组注释（四）：非编码RNA的注释-用Infernal软件对Rfam 12进行RNA注释
date: 2022-04-22
categories: 
- omics
- genome
- annotation

tags:
- genome
- genome annotation
- ncRNA
- Infernal
- Rfam

description: 使用Infernal软件对Rfam数据库比对实现基因组非编码RNA的注释
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283100&auto=1&height=32"></iframe></div>

# 1. ncRNA
非编码RNA(Non-coding RNA, ncRNA) 包括rRNA，tRNA，snRNA，snoRNA 和microRNA 等不编码蛋白质的RNA，它们转录后直接在RNA 水平上就能行使各自的生物学功能，并不需要翻译成蛋白质。

# 2. 注释软件
- 非编码RNA种类繁多，且结构特征各不相同，所以开发出了许多注释特定某一类RNA的软件，比如tRNAScan-SE预测tRNA，rnammer预测rRNA，snoScan 搜索带C/D盒的snoRNAs，SnoGps 搜索带H/ACA盒的snoRNAs，mirScan搜索microRNA等。
- Sanger实验室开发了Infernal软件，建立了1600多个RNA家族，并对每个家族建立了一致性二级结构和协方差模型，形成了Rfam数据库。采用Rfam数据库中的每个RNA的协方差模型，结合Infernal软件可以预测出已有RNA家族的新成员，只是特异性比较差。

如果不是专门研究ncRNA，可以用Infernal注释所有ncRNA。如果需要更精细的注释，则可以选择特定软件注释特定RNA。

这篇博客是介绍用Infernal程序与Rfam数据库一起用来注释与数据库中已知ncRNA同源的序列（这里用来注释完整的基因组）。注释结果包括tRNA，rRNA，snRNA，snoRNA和miRNA等。

# 3. Infernal
[Infernal](http://eddylab.org/infernal/)全称是"INFERence of RNA ALignment"，是一个用来检索DNA序列数据库中RNA序列和结构相似性的软件，通过协方差模型covariance models (CMs)来实现。

## 3.1. 安装Infernal
`conda install -c bioconda infernal`
现在安装的是v1.1.4

安装后可使用的命令包括：
- cmpress：对cm文件进行压缩并建立索引。
- cmscan：用提交的序列在cm数据库中进行检索。
- cmalign：将RNA序列同协方差模型进行比对，并输出为stockholm格式。
- cmbuild：通过多序列比对结果建立一个协方差模型，并保存在新文件中。
- cmcalibrate：对协方差模型(CM)进行校准，在使用cmsearch和cmscan前CM模型需要先经过这个程序的处理。
- cmconvert：用于格式转换，是将infernal 1.0以后的CM转化为当前版本需要的CM。但对1.0之前的版本无效。
- cmemit：一个采样程序，从CM中进行采样并输出。
- cmfetch：从一个大的CM文件中获取一个或多个CM。
- cmsearch：用提交的CM模型在序列数据库中进行检索。
- cmstat：对CM文件中的CM模型进行统计汇总。

# 4. Rfam
[Rfam](http://rfam.xfam.org/)是RNA family数据库，包括ncRNA序列和ncRNA的二级结构，每个family用多序列比对和协方差模型covariance model (CM)来表示。

1. 下载Rfam数据库 
- `wget http://ftp.ebi.ac.uk/pub/databases/Rfam/CURRENT/Rfam.cm.gz`
在[Rfam](http://rfam.xfam.org/)网站下载Rfam最新版本的CM数据库（目前是Rfam 14.7）
- `gunzip Rfam.cm.gz`
解压数据库

2. 下载clanin
- `wget http://ftp.ebi.ac.uk/pub/databases/Rfam/CURRENT/Rfam.clanin`
在[Rfam](http://rfam.xfam.org/)网站下载Rfam数据库配套的clanin文件。

# 5. 注释ncRNA
## 5.1. 建库
- `cmpress Rfam.cm`
使用cmpress压缩并建立索引，生成Rfam.cm.i1f, Rfam.cm.i1i, Rfam.cm.i1m, Rfam.cm.i1p。

## 5.2. 注释
1. 序列索引
推荐的参数：

`nohup cmscan -Z 512 --cut_ga --rfam --nohmmonly --fmt 2 --tblout sample.tblout -o sample.result --clanin Rfam.clanin Rfam.cm genome.fa &`

- -Z：根据基因组大小来定，基因组大小的2倍，Mb单位，选一个整数。比如256Mb的基因组，-Z 512。
- `--cut_ga --rfam --nohmmonly --fmt 2`：推荐使用
- --tblout sample.tblout：指定table格式输出文件
- -o sample.result：指定比对结果输出文件
- --clanin Rfam.clanin：指定clanin文件
- Rfam.cm genome.fa：指定数据库Rfam.cm和基因组genome.fa

note：-o sample.result要放在Rfam.cm genome.fa前面，否则报错。

此步骤**耗时**参考：250Mb基因组，默认线程，耗时2.5h。

2. 结果文件
- sample.result：比对结果
- sample.tblout：table格式结果

## 5.3. 整理结果
### 5.3.1. 将注释结果整理成gff3文件
gff3文件可用于提交注释到数据库。

用perl脚本[infernal-tblout2gff.pl](https://github.com/yanzhongsino/bioscripts/blob/main/saved_scripts/infernal-tblout2gff.pl)实现，脚本来自https://www.cnblogs.com/jessepeng/p/15392809.html。

`perl infernal-tblout2gff.pl --cmscan --fmt2 sample.tblout >sample.infernal.ncRNA.gff3`

### 5.3.2. 统计各类ncRNA总数
1. 整理注释结果文件sample.tblout
提取必需的列，非重叠区域或重叠区域得分高的区域
`awk 'BEGIN{OFS="\t";}{if(FNR==1) print "target_name\taccession\tquery_name\tquery_start\tquery_end\tstrand\tscore\tEvalue"; if(FNR>2 && $20!="=" && $0!~/^#/) print $2,$3,$4,$10,$11,$12,$17,$18; }' sample.tblout >sample.tblout.xls`

2. 下载rfam注释
- 在[rfam官网](https://rfam.xfam.org/)，选择【SEARCH】-【Entry type】
- 然后选中所有的Entry types（包括Gene，Intron，Cis-regulatory element），点击【Submit】，会列出所有RNA family的注释信息。
- 手动选择所有注释信息，复制，粘贴到新建的空白文本文件rfam.txt并保存。
- 把rfam.txt传输到服务器，最好用`dos2unix rfam.txt`转换文件格式为unix版本。
- 拆分第三列`cat rfam.txt | awk 'BEGIN {FS=OFS="\t"}{split($3,x,";");class=x[2];print $1,$2,$3,$4,class}' > rfam_anno.txt`

rfam注释文件rfam_anno.txt包含了所有rfam的类型type和功能描述description信息。

3. 统计ncRNA注释结果
`awk 'BEGIN{OFS=FS="\t"}ARGIND==1{a[$2]=$5;}ARGIND==2{type=a[$1]; if(type=="") type="Others"; count[type]+=1;}END{for(type in count) print type, count[type];}' rfam_anno.txt sample.tblout.xls >sample.ncRNA.statistic`

sample.ncRNA.statistic输出示例：

```
 riboswitch	1
 ribozyme	1
 tRNA	699
Others	41
 miRNA	188
 antisense	4
 rRNA	233
 snRNA	698
```

4. 统计细分分类
- 也可以根据细分分类分别统计，细分分类参考[rfam官网](https://rfam.xfam.org/)，【SEARCH】-【Entry type】。
- 可参考的统计值包括每个细分ncRNA的数量(copy)，平均长度(average length)，总长(total length)，总长占基因组的比例（Percent of the genome）等
- 统计后整理成发表文章用的表格。

比如：snRNA包括snoRNA和splicing，snoRNA包括CD-box，HACA-box和scaRNA。下面用统计CD-box这个细分分类的ncRNA举例。

- 提取CD-box的Accession(RF00000格式)：`grep "CD-box" rfam_anno.txt |cut -f1 >cdbox.tem`
- 提取注释到的CD-box信息：`grep -f cdbox.tem mc.tblout.xls >cdbox.txt`
- cdbox.txt的行数就是CD-box的数量；利用第四五列的位置信息，即可统计平均长度(average length)，总长(total length)，**注意正反链**。

# 6. references
1. https://www.cnblogs.com/jessepeng/p/15392809.html
2. http://www.360doc.com/content/18/1119/05/52645714_795799901.shtml
3. https://genehub.wordpress.com/2019/08/08/%E6%A4%8D%E7%89%A9%E5%9F%BA%E5%9B%A0%E7%BB%84ncrna%E9%A2%84%E6%B5%8B%EF%BC%88trna%E3%80%81rrna%E3%80%81snrna%E3%80%81mirna%EF%BC%89/
4. http://embracethesky.cn/2018/07/08/%e4%bd%bf%e7%94%a8infernal%e5%af%b9rfam-12%e8%bf%9b%e8%a1%8crna%e6%b3%a8%e9%87%8a/#more-99

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>