---
title: hexo建站，github发布，多终端同步markdown
date: 2020-11-18 18:50:30
type: "categories"
comments: false
categories: blog
---

- **markdown**是一种适用撰写文本的语法和文件格式，后缀是.md
- **hexo**是把写的markdown文件生成blog网站的工具，hexo是不仅是博客引擎驱动，还是一个快速、简洁高效的博客框架，可生成静态网页。
- **git**是目前世界上最先进的分布式版本控制系统，用于对开发程序或者其他需要长期追踪版本变更的项目/文件的版本控制。
- **github**提供网络站点进行blog的展示和备份

github pages是github网站的一个功能，专门用于管理用户自行编写的静态网页，提供静态网页存储空间，许多人把它用于搭建独立博客。

github pages需要相应的博客引擎驱动，主流的是[jekyllrb](https://jekyllrb.com/)和[hexo](https://hexo.io)。

**部署和发布blog有很多方法。这里介绍的是markdown撰写博客，hexo建立博客网站，通过git管理博客网站，发布博客网站到github平台和多终端同步的操作。**

部署本地blog的写作环境（git+hexo），并用github.io网站发布blog

1. git和github的配置
   新终端搭建博客撰写环境都需要配置
2. hexo的安装和配置
   多终端同步和单终端有一点差别，建议一开始就配置多终端同步并备份到githbu网站，一步到位，避免博客在单终端丢失。
3. 撰写与发布blog
4. github两个分支实现多终端同步
5. 日常blog撰写和备份操作


# git和github的配置

*notes：username和useremail替换成自己的github用户名和邮箱。*

1. 下载安装[git](https://git-scm.com/)，[node.js](https://nodejs.org/zh-cn/)
     
     使用git bash或者cmd终端检测node的安装，node -v和npm -v出现版本号即安装成功

2. 注册github网站
     
     注册github网站，并创建一个名为username.github.io的仓库
     
     这个特殊的名称直接关联了gitpages，gitpages服务器会自动把上传到这个仓库的文档进行部署发布，并可以通过分配的域名https://username.github.io/实现博客的访问。

3. git配置和生成SSH key密钥对
     
     git配置：打开git bash（任意位置右键选择git bash here)
     
     运行`git config --global user.name username`和`git config --global user.email useremail`
            
     cd ~进入用户主目录，运行`ssh-keygen -t rsa -C useremail@example.com`，一路回车，完成后会在用户主目录（C/Users/you/)下生成.ssh目录，里面有id_rsa和id_rsa.pub两个文件，即SSH key密钥对，id_rsa是私钥，需要保密，id_rsa.pub是公钥，可分享。

4. github添加终端的公钥
     
     打开网页，登录github，打开settings->SSH and GPG keys，点击new SSH key，填上任意title，在key文本框粘贴公钥，即id_rsa.pub文件的内容，添加。

5. 验证与github的添加
     
     在git bash下运行`ssh -T git@github.com`。
     输出You've successfully suthenicated.则代表github账户成功授权你当前使用的终端，本机可连接上github了。

# hexo的安装和配置
1. 下载安装node.js（建议官网下载）
        
    使用git bash或者cmd终端检测node的安装，`node -v`和`npm -v`出现版本号即node和npm 安装成功

2. 安装hexo
        
    在选定位置创建并进入hexo目录，打开git bash，输入`npm install hexo-cli -g`，`npm install hexo --save`

3. 初始化目录`hexo init`
        
    即生成一系列建站需要的文件：
    - _config.yml：站点配置文件
    - node_modules目录:安装的模块，用npm install会重新生成
    - package.json：说明使用哪些包
    - package-lock.json
    - scaffolds目录：文章模板
    - source目录：blogmarkdown源文件
    - theme目录：网站主题

    - .gitignore：记录提交时忽略的文件，即以下文件
    - public目录：使用hexo g命令时会重新生成
    - .deploy_git目录：hexo d命令时会重新生成
    db.json

4. 安装组件`npm install`，`npm install hexo-server --save`,`npm install hexo-deployer-git`
        
    `npm install`安装依赖的组件，会更新node_modules目录

    `npm install hexo-server --save`安装hexo-server以使用服务器

    `npm install hexo-deployer-git`安装hexo-deployer-git，实现通过`hexo -g`部署hexo网站到github端

