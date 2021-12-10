---
title: 绘制进化树
date: 2021-11-22 19:50:00
categories: 
- bio
- bioinfo
tags:
- tutorial
- biosoft
- phylogeny
- evolutionary tree
description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# 进化树的绘制
进化树的绘制有许多软件可以使用。

## 在线绘制进化树
[itol](https://itol.embl.de/)
[EvolView](https://www.evolgenius.info/evolview/#login)

## ggtree【推荐】
R包ggtree绘制进化树
treeio包用于解析各种格式的进化树
ggtree用于绘制进化树


treeio用于解析各种格式的进化树文件，ggtree用于绘制进化树和注释

library(treeio)
library(ggtree)
mtree <- read.newick("mel.tre")
ggtree(mtree) + geom_tiplab(color="black",size = 4,hjust = -0.2)+geom_point(color="#6FE1F8", size=5, alpha=0.7) + geom_text(aes(label=node),size=3) + geom_highlight(node=15,fill="firebrick",alpha=0.6)


mc

treetext<-"((((Mcandidum:0.418672[&&NHX:I=+5728:D=-1170:G=Malvids],Egrandis:0.201388[&&NHX:I=+1420:D=-2519:G=Malvids])[&&NHX:C=1]:0.148853,((Ptrichocarpa:0.219587[&&NHX:I=+5355:D=-1003:G=Fabids],(Csinensis:0.209405[&&NHX:I=+1026:D=-3516:G=Malvids],(Graimondii:0.224188[&&NHX:I=+4875:D=-998:G=Malvids],Athaliana:0.492461[&&NHX:I=+1776:D=-2891:G=Malvids]):0.027774[&&NHX:C=2]):0.012564[&&NHX:C=3]):0.019343[&&NHX:C=4],(Ppersica:0.194515[&&NHX:I=+1159:D=-2949:G=Fabids],(Mtruncatula:0.354580[&&NHX:I=+3310:D=-1984:G=Fabids],Csativus:0.338644[&&NHX:I=+794:D=-3735:G=Fabids]):0.022063[&&NHX:C=5]):0.019111[&&NHX:C=6]):0.011710[&&NHX:C=7]):0.029914[&&NHX:C=8],Vvinifera:0.169100[&&NHX:I=+1522:D=-3045:G=Vitales]):0.241611[&&NHX:C=9],Mguttatus:0.241611[&&NHX:I=+2278:D=-1909:G=Asterids])[&&NHX:C=10];"
mtree <- read.nhx(textConnection(treetext))
p1<-ggtree(mtree,size=0.8) + geom_tiplab(color="black",size = 4.5,hjust = -0.1, align=TRUE, linesize=0.7)+geom_nodepoint(color="#DCDCDC", size=6, alpha=0.9)+geom_nodelab(aes(label=C), size=3.5, hjust = 1)+ geom_label2(aes(x=0.79,label=D),color="black",vjust=-0.2,fill="#7FFF00",alpha=0.6)+geom_label2(aes(x=0.73,label=I),color="black",vjust=-0.2,fill="#FF7D40",alpha=0.6)+xlim(NA,1.1)
p2<- p1 %>% rotate(node=14) %>% rotate(node=16)
p2+geom_strip(10,10,label="Vitales",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(3,7,label="Fabids",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(1,4,label="Fabids",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)
> p2+geom_strip(10,10,label="Vitales",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(3,7,label="Fabids",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(1,4,label="Fabids",offset = 0.14, offset.text =0.01,color = "#802A2A",barsize = 1.5,fontsize = 5)
> p2+geom_strip(10,10,label="Vitales",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(3,7,label="Fabids",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(1,4,label="Malvids",offset = 0.14, offset.text =0.01,color = "#802A2A",barsize = 1.5,fontsize = 5)



bv

treetext<-"(((Copaifera:0.119877[&&NHX:I=+2147:D=-4477:G=Detarioideae],((Xanthocercis:0.052846[&&NHX:I=+3135:D=-2353:G=Papilionoideae],((Lotus:0.079773[&&NHX:I=+731:D=-2017:G=Papilionoideae],Medicago:0.128039[&&NHX:I=+1544:D=-1595:G=Papilionoideae])100:0.016841[&&NHX:C=1],Glycine:0.087362[&&NHX:I=+7766:D=-420:G=Papilionoideae])100:0.039763[&&NHX:C=2])100:0.028086[&&NHX:C=3],(Gleditsia:0.052219[&&NHX:I=+2091:D=-3240:G=Caesalpinioideae],(Chamaecrista:0.081531[&&NHX:I=+1481:D=-3051:G=Caesalpinioideae],Acacia:0.094288[&&NHX:I=+2417:D=-2086:G=Caesalpinioideae])100:0.018064[&&NHX:C=4])100:0.024205[&&NHX:C=5])100:0.009058[&&NHX:C=6])94:0.005981[&&NHX:C=7],(Bauhinia:0.047455[&&NHX:I=+4523:D=-345:G=Cercidoideae],Cercis:0.016091[&&NHX:I=+530:D=-3202:G=Cercidoideae])100:0.051650[&&NHX:C=8]):0.079668[&&NHX:C=9],Quillaja:0.079668[&&NHX:I=+3712:D=-1416:G=Quillajaveae]):0.0[&&NHX:C=10];"
btree<- read.nhx(textConnection(treetext))

treetext<-"(((C_officianalis:0.119877[&&NHX:I=2147:D=4477:G=Detarioideae],((X_zambesiaca:0.052846[&&NHX:I=3135:D=2353:G=Papilionoideae],((L_japonicus:0.079773[&&NHX:I=731:D=2017:G=Papilionoideae],M_truncatula:0.128039[&&NHX:I=1544:D=1595:G=Papilionoideae])100:0.016841[&&NHX:C=1],G_max:0.087362[&&NHX:I=7766:D=420:G=Papilionoideae])100:0.039763[&&NHX:C=2])100:0.028086[&&NHX:C=3],(G_triacanthos:0.052219[&&NHX:I=2091:D=3240:G=Caesalpinioideae],(C_fasciculata:0.081531[&&NHX:I=1481:D=3051:G=Caesalpinioideae],A_pycnantha:0.094288[&&NHX:I=2417:D=2086:G=Caesalpinioideae])100:0.018064[&&NHX:C=4])100:0.024205[&&NHX:C=5])100:0.009058[&&NHX:C=6])94:0.005981[&&NHX:C=7],(B_variegata:0.047455[&&NHX:I=4523:D=345:G=Cercidoideae],C_canadensis:0.016091[&&NHX:I=530:D=3202:G=Cercidoideae])100:0.051650[&&NHX:C=8]):0.079668[&&NHX:C=9],Q_saponaria:0.079668[&&NHX:I=3712:D=1416:G=Quillajaveae]):0.0[&&NHX:C=10];"

treetext<-"(((Copaifera:0.119877[&&NHX:I=2147:D=4477:G=Detarioideae],((Xanthocercis:0.052846[&&NHX:I=3135:D=2353:G=Papilionoideae],((Lotus:0.079773[&&NHX:I=731:D=2017:G=Papilionoideae],Medicago:0.128039[&&NHX:I=1544:D=1595:G=Papilionoideae])100:0.016841[&&NHX:C=1],Glycine:0.087362[&&NHX:I=7766:D=420:G=Papilionoideae])100:0.039763[&&NHX:C=2])100:0.028086[&&NHX:C=3],(Gleditsia:0.052219[&&NHX:I=2091:D=3240:G=Caesalpinioideae],(Chamaecrista:0.081531[&&NHX:I=1481:D=3051:G=Caesalpinioideae],Acacia:0.094288[&&NHX:I=2417:D=2086:G=Caesalpinioideae])100:0.018064[&&NHX:C=4])100:0.024205[&&NHX:C=5])100:0.009058[&&NHX:C=6])94:0.005981[&&NHX:C=7],(Bauhinia:0.047455[&&NHX:I=4523:D=345:G=Cercidoideae],Cercis:0.016091[&&NHX:I=530:D=3202:G=Cercidoideae])100:0.051650[&&NHX:C=8]):0.079668[&&NHX:C=9],Quillaja:0.079668[&&NHX:I=3712:D=1416:G=Quillajaveae]):0.0[&&NHX:C=10];"

ggtree(btree,size=0.8)+geom_tiplab(color="black",size = 4.5,hjust = -0.1, align=TRUE, linesize=0.7)+geom_nodepoint(color="#DCDCDC", size=6, alpha=0.9)+geom_nodelab(aes(label=C), size=3.5, hjust = 1)+geom_label2(aes(x=0.29,label=D),color="black",vjust=-0.2,fill="#7FFF00",alpha=0.6)+geom_label2(aes(x=0.265,label=I),color="black",vjust=-0.2,fill="#FF7D40",alpha=0.6)+geom_strip(9,10,label="Cercidoideae",offset = 0.06,offset.text = 0.005,color = "#802A2A",barsize = 1.5,fontsize = 5)+geom_strip(1,1,label="Detarioideae",offset = 0.06,offset.text = 0.005,color = "#ED9121",barsize = 1.5,fontsize = 5)+geom_strip(2,4,label="Papilionoideae",offset = 0.06,offset.text = 0.005,color = "#FF6100",barsize = 1.5,fontsize = 5)+geom_strip(6,8,label="Caesalpinioideae",offset = 0.06, offset.text =0.005,color = "#D2691E",barsize = 1.5,fontsize = 5)+xlim(NA,0.45)



ggtree(mtree, size=0.8, branch.length='none', mrsd="2021-10-01") \ #画树，size指定枝的粗细，branch.length指定枝长，mrsd指定采样日期用于画时间尺度轴
+ geom_tiplab(color="black", size = 4, hjust = -0.1, align=T, linesize=0.7) \ # 末端节点标上物种，hjust是离进化枝线条的距离,align=T对齐，linesize设置虚线尺寸。
+ geom_point(color="#6FE1F8", size=5, alpha=0.7) \ #节点标记上点，配色
+ geom_text(aes(label=node),size=3) \ #节点标记上数字，尺寸
+ geom_highlight(node=15,fill="firebrick",alpha=0.6) #高亮指定分支（节点node所在分支）
+ geom_label2(aes(x=0.79,label=D),color="black",vjust=-0.2,fill="#7FFF00",alpha=0.6) #做标记,aes里的x指定位置，label指定内容
+ geom_strip(9,10,label="Cercidoideae",offset = 0.06,offset.text = 0.012,color = "#FF6100",barsize = 1.5,fontsize = 5,angle = 90,hjust = 0.5) #显示条带和文字，用于指定clade
+ geom_range("length_0.95_HPD", color='red', size=2, alpha=.5) #在节点上画时间的95%置信范围
+ xlim(NA,1) #如果内容超出边界，就拓展显示


画成有枝长的树，在ggtree里指定branch.length参数。
画时间标尺，在ggtree里指定mrsd参数为最近的采样日期，用于画时间尺度轴。




ref
https://yulab-smu.top/treedata-book/index.html