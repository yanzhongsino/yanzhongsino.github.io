---
title: R及相关软件的安装
date: 2021-12-28 16:30:00
categories: 
- computer language
- R
tags:
- R
- RStudio
- Rtools
- R package
- CRAN
- Bioconductor

description: 记录了R及相关软件RStudio，Rtools的安装，R包的安装方法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=27853350&auto=1&height=32"></iframe></div>

# 1. R
R是常用于统计和画图的计算机语言。

## 1.1. R包存储库
R 语言可以通过各种宏包来拓展功能，一般通过中央软件存储库下载R包，如R基金会资助的最常用的[**The Comprehensive R Archive Network (CRAN)**](https://cran.r-project.org/)和存储R开发的生物数据分析包的生物专业软件存储库[**Bioconductor**](https://www.bioconductor.org/)。

## 1.2. R相关
### 1.2.1. RStudio
- RStudio 是 R 的集成开发环境 (IDE)。
- 它包括一个控制台、支持直接代码执行的语法高亮编辑器，以及用于绘图、历史、调试和工作区管理的工具。
- 有两种格式：RStudio Desktop 是一个常规的桌面应用程序，而 RStudio Server 运行在远程服务器上，并允许使用Web 浏览器访问 RStudio。

### 1.2.2. Rtools
- Rtools是 Windows 上从源代码构建 R 包所需的一组程序。
- Rtools为 Windows 平台提供了一个与 R 兼容的工具链，使得Windows平台R的使用与UNIX-ish平台R的使用许多命令一致。它主要包括 GNU make、GNU gcc 和其他在 UNIX-ish 平台上常用的实用程序。
- 如果只使用CRAN 或 Bioconductor 上提供的R包，则无需Rtools，如果使用需要编译的R包（比如ROracle）或者制作自己的R包，则需要Rtools。

# 2. R的安装和配置
## 2.1. R for Windows
### 2.1.1. 安装R
[R官网](https://www.r-project.org/)或者[R清华镜像](https://mirrors.tuna.tsinghua.edu.cn/CRAN/bin/)下载安装最新版本的R(现在最新版是R4.1.2)。
双击之后按步骤安装。

### 2.1.2. 安装RStudio Desktop
[RStudio官网](https://www.rstudio.com/)下载安装最新版本的Rstudio Desktop。
双击之后按步骤安装。

### 2.1.3. 安装Rtools
[Rtools4](https://cran.r-project.org/bin/windows/Rtools/rtools40.html)下载安装最新版本的Rtools。
双击之后按步骤安装。

## 2.2. R for Linux
### 2.2.1. 安装R
#### 2.2.1.1. 安装R
[R 官网](https://cran.r-project.org/)里参考对应系统的R安装教程。
1. Ubuntu
`apt update`,`apt upgrade`,`apt install r-base`
2. Centos
`dnf install R`
3. Debian
`apt-get update`,`apt-get install r-base r-base-dev`
4. conda安装【推荐】
`conda create -n r r-essentials r-base` #创建新环境r，安装r-base(目前是4.1.3版本)，并从CRAN安装所有的r-essentials包，R Essentials 包包含大约 200 个最流行的数据科学 R 包，包括 IRKernel、dplyr、shiny、ggplot2、tidyr、caret 和 nnet。

安装R后`R --version`查看版本，可能非最新版。

#### 2.2.1.2. 手动安装R
[R source installation ref](https://blog.51cto.com/kuxingseng2016/1846326);[ref2](https://xieduo7.github.io/2018-04-02-R%E5%AE%89%E8%A3%85.html)。
1. R的依赖
参考R官方文档[R installation and administration](https://cran.r-project.org/doc/manuals/r-release/R-admin.html#Essential-and-useful-other-programs-under-a-Unix_002dalike)，包括zlib,bzip,liblzma(XZ),PCRE等。
2. R下载
[R官网](https://www.r-project.org/)或者[R清华镜像](https://mirrors.tuna.tsinghua.edu.cn/CRAN/bin/)下载安装最新版本的R(现在最新版是R4.1.2)。
3. R源码安装
R源码是C语言写的，需要编译安装。
```
tar zxvf R-4.1.2.tar.gz
cd R-4.1.2
./configure --enable-R-shlib #配置，生成Makefile
make #编译
make install #安装
```

### 2.2.2. 安装RStudio Server
#### 2.2.2.1. 安装RStudio Server
RStudio Server 运行在远程服务器上，并允许使用Web 浏览器访问 RStudio。

在[RStodio官网](https://www.rstudio.com/products/rstudio/download/)上选择RStudio Server，选相应的系统版本下载。

1. Ubuntu
`sudo apt-get install r-base` # for Ubuntu 16
2. CentOS
`wget https://download2.rstudio.org/server/centos8/x86_64/rstudio-server-rhel-2021.09.1-372-x86_64.rpm`,
`sudo yum install rstudio-server-rhel-2021.09.1-372-x86_64.rpm` # for CentOS 8

#### 2.2.2.2. 卸载/更新RStudio Server
更新也需要先卸载再安装新的版本。
1. `rstudio-server stop` #暂停当前RStudio Server服务
2. `yum remove rstudio-server` #CentOS卸载，或者 `apt-get remove rstudio-server` #Ubuntu卸载

#### 2.2.2.3. 安装后配置RStudio Server
1. 安装后，通过修改/etc/rstudio/rserver.conf更改端口和R所在路径

```
www-port=8787 # 端口, 默认8787
www-address=0.0.0.0
rsession-which-r=/opt/sysoft/R-4.1.2/bin/R # 安装R的路径
```

2. 用`rstudio-server restart`重启服务
没有任何信息就表示安装成功了，还可以`rstudio-server verify-installation`验证，没有输出就是安装成功。

#### 2.2.2.4. 使用RStudio Server
`ps -aux|grep rstudio-server`查看服务是否运行，`ipconfig`查看服务器IP地址。
 
安装配置好RStudio Server后，直接在浏览器打开http://IP:8787，输入服务器的用户名和密码，即可访问RStudio Server服务。

# 3. R包安装
R 语言可以通过各种宏包来拓展功能，一般通过中央软件存储库下载R包，如最常用的R基金会资助的[**The Comprehensive R Archive Network (CRAN)**](https://cran.r-project.org/)，存储R开发的生物数据分析包的生物专业软件存储库[**Bioconductor**](https://www.bioconductor.org/)，以及[**anaconda**](https://anaconda.org/)。

## 3.1. 通过CRAN安装R包
1. `install.packages("packagename")` R默认的安装方式，会从CRAN下载包并安装。
2. 同时安装多个包: `install.packages(c("packagename1","packagename2"))`
3. 使用R包下载地址安装: `install.packages("http://cran.r-project.org/src/contrib/Archive/ggplot2/ggplot2_0.9.1.tar.gz", repos=NULL, type="source")`
4. 下载包到本地后手动安装: `install.packages("ggplot2_0.9.1.tar.gz", repos = NULL)`

## 3.2. 通过Bioconductor后安装R包
1. 安装Bioconductor
```R
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install(version = "3.13")
```
BiocManager的版本与R版本一一对应，安装时如果版本不对会有提示，根据提示安装对应版本即可。
2. 查看可用的Bioconductor包：`BiocManager::available()`
3. 安装Bioconductor库里有的R包：`BiocManager::install("packagename",version="3.2")`
4. 同时安装多个包：`BiocManager::install(c("packagename1","packagename2"))`

## 3.3. 通过conda安装R包
在[anaconda官网](https://anaconda.org/)查询想要安装的R包，如果库里有可以使用conda安装。

一般需要在包名称前添加r-，比如安装rbokeh和rjava。
`conda install r-rbokeh`,`conda install r-rjava`

## 3.4. 更新和卸载R包
1. 更新：`update.packages()`；conda更新R包caret`conda update r-caret`
2. 卸载：`remove.packages()`。

# 4. R包载入
1. library(packagename)
如果包不存在，library会停止执行，不返回任何值。
2. require(packagename)
require会根据包存在与否返回true或false，并继续下面的语句执行。

# 5. R包使用查询
R包使用方法: `help(package="packagename")`

# 6. reference
1. https://www.bioinfo-scrounger.com/archives/435/