---
title: 参考文献管理软件EndNote的中英文混排技巧
date: 2023-05-11
categories: 
- tools
- academia
- EndNote
tags: 
- tools
- EndNote
- 中英文混排

description: 介绍参考文献管理软件EndNote的中英文混排技巧。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=399341112&auto=1&height=32"></iframe></div>

# 1. EndNote的中英文混排
## 1.1. 添加EndNote的【中文文献】类型
- 在EndNote中选择【Edit】-【References】，找到【Reference Types】选项；
- 【Default Reference Type】选择【unused】1/2/3中的任意一个之前未被修改过的，然后选择【Modify Reference Types】；
- 在弹出的【Modify Reference Types】对话框中按照下面的图片输入每项内容。包括【Generic】命名为【中文文献】，【Author】填入【Author】，【Year】填入【Year】，【Title】填入【Title】，【Secondary Author】填入【Secondary Author】，【Secondary Title】要填入【Journal】，【Volume】填入【Volume】，【Number of Volumes】填入【Issue】，【Pages】填入【Pages】，其他都一致，点击【OK】后就添加了【中文文献】的参考文献类型。

<img src="https://pic3.zhimg.com/v2-b2557ffb99bf0743123d02ca539185ca_r.jpg" title="Modify Reference Types" width="50%"/>

**<p align="center">图 1. Modify Reference Types 对话框 图源：https://zhuanlan.zhihu.com/p/312093759</p>**

## 1.2. EndNote的文献目录(Bibliography)的中英文混排
### 1.2.1. 文献目录(Bibliography)的中英文顺序
- 在EndNote中选择【Edit】-【Output Styles】-【Edit **】，选择需要编辑的参考文献Styles；
- 在【Bibliography】- 【Sort Order】中选择文献目录(Bibliography)的显示排序规则；
- 需要英文在前，中文在后的，则选择【Other...】，然后依次选择排序标准，推荐选择【Language】+【Author】+【Title】，然后保存设置。
- 注意每一条参考文献的【Language】字段，中文可以设置成【Chinese】，英文设置成【English】或不设置。

### 1.2.2. 文献目录(Bibliography)的作者的**et al.** 和**等.**混排
- 基于已添加【中文文献】类型下操作；
- 在EndNote中选择【Edit】-【Output Styles】-【Edit **】，选择需要编辑的参考文献Styles；
- 在【Bibliography】-【Templates】的【Reference Type】中添加【中文文献】的类型；

<img src="https://pic1.zhimg.com/v2-0ac5695cfe8180233993929c00c15bec_r.jpg" title="在Templates中添加中文文献类型" width="50%"/>

**<p align="center">图 2. 在Templates中添加中文文献类型 图源：https://zhuanlan.zhihu.com/p/312093759</p>**

- 将【Journal Article】里的参考格式复制到【中文文献】里，再在Author前面加上Secondary，注意Secondary和Author之间打一个空格，显示为一个点.。

<img src="https://pic4.zhimg.com/v2-2415ffab7677257fc3e16474c89a6c27_r.jpg" title="在Templates中添加中文文献类型2" width="50%"/>

**<p align="center">图 3. 在Templates中添加中文文献类型2 图源：https://zhuanlan.zhihu.com/p/312093759</p>** 

- 在【Bibliography】-【Author List】中，对于显示作者人数进行设置，多作者的缩写默认为【，et al.】，这里代表英文文献多作者在文献目录(Bibliography)中的显示规则；
- 在【Bibliography】-【Editor List】中，对于显示作者人数进行设置，多作者的缩写改为【，等.】，这里代表中文文献多作者在文献目录(Bibliography)中的显示规则；

<img src="https://pic2.zhimg.com/v2-cb026088966e21676868136d33f7e051_r.jpg" title="修改中文文献的多作者显示" width="50%"/>

**<p align="center">图 4. 修改中文文献的多作者显示 图源：https://zhuanlan.zhihu.com/p/312093759</p>** 

- EndNote导入的中文文献可选择【中文文献】这种类型，然后把【Author】这项内容复制到【Secondary Author】，这样就会遵循【中文文献】的引用的显示规则。

ref：https://zhuanlan.zhihu.com/p/312093759

## 1.3. EndNote的正文引用的中英文混排
### 1.3.1. 作者的**et al.** 和**等.**混排
1. 手动修改中文文献的**et al.**成**等.**
- 在Word正文的EndNote引用文献(张三 et al. 2023)中右键，选择【edit citations】-【more】；
- 然后在【Edit & Manage Citations】对话框中把这一条参考文献的【Formatting】从【Default】改成【Exclude Author】，这样正文会不显示作者；
- 继续在【Prefix】中手动添加前缀，这里填写【张三等，】，则正文会显示(张三等，2023);
- 依次修改正文中每一条参考文献。

ref：https://www.baishujun.com/archives/7634.html

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>
