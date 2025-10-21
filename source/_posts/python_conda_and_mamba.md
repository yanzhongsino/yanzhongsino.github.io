---
title: python环境管理工具（anaconda，miniconda，miniforge，mambaforge）和python包管理工具（conda，mamba，pip）
date: 2025-10-20
categories: 
- programming
- python
- conda
tags:
- python
- conda
- mamba
- pip
- anaconda
- miniconda
- miniforge
- mambaforge

description: 介绍python环境管理工具（anaconda，miniconda，miniforge，mambaforge）和python包管理工具（conda，mamba，pip）。推荐安装miniforge，并使用mamba来安装软件包。
---

<div align="middle"><iframe data-testid="embed-iframe" style="border-radius:12px" src="https://open.spotify.com/embed/track/0Gj0Mvge3iiI1gnB4fJl0q?utm_source=generator" width="50%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe></div>

# 1. python环境管理工具和python包管理工具
1. 环境管理工具
- 管理多个编程环境的工具。在一个环境下，可以定义一套完整的工具。
- 例如 anaconda，miniconda，miniforge，mambaforge 等
2. 包管理工具
- 用来管理和安装软件包
- 例如 conda，mamba，pip 等
3. 推荐
- 在这些python环境和python包管理软件中，目前最推荐安装miniforge，然后用mamba来管理python环境和安装python包。

# 2. python环境管理工具：Anaconda, Miniconda, Miniforge, Mambaforge
## 2.1. Anaconda 和 Miniconda vs Miniforge 和 Mambaforge
这些环境管理工具的核心都是通过conda这一工具，来实现 python 环境(environment) 和 包 (package) 管理的，（除了python，很多语言R, Java, C都支持）。

1. 付费/免费 vs 开源
- Anaconda 和 Miniconda 是Anaconda公司的产品（2012年起开发），商用版本付费，个人版本免费。
- Miniforge 和 Mambaforge 是由conda-forge社区维护和更新的，用GitHub托管，开源免费。
2. 支持mamba
- Anaconda 和 Miniconda 默认不支持 mamba，需要手动安装。
- Miniforge 和 Mambaforge 默认支持 mamba。
3. 默认channels
- Anaconda 和 Miniconda 用anaconda.org作为默认channel。
- Miniforge 和 Mambaforge 用conda-forge作为默认channel。
- conda channels是下载packages的来源。
- conda-forge由于齐全和快速更新被作为主流的channel，所以即使用 Anaconda 和 Miniconda 也建议设置 conda-forge 为优先的channel。

## 2.2. Anaconda 和 Miniconda 
1. Anaconda 包含 python 和 conda 以及另外上千多个常用包，以及图形界面 Spyder 等。
2. Miniconda 是 Anaconda 的最小精简版本，只包含了python和conda，只包含了最小的依赖，没有预装任何额外库。安装速度快很多。

## 2.3. Miniforge 和 Mambaforge
1. Miniforge 与 Miniconda 类似，但不属于Anaconda公司，而是开源的，并且支持 mamba 。
2. Mambaforge 改自 Miniforge，做了一些优化。自2023年8月 Miniforge 23.3.1 版本发布后，Miniforge与Mambaforge在功能上已基本完全一致。两者唯一的区别仅在于安装程序名称及由此决定的默认安装路径。

# 3. python包管理工具：conda, mamba, pip
## 3.1. conda, mamba, pip
1. conda
- conda 是通用的python包管理工具，并可以管理C/C++库和R包等依赖
- conda 由 Anaconda 公司维护
2. mamba
- mamba 是开发来替代conda的高速工具
- mamba 由 conda-forge社区开源更新和维护
- mamba 支持多线程，对依赖的解析也更智能，比 conda 安装速度快很多。
3. pip
- pip 是python官方标准的python包管理器，只能安装python包
- pip 由python官方（PyPA）更新维护
- pip 用于从PyPI安装和管理Python标准库之外的其他第三方包。
## 3.2. mamba 优势
2019年 Wolf Vollprecht 开发了mamba，用来解决conda安装和编译速度慢的问题，显著提高了安装包的速度，推荐用mamba替代conda。
1. mamba 的特点
- mamba 核心通过 C++ 实现，执行效率高
- mamba 支持多线程下载和异步下载 repository data 和 packages
- mamba 使用 libsolv加速解决依赖关系
- mamba 与 conda 完美兼容
2. mamba 的限制
- 用mamba install替代conda install，用mamba create替代conda create，
- 但是，conda activate, conda deactivate, conda config等环境和channels管理命令无法用mamba替代。

# 4. 推荐安装和使用
在这些python环境和python包管理软件中，目前最推荐安装miniforge，然后用mamba来管理python环境和安装python包。

## 4.1. 安装Miniforge
1. 下载Miniforge安装文件：在 https://conda-forge.org/miniforge/ 选择对应系统的安装文件
2. 命令`bash Miniforge3-Linux-x86_64.sh`开始安装
- 跟随引导语，两次ENTER和两次yes即可。
- press ENTER to continue: ENTER
- Do you accept the license terms? [yes|no]: yes
- Miniforge3 will now be installed into this location: /home/user_name/miniforge3: ENTER
- Do you wish to update your shell profile to automatically initialize conda? : yes
3. 安装成功，会改变以下文件
- 修改/home/user_name/.bashrc文件，加入conda初始化信息
- 生成/home/user_name/miniforge3目录，miniforge3安装在此目录下
- conda和mamba命令在这个目录下，比如/home/user_name/miniforge3/bin/mamba
4. 安装后需要关闭shell并重开新的shell才能应用
- For changes to take effect, close and re-open your current shell
5. 推荐这样添加channels
- `conda config --show channels` 查看channels
- `conda config --add channels bioconda` 添加bioconda到channels内
- `conda config --add channels conda-forge` 把conda-forge放在top，作为最优先的channel
6. 安装软件包
- `mamba install biopython`
- `mamba install bioconda::blast`

## 4.2. 已安装conda的情况下，安装mamba
- 如果已用anaconda或miniconda安装conda，也可以额外安装mamba
- `conda install mamba -n base -c conda-forge`

References
1. Miniforge github：https://github.com/conda-forge/miniforge

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>