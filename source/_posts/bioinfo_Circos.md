---
title: Circos圈图绘制
date: 2021-11-16 10:10:00
categories: 
- bio
- bioinfo
tags: 
- Circos
description: 记录了绘制圈图的软件Circos的使用逻辑和参数。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=4278314&auto=1&height=32"></iframe></div>


# 1. 介绍circos圈图
介绍基因组的文章里常常出现下面的circos圈图，这篇博客记录了如何绘制这种圈图。

<img src="http://circos.ca/intro/genomic_data/img/circos-conde-nast.png" width=80% height=80% title="Circos圈图" alt="Circos圈图" />

circos圈图的内核是把矩形掰弯成环状，矩形信息的展示以行和列为基础，环形除了行和列，还可以增加环的不同位置之间的关系，比如基因组特征的Circos图中间常见的染色体共线性关系的展示。

# 2. 绘制circos圈图的软件和包
有许多可以绘制circos圈图的软件和包。常见的有[Circos软件](http://circos.ca/),TBtools，R包circlize，python版本的Circos等。

## 2.1. Circos —— 从名字就可以看出是circos绘制软件的霸王
[Circos软件](http://circos.ca/)是2009年Michael Smith Genome Sciences Centre的Martin Krzywinski开发的一款基于perl的做比较基因组Circos圈图绘制的软件，功能完备，被文章里使用的非常多。Martin Krzywinski是生物信息学家和专业摄影师双重身份，所以Circos软件的审美设计是非常可靠的（当然得在合理的配色和布局下，我就用过同一套数据生成两个美学差异极大的图）。

Circos有在线版本[Circos Online](http://mkweb.bcgsc.ca/tableviewer/)，在线使用时是把表格转为圈图，不过只允许最大75行和75列。

下面记录了用Circos绘制基因组圈图的基础操作。

### 2.1.1. Circos安装
`conda create -c bioconda -n circos circos` #单独环境安装

`circos -V` #确认版本

### 2.1.2. Circos运行
`circos -conf circos.conf`

会生成circos.png和circos.svg两个文件。

### 2.1.3. Circos模块化绘制
Circos是模块化绘制的方式，即通过增添数据和设置来实现每个部分的绘制。

基因组圈图中的几个基础模块：染色体模块，特征图模块，共线性模块。
- 染色体模块显示了染色体的数量和长度，常常添加标尺，通常在最外圈。
- 特征图模块可以有多圈，可以是折线图(line)，散点图(scatter)，直方图(histogram)和热图(heatmap)等。通常用直方图或者热图展示基因密度，重复序列密度，GC占比等信息。
- 共线性模块通常放在中心，代表不同染色体间的共线性关系。如果有多个物种，则可以体现物种间的共线性关系。

### 2.1.4. Circos输入文件
输入文件包括数据文件和配置文件。

需要展示的模块数据都保存在单独的数据文件中，然后在配置文件circos.conf中调用数据文件。

有些教程会建议多制作几个不同类别的配置文件，然后在circos.conf调用其他配置文件。
比如单独设置ticks.conf,links.conf，然后在circos.conf里加入下面内容调用
```
<<include ticks.conf>>
<<include links.conf>>
```

比如单独设置plots_histogram.conf，plots_heatmap.conf，plots_text.conf，然后在circos.conf里加入下面内容调用
```
<plots>
<<include plots_histogram.conf>>
 <<include plots_heatmap.conf>>
 <<include plots_text.conf>>
</plots>
```

但如果不是设置特别复杂，直接使用circos.conf一个配置文件即可，所有设置都可以写在circos.conf。

#### 2.1.4.1. 主配置文件circos.conf
##### 2.1.4.1.1. circos.conf由几个部分组成
- <ideogram>绘制染色体
- <ticks>绘制染色体刻度
- <links>绘制染色体间连线（通常代表共线性）
- <plots>绘制图形

##### 2.1.4.1.2. circos.conf示例 

```circos.conf

# 1. 染色体和刻度的设置
karyotype = karyotype.txt #指定染色体数据文件
chromosomes_order = scaf001,scaf002,scaf003,scaf004,contig003,contig006 #指定染色体顺序，若不指定则按照染色体数据文件karyotype.txt的顺序
chromosomes_reverse = /scaf/ #染色体坐标默认是从0到最大值，通过正则匹配把染色体坐标改成最大值到0
chromosomes_scale = /scaf/=0.5rn,/contig/=0.5rn #设置染色体缩放，scaf的所有染色体占据50%的空间，contig的所有染色体占据50%的空间。

<<include etc/colors_fonts_patterns.conf>>
<<include etc/housekeeping.conf>>

chromosomes_units = 1000000 #定义长度单位，后面使用u赋值长度
show_ticks = yes #显示染色体刻度
show_tick_labels = yes #显示刻度标签

# 2. ideogram模块，用于染色体设置
<ideogram>
<spacing>
default = 1u #染色体间间距
 <pairwise scaf001 contig001>
  spacing = 10u
 </pairwise> # 在scaf001和contig001之间流出10u的间距（其他染色体默认是1u）
 <pairwise scaf004 contig006>
  spacing = 10u
 </pairwise> # 在scaf004和contig006之间流出10u的间距（其他染色体默认是1u）
</spacing>
radius = 0.90r #染色体位置
thickness = 20p #染色体的厚
fill = yes
stroke_color = dgrey #颜色
stroke_thickness = 2p

show_label = yes #显示label，染色体名称
label_font = default #字体
label_radius = dims(ideogram,radius) + 0.05r #染色体标签位置
label_size = 48 #字体大小
label_parallel = yes #是否平行
label_format = eval(sprintf("%s",var(chr))) #格式

</ideogram>

# 3. ticks模块，用于染色体刻度设置
<ticks>
radius = 1r #刻度位置
color = black #刻度颜色
thickness = 2p #刻度厚度
multiplier = 1e-6 #输出标签为实际长度与其相乘，实际位置为chromosomes_units*multiplier
format = %d # %d表示显示整数

<tick>
spacing = 1u #刻度间距离，1u表示一个长度单位，即1chromosomes_units
size = 5p #tick的长度
</tick>

<tick>
spacing = 5u
size = 10p
show_label = yes #是否展示标签
label_size = 20p #标签尺寸
label_offset = 10p #让标签往外偏移一点
format = %d #显示整数
</tick>
</ticks>

# 4. image模块，用于输出设置
<image>
dir* = . #输出目录，参数后*表示覆盖已有设置
radius* = 1500p #输出图片半径
angle_offset* = -85 #默认情况下，第一个染色体从-90°位置开始，但设置scaf001和contig001之间的空隙为10u会导致图看起来不对称，通过angle_offset调整角度使图对称。
<<include etc/image.conf>>
</image>

# 5. plots模块，用于特征图形设置
<plots>
<plot>
show = True #是否展示该图形，默认展示
type = scatter #展示的图形类型，折线图(line)，散点图(scatter)，直方图(histogram)和热图(heatmap)
file = genes_num.txt #数据文件
r1 = 0.98r #在圈图中的位置，外径
r0 = 0.89r #在圈图中的位置，内径
min = 10 #展示的数据下限
max = 1000 #展示的数据上限
glyph = circle #散点图的符号类型，圆(circle), 矩形(rectangle), 三角形(triangle)
glyph_size = 2p #散点图符号大小，单位为p
color = pastel1-5-qual-2 #散点图符号颜色，直方图外部线的颜色
stroke_color = pastel1-5-qual-2 #对于散点图，符号外部的颜色
stroke_thickness = 1p #对于散点图，符号外部线的厚度
</plot>

<plot>
type = histogram #直方图
file = gc_rate.txt #数据文件
fill_color = pastel1-5-qual-1 #直方图填充色
color = pastel1-5-qual-2 #散点图符号颜色，直方图外部线的颜色
r1 = 0.78r
r0 = 0.69r
</plot>

<plot>
type = heatmap #热图
file = repeats_num.txt #数据文件
fill_color = pastel1-5-qual-2
r1 = 0.88r
r0 = 0.79r
</plot>

<plot>
type = line #折线图
file = col.txt #数据文件
fill_color = pastel1-5-qual-4
r1 = 0.68r
r0 = 0.59r
</plot>
</plots>

# 6. links模块，用于共线性设置
<links>
<link>
file = anchors.simple_link.txt #共线性数据文件
radius = 0.58r #位置外界
color = blue_a4 #颜色
ribbon = yes

# 6plus. rules模块，放在link内部，用于为共线性模块不同染色体用不同颜色渲染
<rules>
<rule>
condition = var(chr1) eq "scaf001" #判断link文件中左侧染色体`var(chr1)`的名字是不是`eq` "scaf001"，如果是颜色就进行下面设置。
color = pastel1-9-qual-9 #颜色
</rule>

<rule>
condition = var(chr1) eq "scaf002"
color = pastel1-9-qual-7
</rule>

<rule>
condition = var(chr1) eq "scaf003"
color = pastel1-9-qual-6
</rule>

<rule>
condition = var(chr1) eq "scaf004"
color = pastel1-9-qual-8
</rule>

<rule>
condition = var(chr1) eq "scaf005"
color = pastel1-9-qual-5
</rule>

<rule>
condition = var(chr1) eq "scaf006"
color = pastel1-9-qual-4
</rule>

<rule>
condition = var(chr1) eq "scaf007"
color = pastel1-9-qual-3
</rule>

<rule>
condition = var(chr1) eq "scaf008"
color = pastel1-9-qual-2
</rule>

<rule>
condition = var(chr1) eq "scaf009"
color = pastel1-9-qual-1
</rule>

<rule>
condition = var(chr1) eq "scaf010"
color = pastel1-5-qual-5
</rule>

<rule>
condition = var(chr1) eq "scaf011"
color = pastel1-5-qual-4
</rule>

<rule>
condition = var(chr1) eq "scaf012"
color = pastel1-5-qual-3
</rule>

<rule>
condition = var(chr1) eq "scaf013"
color = pastel1-5-qual-2
</rule>

<rule>
condition = var(chr1) eq "scaf014"
color = pastel1-5-qual-1
</rule>
</rules>

</link>
</links>
```

##### 2.1.4.1.3. circos.conf参数详解
1. 颜色选择
Circos的颜色用的是[colorbrewer2网站](https://colorbrewer2.org/)的颜色，也可以参考[circos的colors网址](http://circos.ca/documentation/tutorials/configuration/colors/images)。

Circos中颜色的命名格式为PALETTE-NUMCOLORS-TYPE-IDX：
- PALETTE:调色版名，如rdylbu
- NUMCOLORS: 颜色数目, 11
- 调色版类型: div(diverging), seq(sequential), qual(qualitative)
- IDX: 调色版中的颜色索引

所以brbg-3-div-1代表的是[circos的colors网址](http://circos.ca/documentation/tutorials/configuration/colors/images)里的brbg-div的3组的第一个颜色。

2. 长度单位
控制图形不同元素大小的三个单位，p,r,u。
- p(pixels), 表示绝对大小；如果使用p作为单位，需要参考circos.conf中设置输出图形模块的<image>里定义的radius输出图片半径。
- r(relative), 相对大小；会随着最终图形大小而发生变换。
- u(chromosome unit)，定义chromosome unit之后使用；一般在显示刻度时使用。。

#### 2.1.4.2. 数据文件 —— 染色体文件karyotype.txt
染色体文件karyotype.txt用于生成染色体模块。

##### 2.1.4.2.1. 染色体文件karyotype.txt的获取
从genome.fa.fai里生成karyotype.txt文件`head -12 genome.fa.fai |awk '{print "chr - "$1" "$1" 0 "$2" set3-12-qual-12"}' >karyotype.txt`，取前12条染色体。

修改最后一列来定义染色体颜色，推荐`set3-12-qual`系列，一共12种，可以依次给到12条染色体。

#### 2.1.4.3. 数据文件 —— 线性关系文件_link.txt
线性关系文件_link.txt用于生成共线性模块。

格式是六列数据，用tab分隔，每行都代表一条线，由具有共线性的两组染色体ID、起始位置和终止位置组成。

`contig003  7719    355757  contig014   3459    349346`

##### 2.1.4.3.1. 线性关系文件_link.txt的获取
可以用jcvi生成simple文件，然后用脚本simple2links.py转换成_link.txt文件。

`pip install jcvi`安装jcvi，可以参考博文[MCscan](https://yanzhongsino.github.io/2021/11/05/bioinfo_MCscan/)。

1. 准备sample.bed和sample.cds两个文件
```shell
python -m jcvi.formats.gff bed --type=gene --key=ID sample.gff3 >sample.bed #gff2bed
python -m jcvi.formats.bed uniq sample.bed #生成sample.uniq.bed
seqkit grep -f <(cut -f 4 sample.uniq.bed )  sample.cds.fa | seqkit seq -i > sample.cds #提取cds
mv sample.uniq.bed sample.bed
rm sample.leftover.bed
```

2. 运行共线性分析
```shell
python -m jcvi.compara.catalog ortholog --no_strip_names --cpu=1 sample sample #生成sample.sample.anchors，单个物种就用sample sample，两个物种就准备两套.bed和.cds文件，用sample1 sample2。
python -m jcvi.compara.synteny screen --minspan=30 --simple sample.sample.anchors sample.sample.anchors.new #生成sample.sample.anchors.simple
```

3. MCScanX
从MCScanX结果sample.collinearity中生成类似MCSCAN结果的bv.simple。

```
grep -C 1 "#" sample.collinearity |grep -v "^-"  >sample.col
tail -1 sample.collinearity >> sample.col
cat sample.col |sed -e "s/#.* /#/g" -e "s/[^b]*bv/bv/" |tr '\n' ' '|sed "s/ #/\n/g"|awk '{print $2,$3,$5,$6" 200 "$1}'|sed -e "s/plus/+/g" -e "s/minus/-/g" |grep -v "#" >sample.simple
```

4. simple2links
`python simple2links.py sample.sample.anchors.simple #生成sample.sample.anchors.simple_link.txt`

sample.sample.anchors.simple_link.txt可以作为circos画共线性的输入文件

[simple2links.py脚本地址](https://github.com/xuzhougeng/myscripts/blob/master/simple2links.py)

```python
#!/usr/bin/env python
# simple2links.py

from sys import argv

simple_file = argv[1]

ref_bed = simple_file.split(".")[0] + ".bed"
qry_bed = simple_file.split(".")[1] + ".bed"

ref_dict = {line.split("\t")[3]:line.split("\t")[0:3] for line in open(ref_bed)}
qry_dict = {line.split("\t")[3]:line.split("\t")[0:3] for line in open(qry_bed)}

fo = open(simple_file + "_link.txt", "w")

for line in open(simple_file):
    if line.startswith("#"):
        continue
    items = line.strip().split("\t") 
    ref_start_gene = items[0]
    ref_end_gene = items[1]
    qry_start_gene = items[2]
    qry_end_gene = items[3]
    
    ref_chr, ref_start = ref_dict[ref_start_gene][0:2]
    ref_end = ref_dict[ref_end_gene][2]
    qry_chr, qry_start = qry_dict[qry_start_gene][0:2]
    qry_end = qry_dict[qry_end_gene][2]
    
    circos_input = [ref_chr, ref_start, ref_end, qry_chr, qry_start, qry_end]
    fo.writelines('\t'.join(circos_input) + '\n')
    
fo.close()
```

#### 2.1.4.4. 数据文件 —— 特征图文件
特征图文件用于生成折线图(line)，散点图(scatter)，直方图(histogram)和热图(heatmap)等特征图模块。

##### 2.1.4.4.1. 获取特征图文件
特征图文件格式相同，由四列组成，染色体ID(chr)，起始位置(start)，终止位置(end)，特征数据(value[options])。

前三列是bed格式，最后一列是特征数据，比如在这个位置有几个基因。通常用法是展示染色体的基因密度信息、重复序列密度和GC含量等信息。

1. 准备
```shell
cut -d ' ' -f 3,6 karyotype.txt | tr ' ' '\t' >genome.txt #获取基因组文件
bedtools makewindows -g genome.txt -w 100000 >genome.windows #以100kb为滑窗，沿染色体创建窗口
```
2. 基因密度
```shell
zgrep '[[:blank:]]gene[[:blank:]]' sample.gff3 |awk '{print $1"\t"$4"\t"$5}' >genes.bed #提取基因位置信息
bedtools coverage -a genome.windows -b genes.bed | cut -f 1-4 >genes_num.txt #按照滑窗统计基因密度
```
3. 重复序列密度
```shell
awk '{print $1"\t"$4"\t"$5}' repeats.gff |grep -v "#" >repeats.bed #提取重复位置信息
bedtools coverage -a genome.windows -b repeats.bed | cut -f 1-4 >repeats_num.txt #按照滑窗统计重复密度
```
4. GC含量
`bedtools nuc -fi genome.fa -bed genome.windows | cut -f 1-3,5 | sed '1d' > gc_rate.txt #按照滑窗统计GC含量`

5. 共线性区块的基因密度
```shell
cat bv.col|sed -e "s/#.* /#/g" -e "s/[^b]*bv/bv/" |tr '\n' ' '|sed "s/ #/\n/g"|awk '{print $2,$3,$5,$6}'|grep -v "#" |sed "s/ /\n/g"|sed "/^$/d" |sort |uniq >bv.col.list #获取共线性区块包含的基因列表
grep -f col.list Bauhinia.variegata.gene.gff3 |awk '$3 == "gene" {print $1"\t"$4"\t"$5}' >col.bed #获取基因的位置信息
bedtools coverage -a genome.windows -b col.bed |cut -f 1-4 >col.txt #统计共线性基因密度
```

# 3. references
[Circos paper](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2752132/)
[xuzhougeng 1](https://www.jianshu.com/p/3a31ceef711b)
[xuzhougeng 2](https://www.jianshu.com/p/4b3d3809ac07)
[xuzhougeng 3](https://www.jianshu.com/p/ea3a8933ace9)
[xuzhougeng 4](https://www.jianshu.com/p/1658e702ba17)
[xuzhougeng 5](https://www.jianshu.com/p/31f26d1e5974)