library(treeio)
library(ggtree)

# 画枝长树
species13<-"(M_guttatus:0.183432938,(V_vinifera:0.1306345245,((E_grandis:0.1543952959,(O_opipara:0.0394453858,(M_dodecandrum:0.0135249677,M_candidum:0.0050871044):0.0104449535[100]):0.3062201747[100]):0.1157257592[100],((P_persica:0.1546035429,(M_truncatula:0.2694497818,C_sativus:0.2631067785):0.0184720066[100]):0.0154755336[100],(P_trichocarpa:0.1742225982,(C_sinensis:0.1609068419,(G_raimondii:0.1721238316,A_thaliana:0.3808096307):0.0228427456[100]):0.0105212583[100]):0.0166016265[100]):0.0087138974[99.5]):0.0249623121[100])OROOT:0.183432938);"

tretext<-"(Mguttatus:0.20034956165,(((Egrandis:0.1652493677,(Mcandidum:0.0043291819,Mdodecandrum:0.0098752548):0.3374077423[100]):0.1292005385[100],((Ppersica:0.1648917562,(Csativus:0.2821749094,Mtruncatula:0.2947558221):0.0198712385[100]):0.0163752970[100],((Csinensis:0.1736334072,(Graimondii:0.1868267915,Athaliana:0.4111953713):0.0234125462[100]):0.0105952970[100],Ptrichocarpa:0.1847798524):0.0162970775[100]):0.0099557079[100]):0.0259194505[100],Vvinifera:0.1411632583)OROOT:0.20034956165);"
tree<-read.nhx(textConnection(tretext))
ggtree(tree,size=0.8)+geom_tiplab(color="black",size=5,align = TRUE,linesize=0.7)+geom_nodepoint(size=2,color="firebrick")+geom_rootedge(rootedge = 0.02,size=1)+geom_text2(aes(subset=(node=!isTip),label=branch.length),hjust=-0.1, size=4)+geom_text2(aes(subset=(node=isTip),label=branch.length), color="darkblue",hjust=1, vjust=-0.5, size=4, fontface = "bold")+xlim(-0.02,0.8)
scaleClade(p, node=13, scale=0.3)
在Rstudio保存从6*8inches的pdf文件

# 画时间树
time tree file:
12 species
(((((Mcandidum:0.033388[&&NHX:I=3027:D=467:G=Malvids],Mdodecandrum:0.033388[&&NHX:I=1470:D=1735:G=Malvids]):0.704677,Egrandis:0.738065[&&NHX:I=1420:D=2519:G=Malvids]):0.402123,((Ptrichocarpa:1.026206[&&NHX:I=5421:D=927:G=Malvids],(Csinensis:0.968507[&&NHX:I=982:D=3295:G=Malvids],(Graimondii:0.844889[&&NHX:I=4747:D=691:G=Malvids],Athaliana:0.844889[&&NHX:I=1412:D=2569:G=Malvids]):0.123618):0.057700):0.072031,(Ppersica:1.023953[&&NHX:I=1185:D=2510:G=Fabids],(Mtruncatula:0.916328[&&NHX:I=3117:D=1747:G=Fabids],Csativus:0.916328[&&NHX:I=739:D=3326:G=Fabids]):0.107625):0.074285):0.041950):0.082575,Vvinifera:1.222763[&&NHX:I=1537:D=2881:G=Vitales]):0.035026,Mguttatus:1.257789[&&NHX:I=2330:D=1708:G=Asterids]);

13 species
(M_guttatus: 125.8730, (V_vinifera: 122.3928, ((E_grandis: 76.0013, (O_opipara: 12.1731, (M_dodecandrum: 4.6480, M_candidum: 4.6480) : 7.5251) : 63.8281) : 38.1608, ((P_persica: 102.3591, (M_truncatula: 91.1542, C_sativus: 91.1542) : 11.2049) : 7.7799, (P_trichocarpa: 102.5456, (C_sinensis: 96.0341, (G_raimondii: 82.6460, A_thaliana: 82.6460) : 13.3881) : 6.5116) : 7.5933) : 4.0231) : 8.2308) : 3.4801) ;


