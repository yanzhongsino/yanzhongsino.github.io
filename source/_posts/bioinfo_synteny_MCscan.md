---
title: 基因组间的同线性分析 —— MCscan
date: 2021-11-05 15:10:00
categories: 
- bioinfo
- synteny
tags:
- biosoft
- MCscan
- jcvi
- synteny
- colinearity

description: 用MCscan pipeline做两个基因组间的同线性分析，包括制作同线性点图(dotplot)，同线性深度直方图，同线性染色体图。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5250805&auto=1&height=32"></iframe></div>

# 1. MCscan背景
## 1.1. jcvi介绍
[jcvi](https://github.com/tanghaibao/jcvi)是美国佐治亚大学唐海宝开发的一个生物信息学python库。主要有几个常见pipeline用到了jcvi库，一个是做基因组scaffoldding的[ALLMAPS](https://github.com/tanghaibao/jcvi/wiki/ALLMAPS)，另一个是做基因组同线性分析的[MCscan](https://github.com/tanghaibao/jcvi/wiki/MCscan-%28Python-version%29)，以及提取植物表型数据的[GRABSEEDS](https://github.com/tanghaibao/jcvi/wiki/GRABSEEDS)。

## 1.2. MCscan介绍
用MCscan pipeline做两个基因组间的同线性分析，包括成对同线性(pairwise synteny)分析，宏观同线性(macrosynteny)分析，微观同线性(microsynteny)分析和结果的可视化(visualization)。

成对同线性分析主要是制作同线性点图(dotplot),同线性深度直方图；宏观同线性分析主要是制作同线性染色体图。其中同线性深度直方图，同线性染色体图需要用到同线性点图的结果文件作为输入。

微观同线性分析等后面用到再补坑。

# 2. MCscan使用
## 2.1. 安装jcvi
### 2.1.1. 依赖
jcvi依赖包括biopython,numpy,matplotlib。

MCscan依赖包括LASTAL。

LASTAL可以用`conda install -c bioconda last`安装，但我用conda安装的lastal和lastdb不能多线程运行，所以在运行`python -m jcvi.compara.catalog ortholog --cpu=1`时用参数`--cpu=1`来限制单线程运行。

### 2.1.2. 安装jcvi
任意选一种安装方式
1. `pip install jcvi`
2. `pip install git+git://github.com/tanghaibao/jcvi.git`
3. 
```
cd jcvi
git clone git://github.com/tanghaibao/jcvi.git
pip install -e .
```

## 2.2. 准备输入文件
输入文件包括两个基因组的CDS/氨基酸序列文件（fa格式），两个基因组的注释坐标文件（bed格式）。

### 2.2.1. jcvi支持直接从phytozome下载数据
1. 连接phytozome（需要登录，提前注册账号）
`python -m jcvi.apps.fetch phytozome` 输入账户登录信息后，出现可以下载的基因组list。

2. 下载CDS序列和注释文件(gff格式)，例如下载这两个物种:grape and peach
`python -m jcvi.apps.fetch phytozome Vvinifera,Ppersica`

3. 转换GFF为BED格式
`python -m jcvi.formats.gff bed --type=mRNA --key=Name  --primary_only Vvinifera_145_gene.gff3.gz -o grape.bed`

`python -m jcvi.formats.gff bed --type=mRNA --key=Name  --primary_only Ppersica_139_gene.gff3.gz -o peach.bed`

其中`--primary_only`是用于去除多个转录本的，只保留一个。（原文是多个isoforms）一般不去也不影响后续步骤，加上更好吧。

4. 整理一下，获得CDS文件
`python -m jcvi.formats.fasta format Vvinifera_145_Genoscope.12X.cds.fa.gz grape.cds`

`python -m jcvi.formats.fasta format Ppersica_298_v2.1.cds.fa.gz peach.cds`


### 2.2.2. 自己的数据
1. gff2bed
`python -m jcvi.formats.gff bed --type=mRNA --key=ID --primary_only sampleA.gff3.gz > sampleA.bed`

`python -m jcvi.formats.gff bed --type=mRNA --key=ID --primary_only sampleB.gff3.gz > sampleB.bed`

其中`--primary_only`是用于去除多个转录本的，只保留一个。（原文是多个isoforms）一般不去也不影响后续步骤，加上更好吧。

2. 去重，会生成sampleA.uniq.bed和sampleB.uniq.bed，然后重命名
```shell
python -m jcvi.formats.bed uniq sampleA.bed
mv sampleA.uniq.bed sampleA.bed
python -m jcvi.formats.bed uniq sampleB.bed
mv sampleB.uniq.bed sampleB.bed
```

3. 提取cds和pep
```shell
seqkit grep -f <(cut -f 4 sampleA.uniq.bed ) sampleA.cdna.all.fa.gz | seqkit seq -i > sampleA.cds
seqkit grep -f <(cut -f 4 sampleA.uniq.bed ) sampleB.pep.all.fa.gz | seqkit seq -i > sampleA.pep 

seqkit grep -f <(cut -f 4 sampleB.uniq.bed )  sampeB.cdna.fa.gz | seqkit seq -i  > sampleB.cds
seqkit grep -f <(cut -f 4 sampleB.uniq.bed ) sampleB.pep.fa.gz | seqkit seq -i  > sampleB.pep
```

## 2.3. 同线性点图
### 2.3.1. CDS的同线性点图dotplot
1. 确保`sampleA.cds`,`sampleA.bed`,`sampleB.cds`,`sampleB.bed`四个文件在同一目录下。
2. 运行`python -m jcvi.compara.catalog ortholog --no_strip_names --cpu=1 sampleA sampleB`制作同线性点图。

`--cpu=1`是因为conda安装的lastal不支持多线程，所以用单线程。

3. 运行时如果报错`ERROR savefig failed. Reset usetex to False`是缺少dvipng包没能生成png格式图，但看看结果文件sampleA.sampleB.anchors还是完整生成了就可以进行下一步。

### 2.3.2. 蛋白的同线性点图
与CDS一样，cds数据换成pep数据，在运行参数里加上数据类型`--dbtype prot`即可。

`python -m jcvi.compara.catalog ortholog --dbtype prot --no_strip_names --cpu=1 sampleA sampleB`

`--cpu=1`是因为conda安装的lastal不支持多线程，所以用单线程。

### 2.3.3. 结果
会生成五个结果文件。
- sampleA.sampleB.last: 基于LAST的比对结果
- sampleA.sampleB.last.filtered: LAST的比对结果过滤串联重复和低分比对
- sampleA.sampleB.anchors: 高质量的同线性区块
- sampleA.sampleB.lifted.anchors:增加了额外的锚点，形成最终的同线性区块
- sampleA.sampleB.pdf:同线性点图

同线性点图一般可以看点连成的比较长的线，在同一横坐标范围有几根线，或者在同一纵坐标范围有几根线，从而判断两个基因组的倍数关系。

<img src="https://www.dropbox.com/s/32mmo0kfhtx5b8g/grape.peach.cscore.99.png?raw=1" width=100% title="同线性点图" alt="同线性点图" align=center/>

**<p align="center">Figure 1. 同线性点图结果示例**
from [MCscan jcvi viki](https://github-wiki-see.page/m/tanghaibao/jcvi/wiki/MCscan-%28Python-version%29)</p>

## 2.4. 同线性深度直方图
1. 在同线性点图运行成功获得sampleA.sampleB.anchors结果文件的前提下
2. 运行`python -m jcvi.compara.synteny depth --histogram sampleA.sampleB.anchors`
3. 生成sampleA.sampleB.depth.pdf，显示了同线性深度比例。

<img src="https://www.dropbox.com/s/hhx2dtryrum2gyo/grape.peach.depth.png?raw=1" width=100% title="同线性深度直方图" alt="同线性深度直方图" align=center/>

**<p align="center">Figure 2. 同线性深度直方图结果示例**
from [MCscan jcvi viki](https://github-wiki-see.page/m/tanghaibao/jcvi/wiki/MCscan-%28Python-version%29)</p>

## 2.5. 同线性染色体图
### 2.5.1. 输入文件
1. seqids文件
- 指定展示的染色体ID，两行对应两个物种，每行的染色体之间逗号隔开，不能有空行。
- seqids文件染色体的顺序对应同线性染色体图的顺序，可以根据前面得到的同线性点图来调整顺序使两个基因组的同线性对应，使图更清晰。

```
scaf001,scaf002,scaf003,scaf004,scaf005,scaf006,scaf007,scaf008,scaf009,scaf010,scaf011,scaf012,scaf013,scaf014
scaffold1,scaffold2,scaffold3,scaffold4,scaffold5,scaffold6,scaffold7
```

2. .simple文件
- 每行代表一个同线性区块
- 一共六列，前四列是两个物种的同线性区块的上下限基因，第五列是评分，第六列是方向。
- 用前面分析同线性点图的结果文件.anchors创建。
- 运行`python -m jcvi.compara.synteny screen --minspan=30 --simple sampleA.sampleB.anchors sampleA.sampleB.anchors.new`会生成sampleA.sampleB.anchors.simple文件。

3. layout文件
- 设置绘制参数。前三列是设置图的位置坐标信息，整个图的x和y轴都是[0-1]范围。第四列是设置旋转角度，第五列是设置染色体颜色，第六列是展示的标签，第七列是对齐方式，第八列指定bed文件。
- 注意, #edges下的每一行开头都不能有空格。用于指定simple文件。
```
# y, xstart, xend, rotation, color, label, va,  bed
 .6,     .2,    .8,       0,      lightblue, sample_A, top, sampleA.bed
 .4,     .2,    .8,       0,      lightpink, sample_B, top, sampleB.bed
# edges
e, 0, 1, sampleA.sampleB.anchors.simple
```

### 2.5.2. 运行
`python -m jcvi.graphics.karyotype seqids layout`

生成karyotype.pdf文件，为同线性双染色体图

<img src="https://www.dropbox.com/s/51k8jujyzage3oa/grape.peach.karyotype.png?raw=1" width=100% title="同线性双染色体图" alt="同线性双染色体图" align=center/>

**<p align="center">Figure 3. 同线性双染色体图结果示例**
from [MCscan jcvi wiki](https://github-wiki-see.page/m/tanghaibao/jcvi/wiki/MCscan-%28Python-version%29)</p>

### 2.5.3. tips
- 运行虽然报错`ERROR    savefig failed. Reset usetex to False.`，应该是没能生成png图，但还是生成了pdf图文件。
- 当label用的名称过长时，label和图容易重叠，没有好的办法解决，最好用短一点的label。

### 2.5.4. 个性化
#### 2.5.4.1. highlight指定同线性区块
如果要突出显示指定的同线性区块（一般用于展示染色体倍数关系），则在sampleA.sampleB.anchors.sample文件中的对应同线性区域行的前面加上`yellowgreen*`,表示这行的颜色用yellowgreen展示。

然后再运行`python -m jcvi.graphics.karyotype seqids layout`就会生成突出显示的图。

# 3. references
1. https://github.com/tanghaibao/jcvi/wiki/MCscan-(Python-version)
2. https://xuzhougeng.top/archives/Comparative-genomics-using-JCVI-part-one

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>