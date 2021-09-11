---
title: WGDI的安装和使用
date: 2021-09-11 14:30:00
categories: 
- bio
- biosoft
tags:
- WGDI
- WGD
- Ks

description: 记录软件WGDI的安装和使用。这里记录了WGDI用于基因组内共线性区块的鉴定，Ks计算，WGD鉴定。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# 1. WGDI介绍
WGDI (Whole-Genome Duplication Integrated analysis)是python写的命令行工具，用于多倍化和跨物种基因组比对的综合分析，包括鉴定基因组内共线性区块，Ks计算，全基因组复制事件(WGD, whole genome duplication)的数量和时间的鉴定。

WGDI主要的三个功能是：
- 多倍体推断
- 基因组同源性的层次推断
- 祖先染色体核型分析

# 2. WGDI安装
WGDI的依赖：PAML,MAFFT,MUSCLE,PAL2NAL,IQTREE。

安装有三种方法:
1. conda
`conda create -c bioconda -c conda-forge -n wgdi wgdi` 建议新建一个环境
2. pip
`pip install wgdi`
3. github
```
git clone https://github.com/SunPengChuan/wgdi.git
cd wgdi
python setup.py install
```

# 3. WGDI进行共线性分析
## 3.1. 初始输入文件
### 3.1.1. 输入文件
WGDI需要三个输入文件，分别是基因的位置信息文件sample.gff，染色体长度信息文件sample.len和BLAST的输出文件sample.blast.txt，要求格式如下：
- input.gff: 一共七列，以tab分隔，分别为chr(chromosome number)，geneid(gene name)，start，end，direction(strand+/-)，order(order of each chromosome,starting from 1, 每个chr内部从1开始的顺序），original(original id and not read,旧的id，不会读入)。
	和典型的GFF格式不一样
- input.len: 一共三列，以tab分隔，染色体长度信息和染色体上的基因个数，chr(chromosome number), chr_length(染色体长度bp), chr_gene_number(注释到单个染色体的基因数量)
- sample.blast.txt：蛋白质序列的自我比对，BLAST的-outfmt 6输出格式的文件sample.blast.txt

### 3.1.2. 准备输入文件
1. sample.blastp.txt
用蛋白序列建库，blastp进行蛋白序列的自我比对，获取-outfmt 6格式的输出文件sample.blastp.txt。命令如下：
`makeblastdb -in sample.pep.fa -dbtype prot`
`blastp -num_threads 32 -db sample.pep.fasta -query sample.pep.fa -outfmt 6 -evalue 1e-5 -num_alignments 20 -out sample.blastp.txt &`

使用脚本[generate_conf.py](https://github.com/xuzhougeng/myscripts/blob/master/comparative/generate_conf.py)从基因组genome.fa和注释文件sample.gff3中获取input.gff和input.len文件。命令是`python generate_conf.py -p input genome.fa sample.gff3`。

2. input.gff
用generate_conf.py脚本获取输入文件input.gff和input.len
`python generate_conf.py -p input genome.fa sample.gff3` # 用脚本生成WGDI的输入文件input.gff和input.len,如果存在contig没被注释到基因，会报错:
```
Traceback (most recent call last):
  File "generate_conf.py", line 117, in <module>
    chrom=chrom,lens=lens,count=gene_count[chrom]), file=len_file)
KeyError: 'contig030'
```
并在contig030的地方停止继续写入input.len，input.gff不受影响。

3. input.len
可以用下面的脚本get_len.sh获取input.len

```get_len.sh
## 获取contig的长度
samtools faidx genome.fa
cat genome.fa.fai |cut -f1,2 >chrom.len

## 获取contig的基因计数
cat sample.gff3 |awk '$3 == "gene" {print $1}' >chrom.tem
contig=$(head -1 chrom.tem |sed "s/[0-9]//g")
c_number=$(tail -1 chrom.tem|sed "s/[A-Za-z]//g")
for i in $(seq -w 1 $c_number); do echo -e $contig$i"\t"$(grep "$contig$i" chrom.tem |wc -l);done >chrom.count

## 合并生成input.len
join chrom.len chrom.count|sed "s/ /\t/g" >input.len

## 删除中间文件
rm chrom.tem chrom.len chrom.count
```

## 3.2. 共线性分析
WGDI的分析的参数都是在配置文件中进行设置，所以分析都需要先创建配置文件，然后修改配置文件，最后运行程序。
每个配置项中都有gff1，gff2；lens1，lens2；blast这五个参数，是相同的。

### 3.2.1. 绘制共线性点阵图
WGDI的d模块绘制基因组内的共线性点阵图，初略估计是否有基因复制区域。
共线性点阵图是把两个基因组（做WGD分析则是把一个基因组的两份复制）分别作为横纵坐标，把检测到的同源匹配基因在相应位置做点标记；点数量多且相邻时，会有点组成的线出现，线代表比较长的共线性区块，代表着历史上的复制事件。

1. 创建配置文件input.conf
`wgdi -d \? >input.conf`

创建input.conf配置文件，里面包含[dotplot]配置参数。

2. 修改配置文件
- input.conf配置文件的信息
```
[dotplot]
blast = blast file # sample.blastp.txt文件
gff1 =  gff1 file # 比对项1的input.gff文件
gff2 =  gff2 file # 比对项2的input.gff文件，如果是自我比对，则gff1和gff2一样
lens1 = lens1 file # 比对项1的input.len文件
lens2 = lens2 file # 比对项2的input.len文件，如果是自我比对，则lens1和lens2一样
genome1_name =  Genome1 name # 比对项1的基因组名称
genome2_name =  Genome2 name # 比对项2的基因组名称，如果是自我比对，与genome1_name一致
multiple  = 1 # 最好的同源基因对数量，输出结果图中会用红点表示
score = 100 # blast输出的score过滤
evalue = 1e-5 # blast输出的evalue过滤
repeat_number = 10 # 允许去除超过部分种群的同源基因数量
position = order
blast_reverse = false
ancestor_left = ancestor file or none # 点阵图左侧物种的祖先染色体区域，一般设置成none就好
ancestor_top = ancestor file or none # 点阵图顶侧物种的祖先染色体区域，一般设置成none就好
markersize = 0.5 # 点的尺寸
figsize = 10,10 # 图的尺寸
savefig = savefile(.png, .pdf, .svg) # 保存的图片结果名称和格式
```

- input.conf配置文件示例
```
[dotplot]
blast = sample.blastp.txt
gff1 =  input.gff
gff2 =  input.gff
lens1 = input.len
lens2 = input.len
genome1_name =  sample
genome2_name =  sample
multiple  = 1
score = 100
evalue = 1e-5
repeat_number = 10
position = order
blast_reverse = false
ancestor_left = none
ancestor_top = none
markersize = 0.5
figsize = 10,10
savefig = out.pdf
```

3. 运行
`wgdi -d input.conf`

4. 结果
结果保存在out.pdf的图中。

结果解释：
- 图中有三种颜色，红色表示genome2的基因在genome1中的最优同源匹配，次优的四个基因是蓝色，其余的是灰色。
- WGDI会过滤掉自身与自身的比对结果，所以图中对角线出现的片段不是自身比对结果，而是串联基因（tandem）形成的共线性区块。
- 如果基因组存在加倍事件，则基因组不同区域的同源基因的排布顺序会比较一致，在图上就能观察到多个点排列组成的“线”。

### 3.2.2. 获取共线性区块
WGDI的icl(Improved version of ColinearScan)模块用于获取共线性区块的具体位置信息。与MCScanX软件功能一致。

1. 建立配置文件
`wgdi -icl \? >>input.conf`

在已有配置文件input.conf的基础上添加[collinearity]配置参数。

2. 修改配置参数
```
[collinearity]
gff1 =  gff1 file # 比对项1的input.gff文件
gff2 =  gff2 file # 比对项2的input.gff文件，如果是自我比对，则gff1和gff2一样
lens1 = lens1 file # 比对项1的input.len文件
lens2 = lens2 file # 比对项2的input.len文件，如果是自我比对，则lens1和lens2一样
blast = blast file # sample.blastp.txt文件
blast_reverse = false
multiple  = 1 # 最好的同源基因对数量，输出结果图中会用红点表示
process = 8 # 线程
evalue = 1e-5 # blast输出的evalue过滤
score = 100 # blast输出的score过滤
grading = 50,25,10 # 为red,blue,gray三种颜色的点指定不同的分数
mg = 25,25 # 检测共线性区域的最大gap值，即共线性区块内含的最大空缺基因数量。
pvalue = 1 # 显著性，评估共线性blocks的紧密程度和独特程度的参数，取值范围是0-1，更好的共线性范围是0-0.2。
repeat_number = 10 # 允许去除超过部分种群的同源基因数量
positon = order
savefile = out.collinearity # 保存共线性结果的文件名
```

3. 运行
`wgdi -icl input.conf`

4. 结果
结果out.collinearity中记录着共线性区域。

\# Alignment起始的行记录着共线性区块的信息，包括得分(score),显著性(pvalue),基因对数(N)，strand(plus/minus)。

### 3.2.3. 计算KaKs
WGDI的ks模块计算共线性区块的基因对间的ka和ks。

1. 建立配置文件
`wgdi -ks \? >>input.conf`

在已有配置文件input.conf的基础上添加[ks]配置参数。

2. 修改配置参数
```
[ks]
cds_file =  input.cds.fa # 基因组(一个或多个)的cds序列
pep_file =  input.pep.fa # 基因组(一个或多个)的氨基酸序列；可选参数，如果不设置，将用biopython模块翻译cds文件获得。
align_software = muscle # 选择多序列比对软件:{muscle,mafft}
pairs_file = out.collinearity # tab分隔的共线性基因对。上一步共线性分析的结果文件，也支持MCScanX的共线性分析结果文件。
ks_file = out.ks # 保存ks结果的文件名
```

3. 运行
`wgdi -ks input.conf`

WGDI会用muscle根据氨基酸序列进行联配，然后用pal2pal.pl基于cds序列将氨基酸联配转为密码子联配，最后用paml中的yn00和ng86两种方法计算ka和ks。

4. 结果
out.ks结果有6列，对应的是每个基因的ka和ks。

```
$ head -2 out.ks
id1  id2  ka_NG86  ks_NG86  ka_YN00  ks_YN00
vvi161s1g00311	vvi161s1g00312	0.2986	1.2027	0.3047	1.3864
```

### 3.2.4. 整合共线性区块信息
WGDI的bi模块可以整合共线性区块和ks信息成一个可读性更强的csv文件。

1. 建立配置文件
`wgdi -bi \? >>input.conf`

在已有配置文件input.conf的基础上添加[blockinfo]配置参数。

2. 修改配置参数
```
[blockinfo]
blast = input.blastp.txt
gff1 =  input.gff
gff2 =  input.gff
lens1 = input.len
lens2 = input.len
collinearity = out.collinearity # 共线性区域结果文件
score = 100
evalue = 1e-5
repeat_number = 20
position = order
ks = out.ks # ks结果文件
ks_col = ks_NG86 # 声明使用ks结果文件的列名
savefile = block_info.csv
```

3. 运行
`wgdi -bi input.conf`

4. 结果

结果文件block_info.csv共11列：
- id：共线性结果的id
- chr1,start1,end1：点图左边的基因组共线性区块的位置范围
- chr2,start2,end2：点图上边的基因组共线性区块的位置范围
- pvalue：共线性结果评估，一般认为<0.01更合理
- length：共线性区块长度
- ks_median：共线性区块上所有基因对ks的中位数，用于评估ks分布
- ks_average：共线性区块上所有基因对ks的平均值
- homo1-n：multiple参数为n，则有n列，表示一个基因有n个最佳匹配时的取值，对应点阵图中的红点
- block1,block2：共线性区块上基因order的位置
- ks：共线性区块上基因对的ks值
- density1,density2：共线性区块的基因分布密集程度，值越大越密集
- class1,class2：在alignment模块中用到，对应两个block分组。默认值是0，表示两个block是同一组。这两列需要根据覆盖率、染色体核型等多个方面进行确定。

## 3.3. 根据ks分布拟合单次WGD事件
1. 背景知识
在Lynch和Conery在2000年发表在Science的论文中，他们证明了小规模基因复制的Ks分布是L型，而在L型分布背景上叠加的峰则是来自于演化历史中某个突然的大规模复制事件。
L型分布（呈指数分布, exponential distribution), 最初的峰可能是近期的串联复制引起，随着时间推移基因丢失，形成一个向下的坡。正态分布(normal distribution)的峰则是由全基因组复制引起。

这就意味着我们可以根据ks频率分布图的正态分布峰来判断物种历史上发生过的全基因组复制事件，并通过ks值拟合峰值获得WGD事件发生的时间。

2. 根据ks分布拟合单次WGD事件的峰
- 通过kspeaks(-kp)模块过滤blockinfo文件，根据ks峰值分布设置ks_area过滤参数，同时可以调整homo参数看看效果，得到单独一个WGD事件形成的共线性区块和相应的ks值。
- 接着用PeaksFit(-pf)模块对值的峰进行拟合得到模型参数。
- 最后用KsFigures(-kf)将拟合结果绘制到一张图上。
- 想看单个WGD事件的ks点阵图还可以把blockks的blockinfo参数换成kspeaks(-kp)模块过滤后得到的ks_median.distri.csv，跑一遍绘制ks点阵图(-bk)模块获取过滤后的ks点阵图。

因为用WGDI对ks频率分布图的峰进行拟合时，一次只能拟合一个峰。当发现两次或多次WGD事件后，需要两次或多次重复以上分析过程，分别获取WGD的ks拟合图。

## 3.4. ks拟合和可视化
### 3.4.1. ks可视化——绘制ks点阵图
第一步绘制的点阵图里包含基因组上检测到的所有同源基因对（所以点特别多），bk模块绘制的ks点阵图的点只包含确认了共线性的基因对，用ks值作为点的颜色信息，可以根据ks点阵图的共线性区域的颜色来区分不同时期的多倍化事件。

1. 建立配置文件
`wgdi -bk \? >>input.conf`

在已有配置文件input.conf的基础上添加[blockks]配置参数。

2. 修改配置参数
```
[blockks]
lens1 = input.len
lens2 = input.len
genome1_name =  sample
genome2_name =  sample
blockinfo =  block_info.csv # bi模块整合结果
pvalue = 0.2 # 共线性区块的显著性，对应blockinfo的pvalue列
tandem = true # 是否过滤串联基因形成的共线性区（在点阵图上对角线附近）
tandem_length = 200 # 如果tandem=true，那么评估tandem的标准为两个区块的物理距离
markersize = 1
area = 0,2 # 输出结果中ks的取值范围
block_length =  5 # 一个共线性区块的最小基因对数量，对应blockinfo的length列
figsize = 8,8
savefig = ks.dotplot.pdf
```

3. 运行
`wgdi -bk input.conf`

4. 结果
结果保存在ks.dotplot.pdf文件中。

结果解释：
- 图中每个点都是共线性区块的基因对，点的颜色是ks值。
- 可以用共线性区块的ks中位数来初步判断复制发生的时间，所以图中颜色相近的共线性区块看作同时发生的复制，当图中大致观察到两种颜色的点和线时，表示对应的两次全基因组复制事件。

### 3.4.2. ks可视化——过滤后绘制ks频率分布图
通过计算共线性区块的基因对ks值，可以获得基因对复制发生的时间，如果有全基因组复制（WGD）发生，那么现有物种的基因组会留下许多ks相近的基因对，通过ks频率分布图可以看到峰，由此判断WGD的发生次数和发生时间。

1. 建立配置文件
`wgdi -kp \? >>input.conf`

在已有配置文件input.conf的基础上添加[kspeaks]配置参数。

2. 修改配置参数
```
[kspeaks]
blockinfo = block_info.csv
pvalue = 0.2 # 共线性区块的显著性，对应blockinfo的pvalue列
tandem = true # 是否过滤串联重复基因形成的共线性区（在点阵图上对角线附近）
block_length = 5 # 共线性区块的基因对的数目
ks_area = 0,10 # 对应blockinfo的ks列，0-10表示只保留ks在0-10的基因对。
multiple  = 1 # 选择homo中的哪列用于后续过滤，一般选homo1
homo = -1,1 # 根据homo的共线性区块中基因对的总体得分（取值范围-1，1，值越大表示最佳匹配的基因对越多，也就是越近期的WGD事件得到的基因对）对共线性区块进行过滤，只取得分在-1-1之间的共线性区块，即不根据homo过滤。
fontsize = 9
area = 0,3
figsize = 10,6.18
savefig = ks_median.distri.pdf # ks频率分布图结果
savefile = ks_median.distri.csv # 对block_info.csv的过滤后结果，与block_info.csv格式一致
```

3. 运行
`wgdi -kp input.conf`

根据配置参数过滤blockinfo文件的低质量数据，再绘制频率分布图。

4. 结果
结果保存在ks_median.distri.pdf（ks峰图）和ks_median.distri.csv（输入文件block_info.csv过滤后结果，与block_info.csv格式一致）。

从峰图的峰的数量可以估计WGD的数量。

5. 说明
因为用WGDI对ks频率分布图的峰进行拟合时，一次只能拟合一个峰。当发现两次或多次WGD事件后，需要两次或多次重复以上分析过程，分别获取WGD的ks拟合图。

所以也可以先不过滤绘制ks频率分布图，初步查看ks的分布情况；再根据每次WGD事件的分布设置ks_area,multiple,homo三个参数来分离单次WGD事件的数据，再进行过滤-拟合-作图。

三个参数的设置：
- ks_area：根据单次WGD的ks分布范围设置，过滤出单次WGD的数据。
- multiple：选择不同的homo，一般没有特殊需求，选择最优的，即1。
- homo：设置不同的homo范围，观察做出的图中峰的变化来确定homo范围是否合理。

### 3.4.3. 拟合ks频率分布图的峰

1. 建立配置文件
`wgdi -pf \? >>peak.conf`

在已有配置文件input.conf的基础上添加[kspeaks]配置参数。

2. 修改配置参数
```
[peaksfit]
blockinfo  =  ks.distri.csv
mode  =  median
bins_number  =  200
ks_area  =  0,10
fontsize  =  9
area  =  0,3
figsize  =  10,6.18
savefig  =  ks.peaksfit.pdf 
```

3. 运行
`wgdi -pf peak.conf`

得到拟合图ks.peaksfit.pdf和一些参数(包括一个R-square和3个拟合参数the gaussian fitting curve parameters）;拟合参数用于后续ksfigure模块分析。

### 3.4.4. 拟合结果作图——kf模块
1. 建立配置文件
在已有配置文件input.conf的基础上添加[ksfigure]配置参数。
`wgdi -kf \? >>input.conf`

2. 创建all_ks.csv文件
- all_ks.csv文件内容
第一行为标题行，之后行为数据行。
共有4+3n列(n是peak数量，即WGD的次数），逗号分隔，第一列是样本信息，第2-3列对应线条属性(color,linewidth,linestyle)，后面列都是拟合参数（the gaussian fitting curve parameters拟合参数，每个peak有3个拟合参数，n个peaks有3n个，依次列出。
标题行要包含所有列（可为空）；数据行根据各自的峰数量可能有不同列数（比如一个峰的7列，两个峰的10列）。

- all_ks.csv文件示例
两个样本，三次比对(sample1_sample1,sample2_sample2,sample1_sample2)，两个峰的all_ks.csv文件示例：
```
,color,linewidth,linestyle,,,,,,
sample1_sample1,green,1,--,5.02360835744403,0.8319599832352784,0.10382203381206191,2.084853812322066,1.8332872127128195,0.2506813629824027
sample2_sample2,red,1,--,5.02360835744403,0.8319599832352784,0.10382203381206191,2.084853812322066,1.8332872127128195,0.2506813629824027
sample1_sample2,yellow,1,-,3.00367275,1.288717936,0.177816426
```

3. 修改配置参数
```
[ksfigure]
ksfit = all_ks.csv # 拟合参数文件
labelfontsize = 15
legendfontsize = 15
xlabel = none
ylabel = none
title = none
area = 0,4
figsize = 10,6.18
savefig =  all_ks.pdf #保存的拟合图
```

4. 运行
`wgdi -kf input.conf`

得到all_ks.pdf拟合图

# 4. references
[WGDI's documentation](https://WGDI.readthedocs.io/en/latest/)
[WGDI's github](https://github.com/SunPengChuan/wgdi)
[xuzhougeng's WGDI上](https://www.jianshu.com/p/a50548e81ac0)
[xuzhougeng's WGDI中](https://www.jianshu.com/p/0e4a2807468d)
[xuzhougeng's WGDI下](https://www.jianshu.com/p/28a4c3045919)
[xuzhougeng's WGDI blockinfo blog](https://www.jianshu.com/p/e97fdcf5d06f)