5. 配置博客布署deploy参数
   
   修改hexo下的站点配置文件_config.yml中deploy参数。
   repository修改成github.io库的网址，branch填master分支。
    
    ```
    deploy:
        type: git
        repository: https://github.com/username/username.github.io.git
        branch: master
    ```

6. hexo其他配置
    
    在创建的博客网站文件夹下的`_config.yml`文件中进行网站的大部分配置。参考[hexo配置](https://hexo.io/zh-cn/docs/configuration.html)


   + 更换主题【optional】
        
        hexo网站https://hexo.io/themes/有许多主题可选，比如常被使用的NexT主题
        在Hexo主题页面ctrl+F并输入next查找到NexT主题，然后点击进入到NexT主题的github页面，该页面存储了NexT主题的源码,复制next主题的github仓库位置，并在hexo/themes位置下运行命令`git clone https://github.com/theme-next/hexo-theme-next.git`克隆next主题
        
        然后修改hexo下的站点配置文件_config.yml中的主题行为theme: next

   + 网站其他配置【optional】
        
        更改站点配置文件_config.yml中其他参数，例如

        ```
            # Site
            title: username's blog #网站标题
            subtitle: '' #网站副标题
            description: '' #网站描述（主要用于SEO）
            keywords: 
            author: username #作者
            language: en #网站使用语言
            timezone: Asia/Shanghai #网站时区，要在时区列表中选取，Hexo默认使用您电脑的时区
        ```

# 撰写与发布blog

1. 撰写blog
   
   blog文件存放在hexo/source/_posts下，markdown（.md)格式文件，在这个位置新建和编辑markdown文档，这就是写文章的地方。

2. 发布blog

    使用git bash进入博客建站所在位置hexo目录下，使用`hexo clean & g -d` 合并三条命令实现发布blog，或者一步一步实现

    + `hexo clean` 【optional】
        INFO  Validating config
        INFO  Deleted database.
        INFO  Deleted public folder.
        清除缓存文件（db.json\)和已生成的静态文件（public目录）
    + `hexo generate`
        生成静态页面，静态网页会根据source>\_posts目录下所有markdown文件编译成静态网页（html），生成的静态文件会在博客网站文件夹下生成的public文件夹中。
        
        生成静态文件后，可以使用`hexo serve`命令启动本地服务器来查看博客网站。

    + `hexo deploy`
        部署public到github的master分支（部署位置：站点配置文件_config.yml的deploy参数设置的github账号和分支名），即部署到gitpages。
        
        部署之后，可以在githube的page项目中查看到变化。我的是<https://github.com/yanzhongsino/yanzhongsino.github.io>。很快也可以在<https://yanzhongsino.github.io/>线上看到生成的博客。
    
每次修改blog或新增blog内容后，可用hexo clean & g -d快速部署到github端master分支，从而实现github.io网站博客的更新。

# github两个分支实现多终端同步【推荐配置】
在进行hexo的安装和配置后，可以通过命令`hexo clean & g -d`发布博客网站到github.io网站，但需要备份博客源文件和多终端协作时，便有一些问题。

- 单终端配置hexo的问题：
    
    hexo建站部署文件框架为：blog原始md文件存放在hexo建站位置的/source/_posts下，发布blog的命令`hexo clean & g -d`会生成/public并把它部署到github端的master分支，但在source/_posts下的原始md文件和hexo建站部署文件只在当前电脑存放。
- 多终端同步的意义：
    
    把博客md源文件和hexo部署文件备份到github，实现多端共享，共享md源文件可以避免单终端md博客文件丢失，共享hexo部署文件可以在不同电脑上写blog，并部署到github终端，保持一致。
- 多终端同步实现原理：
    
    在github上新建hexo分支，hexo分支用于同步博客md源文件和hexo配置文件（即同步整个hexo目录），master分支用于发布博客到github.io网站（即同步public目录）。两个分支相互独立，永远不要合并。
- 多终端同步实现具体操作:
    
    github网站新建hexo分支，并把hexo分支设为默认分支，`git clone`克隆hexo分支到本地终端，把克隆的目录下的.git文件夹移动到hexo配置好的建站目录hexo下。
    用三条命令`git add .`，`git commit -m "submit"`，`git push origin hexo`同步.git文件夹所在目录到github端hexo分支，这样操作之后克隆github端hexo分支便包含了hexo部署文件和md源文件，从而实现多端共享，github端的master分支只用于发布blog（同步生成的public目录）。

