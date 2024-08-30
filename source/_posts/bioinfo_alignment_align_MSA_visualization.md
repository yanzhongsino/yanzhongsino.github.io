---
title: 多序列比对的整体可视化
date: 2024-08-02
categories: 
- bioinfo
- alignment
- align
- MSA
tags: 
- alignment
- Multiple Sequence Alignment
- MSA
- visualization
- AliView
- msaviewer
description: 多序列比对的可视化，黑白、彩色的可视化，包括用AliView进行多序列比对的整体可视化。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1306436987&auto=1&height=32"></iframe></div>

**一句话总结：推荐使用AliView进行多序列比对的整体可视化。**

# 1. 多序列比对的可视化
通过肉眼查看多序列比对结果，可以用MEGA, Bioedit, Genious, AliView等软件。
- MEGA, Bioedit可以查看局部比对结果，并对碱基/氨基酸渲染颜色，但无法从整体上查看比对结果，即无法对比对结果进行缩放（Zoom out/in）。
- Genious可以对比对结果缩放（Zoom out/in），从而从整体上查看比对结果。但是它是商业软件，费用较高。可以用邮箱注册试用。
- AliView是免费开源的软件，支持对比对结果缩放（Zoom out/in），从整体上查看比对结果，推荐使用。

# 2. AliView
- AliView是Uppsala University团队开发的一个多序列比对可视化工具：https://ormbunkar.se/aliview/。
- 免费开源，在github上可获取：https://github.com/AliView/AliView。
- 支持FASTA,PHYLIP,NEXUS,CLUSTAL,MSF格式的输入文件。

需要查看多序列比对的整体效果，只需在AliView上打开多序列比对文件，然后File-Export alignments as image即可导出图，注意导出的图不包含序列ID。

<img src="https://www.ormbunkar.se/aliview/images/screenshot_mac_400v2.png" width=50% title="多序列比对整体图" align=center/>

**<p align="center">Figure 1. 多序列比对整体图</p>**

# 3. msaviewer
另外，NCBI有个在线工具msaviewer(NCBI Multiple Sequence Alignment Viewer)，支持对多序列比对结果进行缩放（Zoom out/in），查看整体比对。

https://www.ncbi.nlm.nih.gov/projects/msaviewer/


# 4. references
1. https://ormbunkar.se/aliview/
2. https://github.com/AliView/AliView3
3. https://www.ncbi.nlm.nih.gov/projects/msaviewer/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>