timetree<-"(((((M_candidum:3.3388[&&NHX:I=3027:G=Malvids],M_dodecandrum:3.3388[&&NHX:I=1470:D=1735:G=Malvids]):70.4677[&&NHX:I=3476:D=1021],E_grandis:73.8065[&&NHX:I=1420:D=2519:G=Malvids]):40.2123[&&NHX:I=971:D=823],((P_trichocarpa:102.6206[&&NHX:I=5421:D=927:G=Malvids],(C_sinensis:96.8507[&&NHX:I=982:D=3295:G=Malvids],(G_raimondii:84.4889[&&NHX:I=4747:D=691:G=Malvids],A_thaliana:84.4889[&&NHX:I=1412:D=2569:G=Malvids]):12.3618[&&NHX:I=705:D=261]):5.7700[&&NHX:I=43:D=163]):7.2031[&&NHX:I=313:D=172],(P_persica:102.3953[&&NHX:I=1185:D=2510:G=Fabids],(M_truncatula:91.6328[&&NHX:I=3117:D=1747:G=Fabids],C_sativus:91.6328[&&NHX:I=739:D=3326:G=Fabids]):10.7625[&&NHX:I=181:D=66]):7.4285[&&NHX:I=60:D=396]):4.1950[&&NHX:I=49:D=151]):8.2575[&&NHX:I=102:D=82],V_vinifera:122.2763[&&NHX:I=1537:D=2881:G=Vitales]):3.5026[&&NHX:I=45:D=1],M_guttatus:125.7789[&&NHX:I=2330:D=1708:G=Asterids]);"

treetext<-"(((Egrandis:0.1652493677[&&NHX:I=1420:D=2519:G=Malvids],(Mcandidum:0.0043291819[&&NHX:I=3027:D=467:G=Malvids],Mdodecandrum:0.0098752548[&&NHX:I=1470:D=1735:G=Malvids])100:0.3374077423)100:0.1292005385,((Ppersica:0.1648917562[&&NHX:I=1185:D=2510:G=Fabids],(Csativus:0.2821749094[&&NHX:I=739:D=3326:G=Fabids],Mtruncatula:0.2947558221[&&NHX:I=3117:D=1747:G=Fabids])100:0.0198712385)100:0.0163752970,((Csinensis:0.1736334072[&&NHX:I=982:D=3295:G=Malvids],(Graimondii:0.1868267915[&&NHX:I=4747:D=691:G=Malvids],Athaliana:0.4111953713[&&NHX:I=1412:D=2569:G=Malvids])100:0.0234125462)100:0.0105952970,Ptrichocarpa:0.1847798524[&&NHX:I=5421:D=927:G=Malvids])100:0.0162970775)100:0.0099557079)100:0.0259194505,Vvinifera:0.1411632583[&&NHX:I=1537:D=2881:G=Vitales],Mguttatus:0.4006991233[&&NHX:I=2330:D=1708:G=Asterids]);"



tree<-read.nhx(textConnection(timetext))
p1<-ggtree(tree,size=0.8)+geom_tiplab(color="black",size=4.5,align = TRUE,linesize=0.7)+geom_label2(aes(x=120,label=D),color="black",vjust=-0.2,fill="springgreen",alpha=0.6)+geom_label2(aes(x=108,label=I),color="black",vjust=-0.2,fill="tomato",alpha=0.6)+geom_nodepoint(size=2,color="firebrick")+geom_rootedge(rootedge = 5,size=1)+theme_tree2()+scale_x_continuous(limits=c(-5,150),breaks = seq(0,125,25),labels = c(125,100,75,50,25,0))+geom_text2(x=-2.5,y=2.6,label="C1",color="black",fontface = "bold")+geom_text2(x=20,y=8.5,label="C2",color="black",fontface = "bold")
p2<-ggtree::rotate(p1,18)
scaleClade(p2, node=13, scale=0.4)+geom_text2(x=20,y=4.9,label="C2",color="black",fontface = "bold")
保存成6*7inches的pdf文件