## 多终端同步配置
多终端同步配置具体步骤如下：

1. 安装node.js
2. 在github网页新建hexo分支并设为default默认分支
3. 克隆hexo分支到本地
    
    在git bash下进入想创建存储blog文件的位置，运行`git clone git@github.com:username/username.github.io.git`，在当前目录克隆github的github.io仓库的默认分支（应为hexo）。

    克隆的目录中的.git文件夹是github端hexo分支的版本库Repositor，.git文件夹所在目录（目前是username.github.io）对应github的hexo分支（把.git移动到哪hexo分支就定位到哪）。

4. 移动.git目录

    .git目录是通过git命令与github端hexo分支同步的关键文件。

   + git bash下进入github.io后缀文件夹
   + 进入git clone生成的github.io后缀目录，除.git目录外的文件可以不用保留，.git目录移动到hexo安装和建站的目录下。

5. 同步本地hexo部署文件和blog源文件到github端的hexo分支
   
   在hexo部署目录下通过git三步命令同步hexo部署目录到github端的hexo分支。
   
   以后有修改新增blog或更改配置都用这三步同步hexo分支。

    + git add .        
        
        添加所有工作区（即本地github.io文件夹）的修改到暂存区(即.git里的stage/index文件)

    + git commit -m "commit notes"
        
        提交暂存区的所有更改到本地的默认分支（此时为hexo）。

    + git push/git push origin hexo
        
        提交本地分支到github端的hexo分支，origin指本地默认分支（即.git文件夹所在目录），github端hexo是默认分支，所以可以省略origin hexo，直接用git push便会同步本地github.io仓库到github端的默认分支（即hexo）。

        从而实现github端hexo分支备份了本地hexo部署目录，包括目录下的source文件夹的blog源文件。

6. blog撰写与发布
   
   blog撰写与发布单终端和多终端是一样的操作。
   撰写在source/_posts下进行，推荐使用markdown语法撰写博客；发布使用`hexo clean & g -d`命令。

## 配置后新终端使用同步功能
按照以上进行多终端同步配置后，在新的终端或者本地文件丢失后，可以通过以下步骤快速搭建博客撰写环境并同步博客数据。

1. git和github的安装配置同上
2. 安装node.js
3. 克隆hexo分支到本地
    
    在git bash下进入想创建存储blog文件的位置，运行`git clone git@github.com:username/username.github.io.git`，在当前目录克隆github的github.io仓库的默认分支（应为hexo）。
4. 安装组件
   通过git bash进入本地github.io文件夹下依次执行`npm install hexo`,`npm install`,`npm install hexo-deployer-git`（注意不要 hexo init）安装hexo和需要的组件
5. 进行日常blog撰写和备份


# 日常blog撰写和备份操作
- blog撰写
   
   在本地source/_posts下添加和修改md文档实现blog的日常撰写。

- blog备份
   
   只要blog有更改或者新增，或者配置文件有修改，即工作区（即本地的hexo目录或github.io目录）有文件修改，则建议对文件进行备份到GitHub端的hexo分支。
   用三条命令`git add .`，`git commit -m "submit"`，`git push origin hexo`备份工作区，包括md博客源文件和hexo部署到github端的hexo分支。

- blog发布
    
    可根据自身需求决定是否发布blog到github.io网站，一般写的blog完整程度比较高时可以发布。使用``hexo clean & g -d``命令，根据source/_posts下的博客源文件生成public目录（网站html并同步到github端的master分支，即发布blog到github.io网站。



**小记**
学习hexo+githubpages建站已是两年前（2018.06）的事，那时初学前端，好多新知识需要记录，就先建好站用来发布。结果转行前端从入门到放弃只不过两三月，便把博客搁置了。

前几天拾起来，建站知识已忘得精光，原来应该是配置好了hexo的多端同步，但由于两年的荒废，遗忘导致我把hexo和master分支合并了，博客的原始文件有没备份，所以一切从头开始。

学了几天，博客搭建初步捡回来了。可不能再忘，这次笔记记得很精细，下次如果需要看看笔记应该可以捡回来。

2020.11.18 Yan Zhong in Guangzhou