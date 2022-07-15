---
title: genome survey
date: 2022-01-30 14:30:00
categories:
- bio
- bioinfo
tags:
- genome survey

description: 用Astral软件基于基因组数据建物种树。
---

<div align="middle"><music URL></div>



## ASTRAL的安装
### ASTRAL的安装
1. git clone安装
```
git clone https://github.com/smirarab/ASTRAL.git #克隆ASTRAL的github库，生成ASTRAL目录
cd ASTRAL
unzip Astral.5.7.8.zip #解压缩
```
解压缩后生成Astral目录，里面的`astral.5.7.8.jar`可直接使用。


2. 下载压缩包
```
wget https://github.com/smirarab/ASTRAL/archive/refs/heads/master.zip #下载压缩包
unzip master.zip #解压缩
```

解压缩默认生成ASTRAL-master目录，里面的`astral.5.7.8.jar`可直接使用。


### ASTRAL-MP —— 多线程版本【推荐】
1. git clone安装
```
git clone https://github.com/smirarab/ASTRAL.git #克隆ASTRAL的github库，生成ASTRAL目录
cd ASTRAL
git checkout MP #转换github分支到MP
unzip Astral.5.15.5.zip #解压缩
```
解压缩后生成Astral目录，里面的`astral.5.15.5.jar`可直接使用。


2. 下载压缩包
```
wget https://github.com/smirarab/ASTRAL/archive/refs/heads/MP.zip #下载压缩包
unzip MP.zip #解压缩
```

解压缩默认生成ASTRAL-MP目录，里面的`astral.5.15.5.jar`可直接使用。

### 【可选】安装AVX2 —— 加速4倍


运行报错

```
A fatal error has been detected by the Java Runtime Environment:
SIGILL (0x4) at pc=0x00007fd92c19b34a, pid=3597678, tid=3597891
```

想要加速，还可以增加java使用的内存来实现。java -jar -Xmx100G

# 基因树
raxml-ng或者iqtree建基因树，1000 bootstraps

# 物种树
## 准备输入文件
1. genes.tre
合并所有基因的最佳树(带支持率)到一个文件，每棵基因树一行。
从raxml-ng的结果获取：`ls *raxml.support > genes.tre`

Astral的开发者的建议：把genes.tre树中支持率低的分支坍塌，再运行Astral结果会更精确。

有文章里是测试了不坍塌，坍塌<10，20，30，40，50的基因树分别运行astral，然后发现没有区别，最后用的<50的结果。
我的建议：有时间就测试下不坍塌，坍塌<10，30的结果。没时间就直接跑坍塌<10的(也是开发者建议的)。

`conda install -c bioconda newick_utils`
用Newick utils生成坍塌低支持率的树。

2. bootstraps.path

从raxml-ng的结果获取：`ls *raxml.bootstraps > bootstraps.path`


## 建astral物种树
nohup java -jar -Xmx100G /path/to/astral.5.7.8.jar -i genes.tre -o species.tre -r 1000 2> astral.log &
-Xmx100G 指定内存大小，基因数量多时可指定高一些
- -i 输入合并的基因树文件
- -o 物种树的输出文件，使用了-b和-r参数就会有-r设定的1000棵。
- -r 1000：bootstrap自展检验的重复次数
- 2>astral.log：保存日志文件
- --outgroup Outgroup：外类群设定，astral得到的是无根的物种树，所以这个外类群设置只是用于结果展示，在计算分析中不被考虑进去。

`java -Xmx100G -Djava.library.path=/path/to/ASTRAL/lib/ -jar /path/to/astral.5.15.5.jar -i genes_bs10.tre -o species_bs10.trees -T 16 -C -b bootstraps.file -r 1000 > astral_bs10.log 2>&1 &`

输入文件是含有所有genetrees的Newick格式的文件。输入的genetree被当做unrooted tree,不管他们是否有根。输入的genetree支持多分支。
输出文件也是Newick格式，也是被当做unrooted tree。

astral测量branch length 是用coalscent units。不是我们通常认为的boostrap value

1.astral 只测量internal branches 不测量terminal branches
2.branch lengthes 是直接测量genetree的不一致性，有可能因为统计噪音被低估。
3.support value 测量quadriparition的support而不是通常的bipartion（二分位）。（astral 认为自己的support value 比boostrap value 可信度更高。）

### 输出文件
1. species.trees
分析结果，物种树文件。

如果没使用-b参数，species.trees只包含一棵物种树，即为最终结果。

如果使用-b参数进行bootstrap运算，species.trees则包含所有运算的结果和最终的物种树。
species.trees文件前面1000行是bootstrap运算结果，支持率范围是[0-1]。
species.trees文件最后两行的物种树支持率是[0-100]，是最终的物种树。倒数第二行是只包含支持率的格式，倒数第一行是包含支持率和枝长的格式。

关于-b和-r参数：ASTRAL的开发者认为不用bootstrap(-b参数)的默认算法得到的支持率更准确，但还是提供了-b参数用于bootstrap运算。
当使用-b参数时，-o输出的物种树会有-r设定的数量(这里设置的1000棵)。这时，最终的物种树就要到astral.log的文本末寻找。

2. astral.log
运行log文件。

如果使用-b参数，里面包含每次bootstrap(replicate 0-999)运行的日志，包含每次bootstrap的树结果，支持率范围是[0-1]。
在bootstrap运行日志后，还有一次Running the main analysis的结果，支持率范围也是[0-1]。(ps:我查看了结果，发现这棵树的拓扑结构与species.tree最后的物种树的拓扑是一致的，枝长也是一致的，支持率不一致。)



#### astral
先用其他建树软件，比如raxml对每个基因单独建树，再合并所有的基因树（newick格式），合并每个基因树的RAxML_bipartitions.ml成一个genes.tre文件

###




##### 为物种树打分
java -jar -Xmx100G /path/to/astral.5.7.4.jar -q species.tre -i genes.tre -o scored.tre 2> scored.log

与基因树genes.tre比较后，为物种树species.tre打分。

quartet score 和branch length 和 branch support values。0.9表示genetree产生的quartet tree的90%存在于species tree中。

结果如下：
Quartet score is : 4803
Normalized quartet socre is: 0.4798201798201798

代表基因树中有4803个quartet trees存在物种树中，占所有quartet trees的47.98%，意味着genetree和species tree的不一致程度很高。


##### 扩展内容 —— 分支注释参数


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>