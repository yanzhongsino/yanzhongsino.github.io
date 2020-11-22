---
title: hexo建站，github发布，多终端同步
date: 2018-06-04 15:53:00
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

1. 下载安装[git](https://git-scm.com/)
     
     

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
1. 下载安装[node.js](https://nodejs.org/zh-cn/)
        
    使用git bash或者cmd终端检查node的安装，`node -v`和`npm -v`出现版本号即node和npm 安装成功

2. 安装hexo
        
   `npm install hexo-cli -g`命令安装hexo-cli（hexo的cli命令行模块）

3. 初始化目录`hexo init`
    选定位置通过git bash进入，创建hexo文件夹并进入hexo，hexo init命令初始化hexo文件夹。   
    即生成一系列建站需要的文件：
    - _config.yml：站点配置文件
    - node_modules目录:安装的模块，用npm install会重新生成
    - package.json：说明使用哪些包
    - package-lock.json
    - scaffolds目录：文章模板
    - source目录：blogmarkdown源文件
    - theme目录：网站主题

    - .gitignore：记录提交时忽略的文件，即以下文件
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

    使用git bash进入博客建站所在位置hexo目录下，使用`hexo clean & hexo g -d` 合并三条命令实现发布blog，或者一步一步实现

    + `hexo clean` 【optional】
        INFO  Validating config
        INFO  Deleted database.
        INFO  Deleted public folder.
        清除缓存文件（db.json\)和已生成的静态文件（public目录）
    + `hexo generate`
        生成静态页面，静态网页会根据source>\_posts目录下所有markdown文件编译成静态网页（html），并生成public目录存储生成的静态文件。
        
        生成静态文件后，可以使用`hexo serve`命令启动本地服务器来查看博客网站。

    + `hexo deploy`
        部署public到github的master分支（部署位置：站点配置文件_config.yml的deploy参数设置的github账号和分支名），即部署到gitpages。
        
        部署之后，可以在githube的page项目中查看到变化。我的是<https://github.com/yanzhongsino/yanzhongsino.github.io>。很快也可以在<https://yanzhongsino.github.io/>线上看到生成的博客。
    
每次修改blog或新增blog内容后，可用hexo clean & g -d快速部署到github端master分支，从而实现github.io网站博客的更新。

# github两个分支实现多终端同步【推荐配置】
在进行hexo的安装和配置后，可以通过命令`hexo clean & hexo g -d`发布博客网站到github.io网站，但需要备份博客源文件和多终端协作时，便有一些问题。

- 单终端配置hexo的问题：
    
    hexo建站部署文件框架为：blog原始md文件存放在hexo建站位置的/source/_posts下，发布blog的命令`hexo clean & hexo g -d`会生成/public并把它部署到github端的master分支，但在source/_posts下的原始md文件和hexo建站部署文件只在当前电脑存放。
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
   
   以后有修改新增blog或更改配置都用这三步同步hexo分支。三条命令执行前建议通过`hexo clean`清除缓存和public目录，以免备份不需要的文件。

    + git add .        
        
        添加所有工作区（即本地github.io文件夹）的修改到暂存区(即.git里的stage/index文件)

    + git commit -m "commit notes"
        
        提交暂存区的所有更改到本地的默认分支（此时为hexo）。

    + git push/git push origin hexo
        
        提交本地分支到github端的hexo分支，origin指本地默认分支（即.git文件夹所在目录），github端hexo是默认分支，所以可以省略origin hexo，直接用git push便会同步本地github.io仓库到github端的默认分支（即hexo）。

        从而实现github端hexo分支备份了本地hexo部署目录，包括目录下的source文件夹的blog源文件。

6. blog撰写与发布
   
   blog撰写与发布单终端和多终端是一样的操作。
   撰写在source/_posts下进行，推荐使用markdown语法撰写博客；发布使用`hexo clean & hexo g -d`命令。

## 配置后新终端使用同步功能
按照以上进行多终端同步配置后，在新的终端或者本地文件丢失后，可以通过以下步骤快速搭建博客撰写环境并同步博客数据。

1. git和github的安装配置同上
2. 安装node.js
3. 克隆hexo分支到本地

    在git bash下进入想创建存储blog文件的位置，运行`git clone git@github.com:username/username.github.io.git`，在当前目录克隆github的github.io仓库的默认分支（应为hexo）。

4. 安装组件
   
   通过git bash进入本地github.io文件夹下依次执行 `npm install hexo-cli -g`命令安装hexo-cli（hexo的cli命令行模块）,`npm install`命令安装依赖的模块,`npm install hexo-deployer-git`命令安装部署模块hexo-deployer-git（注意不要 hexo init）。

5. 进行日常blog撰写和备份


# 日常blog撰写和备份操作

1. blog同步
  
  养成习惯，每次开始撰写blog前都通过git bash进入工作区，进行`git pull`命令把github端的hexo分支的更新（更新可能是其他终端上提交的）同步到本地，实现多终端的内容完全同步。

2. blog撰写
   
   在本地source/_posts下添加和修改md文档实现blog的日常撰写。

3. blog备份
   
   只要blog有更改或者新增，或者配置文件有修改，即工作区（即本地的hexo目录或github.io目录）有文件修改，则建议对文件进行备份到GitHub端的hexo分支。
   用三条命令`git add .`，`git commit -m "submit"`，`git push origin hexo`备份工作区，包括md博客源文件和hexo部署到github端的hexo分支。三条命令执行前建议通过`hexo clean`清除缓存和public目录，以免备份不需要的文件。

4. blog发布
    
    可根据自身需求决定是否发布blog到github.io网站，一般写的blog完整程度比较高时可以发布。使用`hexo clean & hexo g -d`命令，根据source/_posts下的博客源文件生成public目录（网站html并同步到github端的master分支，即发布blog到github.io网站。


总结一下，在配置好写作环境后的任意一台终端的日常工作流应该是：
1. `git pull`同步远程github库的更新
2. 在source/_posts/下添加md格式的blog，或者修改已有的blog
3. `git add .`,`git commit -m "commit notes"`,`git push`把修改备份到github端
4. 下次写作重复以上三个步骤
5. 直至blog完善成熟后，用命令`hexo clean & hexo g -d`生成网站并部署到github.io


# next主题同步和更新

- Issue
    建站日记写好后，在另一个终端通过git clone，修改后部署发现githubio没有显示，检查后发现由于next主题也是通过git clone获取的，git不能直接管理一个git项目（有.git文件夹的被识别为一个git项目）中嵌套的其他git项目，即`git add .`,`git commit -m "commit notes"`,`git push`命令对hexo项目下的next项目无效，themes/next主题没有被同步到github端hexo分支。

- Solution A 把next主题的git项目改为普通文件进行同步
    删除themes/next目录（删除前备份next目录到其他位置），然后`git add .`,`git commit -m "delete theme next"`,`git push`命令同步到github端的hexo分支。
    把备份的next目录移回themes下，并把next目录下的.git目录删除，然后后`git add .`,`git commit -m "commit notes"`,`git push`命令同步到github端的hexo分支。此时next目录不作为git项目被同步到github的hexo端，而是普通文件。
    优点是操作简单，缺点是next主题的更新较为麻烦。

- Solution B fork+subtree同步next主题
    删除themes/next目录（删除前备份next目录下修改过的文件，比如主题配置文件_config.yml），然后`git add .`,`git commit -m "delete theme next"`,`git push`命令同步到github端的hexo分支。

    fork主题项目，访问[next](https://github.com/theme-next/hexo-theme-next)主题，点击右上角fork，表示把next这个git项目完整的复制一份到自己的github下，你的github账号下会有一个hexo-theme-next的新仓库。

    *notes：注意fork的next版本与hexo版本相匹配，next V8与hexo v5，否则githubpages为空白（过来人踩过的坑~~）*

    `git remote add -f next git@github.com:yanzhongsino/hexo-theme-next.git` #添加远程仓库hexo-theme-next到本地，仓库名设置为next，远程仓库网址粘贴你的github账号下fork的hexo-theme-next的网址。`git remote`显示所有添加的远程仓库，此时便能看见next。

    `git subtree add --prefix=themes/next next master --squash` #添加subtree，把本地next仓库作为子仓库添加到本地themes/next目录下，分支为master，--squash的意思是把subtree的改动合并成一次commit提交。

    ~~在本地替换themes/next目录下的主题配置文件_config.yml，把删除前修改好的放进themes/next目录~~，用`git add .`,`git commit -m "add theme next"`,`git push`命令同步本地的next目录到github端的hexo分支，实现next主题目录与githubio项目一起同步。此时，github端fork的hexo-theme-next项目也成为github端github.io项目的子项目。

    
- Isssue
    当next主题发生更新时，由于next主题的配置文件_config.yml不一致，使用git pull拉取需要解决冲突问题，或者手动替换配置文件。
    
~~- Solution A 使用source/_data/next.yml进行主题配置和同步~~
    使用source/_data/next.yml进行主题配置和同步的方案在next v8版本被告知不支持【20201122】
    ~~为了避免这种不便，next开发者提供了两种[解决方案](https://github.com/theme-next/hexo-theme-next/blob/master/docs/zh-CN/UPDATE-FROM-5.1.X.md)。~~
    ~~这里介绍第二种NexT方式的解决方案。~~
    
    ~~把主题配置修改的任务转移到新建的位置：复制一份主题配置文件/themes/next/_config.yml到新建的位置/source/_data/next.yml，并且维持/themes/next/_config.yml的默认值，在/source/_data/next.yml中修改主题配置。当next主题在配置文件/themes/next/_config.yml增加新的特性时，可随时复制新特性的内容到这个新的文件/source/_data/next.yml，这样更新主题时便不需额外处理冲突或者手动替换配置文件了。~~

- Solution B 使用_config.next.yml进行主题配置和同步【需要>hexo5.0】
    在本地的github.io的工作目录下，创建一个_config.next.yml的文件，把需要更改的主题设置从next/_config.yml复制到_config.next.yml并进行更改，用_config.next.yml进行主题配置和同步。

    当next主题在配置文件/themes/next/_config.yml增加新的特性时，可随时复制新特性的内容到这个文件_config.next.yml中，实现主题的配置和同步。

    将本地的修改推送给子项目（github端fork的hexo-theme-next项目）`git subtree push --prefix=themes/next next master`，对github端fork的hexo-theme-next项目进行pull、push操作需要使用`git subtree`命令。

   
平常使用同步同前面介绍的一致。`git add .`,`git commit -m "commit notes"`,`git push`命令同步hexo项目和blog源文件到github端的hexo分支，`hexo clean & hexo g -d`命令部署网站（生成public）并发布到githubio网站（同步到github端的master分支）。
当next主题项目下文件更改时，增加了`git subtree push --prefix=themes/next next master`推送本地更改到子项目（github端fork的hexo-theme-next项目）的操作。


- next主题更新【还未验证】
    当next的源项目更新后，希望自己的网站和hexo部署同步更新的操作。
    通过git bash进入本地next主题子项目，更新子项目：`git fetch next master`,`git pull https://github.com/example/hexo-theme-next.git`命令拉取next项目源更新的仓库到本地。
    然后`git add .`,`git commit -m "commit notes"`,`git push`命令推送本地更改到主项目，`git subtree push --prefix=themes/next next master`推送本地更改到子项目。    



**小记**

学习hexo+githubpages建站已是两年前（2018.06）的事，那时初学前端，好多新知识需要记录，就先建好站用来发布。结果转行前端从入门到放弃只不过两三月，便把博客搁置了。

前几天拾起来，建站知识已忘得精光，原来应该是配置好了hexo的多端同步，但由于两年的荒废，遗忘导致我把hexo和master分支合并了，博客的原始文件有没备份，所以一切从头开始。

学了几天，博客搭建初步捡回来了。可不能再忘，这次笔记记得很精细，下次如果需要看看笔记应该可以捡回来。

2020.11.18 Yan Zhong in Guangzhou

next主题的同步稍微有点麻烦，在理解git的各种命令的含义的基础上理解各种操作实现了什么目的会更有帮助。

2020.11.22 Yan Zhong in Guangzhou