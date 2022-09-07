---
title: 批量计算Ka和Ks
date: 2022-09-07
categories:
- bio
- bioinfo
- Ks
tags:
- Ks
- Ka
- 4dtv
- ParaAT
- KaKs_Calculator

description: 记录了使用ParaAT和KaKs_Calculator2.0批量计算基因对/序列对的Ka、Ks和4dtv的方法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1901371647&auto=1&height=32"></iframe></div>

# 1. ParaAT
参考blog.ParaAT：https://yanzhongsino.github.io/2021/10/29/bioinfo_align_pep2cds/

# 2. KaKs_Calculator 2.0
KaKs_Calculator 2.0工具包包含了17种计算Ka和Ks的方法，包含Gamma系列，并可识别基于蛋白编码序列的滑动窗口，基于C++和Java，在Windows和Linux平台都可使用。

# 3. Ka和Ks计算
- ParaAT比对指定的基因对的氨基酸序列，并转化成比对的CDS序列，并可指定输出格式，如axt格式。
- KaKs_Calculator用于计算比对好的基因对的kaks。

用还会用到两个脚本:
- axt2one-line.py:https://github.com/scbgfengchao/4DTv/blob/master/axt2one-line.py转换axt格式为单行
- calculate_4DTV_correction.pl:https://github.com/JinfengChen/Scripts/blob/master/FFgenome/03.evolution/distance_kaks_4dtv/bin/calculate_4DTV_correction.pl计算4dtv。

# 4. 软件准备
1. 安装ParaAT.pl
- 参考blog.ParaAT：https://yanzhongsino.github.io/2021/10/29/bioinfo_align_pep2cds/

```shell
wget ftp://download.big.ac.cn/bigd/tools/ParaAT2.0.tar.gz
tar -zxf ParaAT2.0.tar.gz
cd ParaAT2.0
ParaAT.pl -h
```

2. 安装KaKs_Calculator2.0
KaKs_Calculator2.0下载地址：https://sourceforge.net/projects/kakscalculator2/?source=typ_redirect

```shell
wget https://altushost-swe.dl.sourceforge.net/project/kakscalculator2/KaKs_Calculator2.0.tar.gz
tar -zxf KaKs_Calculator2.0.tar.gz
chmod 777 ./KaKs_Calculator2.0/bin/Linux/KaKs_Calculator
chmod 777 ./KaKs_Calculator2.0/src/AXTConvertor
```

- 然后把`./KaKs_Calculator2.0/bin/Linux/`和`./KaKs_Calculator2.0/src/`添加到环境变量即可使用`KaKs_Calculator`和`AXTConvertor`命令。

# 5. 文件准备
ParaAT.pl需要三个输入文件，参考blog.ParaAT：https://yanzhongsino.github.io/2021/10/29/bioinfo_align_pep2cds/

1. sample.id
- 两列，每行对应两条要做成对比对的序列ID；
- 任意行，ParaAT可以批量处理多个成对比对。
- 一个例子：`cat sample.collinearity |grep "species_prefix"|cut -f2,3 >sample.id` 用MCScanX的结果文件提取blocks的同源gene对，获得sample.id文件。

2. cds.fa
- 包括所有需要比对的cds序列的文件

3. pep.fa
- 包括所有需要比对的蛋白序列的文件

# 6. 操作步骤
## 6.1. 用ParaAT获取基因对比对序列
ParaAT比对sample.id指定的基因对的氨基酸序列，并转化成比对的CDS序列，并可指定输出为axt格式。

1. 命令

```shell
echo "24" >proc #指定ParaAT.pl使用线程
ParaAT.pl -g -t -h sample.id -n cds.fa -a pep.fa -m mafft -p proc -f axt -o sample.paraat 2> paraat.log & #用ParaAT.pl调用mafft做每对基因的蛋白比对，并把蛋白比对转化成cds比对，输出axt格式。-g移除比对有gap的密码子，-t移除mismatched codons；-o指定生成目录；
```

2. notes
- axt格式包括三行，第一行两个序列ID之间用短横杠-相连，第二行第一条序列，第三行第二条序列。
- 建议加上-g和-t，免得后面计算Ks时报错`Error. The size of two sequences in 'ctg00816-ctg08844' is not equal。`
- ParaAT.pl命令中加上-k参数可以在获得axt文件后自动调用KaKs_Calculator计算kaks值，使用MA模型，比YN模型慢很多，推荐不加-k参数，而是手动用KaKs_Calculator的YN模型，生成sample.axt_yn.kaks文件。

## 6.2. 用KaKs_Calculator计算基因对的Ka、Ks和4dtv值
ParaAT.pl的-k参数只能指定KaKs_Calculator的MA模型计算kaks值，如果需要指定其他的模型，则可以手动运行计算。

KaKs_Calculator可计算比对好的CDS序列的Ka和Ks。

1. 计算Ka和Ks
- 获得all.kaks文件
```shell
cd sample.paraat # 进入ParaAT.pl生成的文件夹
for i in `ls *axt`;do KaKs_Calculator -i $i -o ${i}_yn.kaks -m YN;done #用YN模型计算每个gene对的KaKs，生成四列数据，gene对，Ka，Ks，Ka/Ks
for i in `ls |grep "_yn.kaks"`;do awk 'NR>1{print $1"\t"$3"\t"$4"\t"$5}' $i >>../all.kaks;done #合并kaks到all.kaks文件
```

2. 计算4dtv
- 获得all.4dtv文件
```shell
cd sample.paraat # 进入ParaAT.pl生成的文件夹
for i in `ls *axt`;do axt2one-line.py $i ${i}.one-line;done #多行axt文件转换成单行
ls |grep "axt.one-line"|while read id;do calculate_4DTV_correction.pl $id > ${id%%one-line}4dtv;done #计算4dtv值，生成两列数据，gene对，4dtv
for i in `ls |grep "4dtv"`;do awk 'NR>1{print $1"\t"$3}' $i >>../all.4dtv;done #合并4dtv值到all.4dtv
```

3. 合并和整理结果
- 获得all.results文件
```shell
cd .. #返回上一级目录
join all.kaks all.4dtv |sed "s/ /\t/g" |awk '$3 != "NA" {print $0}' |sed '1i\genepair\tKa\tKs\tKa/Ks\t4dtv_corrected' >all.results #以gene对为基准，join合并kaks和4dtv值到一个文件，然后过滤Ks值为NA的无效数据，添加标题行。
rm all.kaks* all.4dtv* #删除中间文件
```

# 7. references
1. ParaAT paper：https://www.sciencedirect.com/science/article/pii/S0006291X12003518
2. KaKs_Calculator2.0 github：https://github.com/kullrich/kakscalculator2
3. KaKs_Calculator2.0 paper：https://www.sciencedirect.com/science/article/pii/S1672022910600083?via%3Dihub

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>