p1<-ggtree(tree,size=0.8)+geom_tiplab(color="black",size=4.5,hjust = -0.1,align = TRUE,linesize=0.7)+geom_label2(aes(x=120,label=D),color="black",vjust=-0.2,fill="springgreen",alpha=0.6)+geom_label2(aes(x=110,label=I),color="black",vjust=-0.2,fill="tomato",alpha=0.6)+geom_nodepoint(size=2,color="firebrick")+geom_rootedge(rootedge = 5,size=1)+theme_tree2()+scale_x_continuous(limits=c(-5,165),breaks = seq(0,125,25),labels = c(125,100,75,50,25,0))+geom_text2(x=-2.5,y=2.8,label="C1",color="black",fontface = "bold")+geom_text2(x=20,y=8.5,label="C2",color="black",fontface = "bold")
p2<-ggtree::rotate(p1,18)
p2+geom_strip(11,11,label="Vitales",offset = 28, offset.text =2,color = "grey31",barsize = 1.5,fontsize = 5)+geom_strip(12,12,label="Asterids",offset = 28, offset.text =2,color = "grey31",barsize = 1.5,fontsize = 5)+geom_strip(4,8,label="Fabids",offset = 28, offset.text =2,color = "grey31",barsize = 1.5,fontsize = 5)+geom_strip(3,5,label="Malvids",offset = 28, offset.text =2,color = "grey31",barsize = 1.5,fontsize = 5)

保存6*9inches的pdf文件


# venn图
setwd("D:/SYSU/SYSULab/project/202108_Melastoma.candidum/2.comparativegenome/plot/")

画venn图
library(VennDiagram)
library(grid)
data <- read.table("orthogroups.venn", header = T, sep = "\t")
attach(data)
grid.newpage()
venn.quad.plot <- venn.diagram(x = list(Mcandidum=Mcandidum, Vvinifera=Vvinifera, Egrandis=Egrandis, Mtruncatula=Mtruncatula), #等号前面是图上显示的数据名称，等号后面是数据源，数据顺序与图上顺序一致。
filename = NULL,
col = "black",
lty = "dotted", #边框线类型
lwd = 4,
fill = c("cornflowerblue", "green", "orange", "darkorchid1"),
alpha = 0.60,
label.col = c("black", "white", "black", "white", "white", "white",
                 "white", "white", "black", "white",
                  "white", "white", "white", "black", "white"),
    cex = 2.5,
    fontfamily = "serif",
    fontface = "bold",
    cat.col = c("darkblue", "darkgreen", "orange", "darkorchid4"),
    cat.cex = 2.5, #分类名称字体大小
    cat.fontfamily = "serif"
);
grid.draw(venn.quad.plot);

保存成6*7 inches 的pdf格式； 600(height)*700(width)的png格式
保存的pdf是可编辑的，所以如果需要调整文字大小/字体/位置，都可以在pdf上编辑。



# 画upset图
mutations <- read.csv("orthogroups.upset", header=TRUE, sep = "\t")

upset(mutations,sets = c("Mcandidum","Mdodecandrum","Athaliana","Egrandis","Ptrichocarpa","Mtruncatula","Vvinifera"),nintersects = 10,order.by = c("degree","freq"), decreasing = c(TRUE,TRUE), point.size = 3, line.size = 1, mainbar.y.label = "Intersection size of gene family", sets.x.label = "genome size", text.scale = c(2, 2, 1.6, 1.5, 1.8, 1.8),queries = list(list(query = intersects,params = list("Mcandidum", "Mdodecandrum"),color="red",active = T)))

保存成6*7 inches 的pdf格式； 600(height)*700(width)的png格式