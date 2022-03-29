---
title: hexo建站，github.io发布，多终端同步
date: 2022-03-29 21:30:00
categories: 
- blog
- config
tags: 
- blog


description: 这篇博客是继hexo+github.io建站后，记录hexo的站点配置文件和主题配置文件的具体配置以及实现常用的功能的操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=418550511&auto=1&height=66"></iframe></div>


# 配置文件
1. 站点配置文件
   hexo建站后，用于配置全局的文件在根目录下，是_config.yml文件。
2. 主题配置文件
   在根目录下的themes目录里存放着主题目录和文件，每一个主题目录内都有主题配置文件_config.yml。在建站时使用中的主题下的主题配置文件才会生效。

## 管理主题配置文件
由于主题配置文件名称与站点配置文件名称一样，且主题配置文件会随着主题的版本更新而变化，需要谨慎管理已修改的主题配置文件，以保持生效和同步。

目前有以下几种常用管理方案：
不变方案的优势时遇到的冲突少，但在主题更新时需要多一点备份和主题配置工作；合并方案和更改方案是为了避免主题更新误把主题配置文件替换，但对于更换主题较为麻烦。

1. 不变方案
- 主题配置修改：保持站点配置文件和主题配置文件的位置和名称，直接在主题配置文件里修改主题配置。
- 主题配置同步：当主题版本更新时，注意把修改过的旧的主题配置文件保存，更新主题版本后，把新的主题配置文件替换成旧的。notes：由于新的主题配置文件可能包含新的设置项，替换旧的后也可能导致主题不生效，可以不替换，而是依据旧的主题配置文件修改新的主题配置文件，确保生效。

2. 合并方案
- 主题配置修改：把主题配置文件的内容复制到站点配置文件，每行都缩进两个空格(在VScode里选中内容，ctrl+]即可缩进)，然后在主题配置内容前一行添加`theme_config:`标题行，主题配置的修改即可在站点配置文件中进行，原有的主题配置文件不改动。
- 主题配置同步：因为站点配置文件是随时与站点保持同步的，所以主题版本更新时，无需额外保存主题配置文件，而是把更新后的主题配置文件中新加项添加到站点配置文件的主题配置项中即可。

3. 更改方案一【需要hexo版本>5.0】
- 主题配置修改：在站点根目录下，创建一个_config.next.yml的文件，把主题配置文件内容复制到_config.next.yml中，并在这个文件进行主题配置。
- 主题配置同步：当主题版本更新，新的主题配置文件中有新加项时，可随时复制新加项的内容到这个文件_config.next.yml中，实现主题的同步。

4. 更改方案二【next v8不支持】
- 上面的更改方案一中的_config.next.yml也可以改成source/_data/next.yml(没有_data目录就创建)

# 站点配置
在[hexo配置文档](https://hexo.io/zh-cn/docs/configuration.html)里查看站点配置。

# 主题的安装 —— 以next为例
[hexo网站](https://hexo.io/themes/)有许多主题可选，hexo主题安装都是一样的，这里以最常用的next主题举例。
## 安装hexo主题
hexo安装主题只需要将主题文件夹拷贝到站点目录的themes目录下。

可以通过git来克隆最新版本的主题，例如克隆next主题，在站点目录下运行`git clone https://github.com/iissnan/hexo-theme-next themes/next`，就会复制最新版本的next主题文件夹到themes下并被命名为next。

## 启用hexo主题
在站点配置文件中把theme字段的值改为`next`即可启用next主题。

## 验证主题
1. 验证主题是否正确启用，最好先用`hexo clean`清楚hexo缓存。
2. `hexo s --debug`启用hexo本地站点，并开启调试模式(--debug)
3. 当命令行输出中提示出：`INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.`即可使用浏览器访问`http://localhost:4000`，检查站点是否正确运行。如果看到主题页即成功安装和启用了主题。

# 主题设定 —— 以next为例
## 主题配置
在[next主题配置文档](https://theme-next.iissnan.com/theme-settings.html)里查看next的主题配置。

next的配置非常丰富，包括：
1. 设置RSS
2. 添加标签页面
3. 添加分类页面
4. 设置字体
5. 设置代码高亮主题
6. 侧边栏社交链接
7. 开启打赏功能
8. 设置友情链接
9. 腾讯公益404页面
10. 站点建立时间
11. 订阅微信公众号
12. 设置动画效果
13. 设置背景动画


## 第三方服务集成
### 评论系统
#### Disqus

### 数据统计与分析
#### 百度统计
添加百度统计数据

1. 登录[百度统计网站](https://tongji.baidu.com/web/10000443118/homepage/index)
2. 选择 管理-代码管理-代码获取
3. 添加博客的网站和网址
4. 获取统计代码
一个统计代码的例子：
```
<script>
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?a1b2c3d4e5f6g7h8j8k9";
  var s = document.getElementsByTagName("script")[0]; 
  s.parentNode.insertBefore(hm, s);
})();
</script>
```
5. 把统计代码中的hm.src值的hm.js?之后的部分，这里的例子是**a1b2c3d4e5f6g7h8j8k9**复制
6. 编辑主题配置文件(比如next主题下的_config.yml)的baidu_analytics字段，把复制的值设置成百度统计脚本id。

#### Google分析

#### 阅读次数统计(LeanCloud)
为next主题的博客添加文章阅读量统计功能

### 内容分享服务

### 搜索服务

### 其他服务


# 主题同步 —— 以next为例

## next主题同步到github出错
如果next主题也是通过git clone获取的，git不能直接管理git项目（有.git文件夹的被识别为一个git项目）中嵌套的其他git项目，即`git add .`,`git commit -m "commit notes"`,`git push`命令对hexo项目下的next项目无效，themes/next主题没有被同步到github端hexo分支。

1. Solution A 把next主题的git项目改为普通文件进行同步
- 删除themes/next目录（删除前备份next目录到其他位置），然后`git add .`,`git commit -m "delete theme next"`,`git push`命令同步到github端的hexo分支。
- 把备份的next目录移回themes下，并把next目录下的.git目录删除，然后后`git add .`,`git commit -m "commit notes"`,`git push`命令同步到github端的hexo分支。此时next目录不作为git项目被同步到github的hexo端，而是普通文件。
- 优点是操作简单，缺点是next主题的更新较为麻烦。

2. Solution B fork+subtree同步next主题
- 删除themes/next目录（删除前备份next目录下修改过的文件，比如主题配置文件_config.yml），然后`git add .`,`git commit -m "delete theme next"`,`git push`命令同步到github端的hexo分支。
- fork主题项目，访问[next](https://github.com/theme-next/hexo-theme-next)主题，点击右上角fork，表示把next这个git项目完整的复制一份到自己的github下，你的github账号下会有一个hexo-theme-next的新仓库。

    *notes：注意fork的next版本与hexo版本相匹配，next V8与hexo v5，否则githubpages为空白（过来人踩过的坑~~）*

- `git remote add -f next git@github.com:yanzhongsino/hexo-theme-next.git` #添加远程仓库hexo-theme-next到本地，仓库名设置为next，远程仓库网址粘贴你的github账号下fork的hexo-theme-next的网址。`git remote`显示所有添加的远程仓库，此时便能看见next。
- `git subtree add --prefix=themes/next next master --squash` #添加subtree，把本地next仓库作为子仓库添加到本地themes/next目录下，分支为master，--squash的意思是把subtree的改动合并成一次commit提交。
- 在本地替换themes/next目录下的主题配置文件_config.yml，把删除前修改好的放进themes/next目录，用`git add .`,`git commit -m "add theme next"`,`git push`命令同步本地的next目录到github端的hexo分支，实现next主题目录与githubio项目一起同步。此时，github端fork的hexo-theme-next项目也成为github端github.io项目的子项目。

# 主题更新 —— 以next为例
## next主题更新造成主题配置文件的冲突
当next主题发生更新时，由于next主题的配置文件_config.yml不一致，使用git pull拉取需要解决冲突问题，或者手动替换配置文件。
1. Solution A 把next主题的配置文件_config.yml合并到博客配置文件_config.yml
- 把next主题的配置文件_config.yml的内容复制到博客配置文件_config.yml，每行都缩进两个空格(在VScode里选中内容，ctrl+]即可缩进)，然后在内容前一行添加`theme_config:`标题行。有任何主题配置都在博客配置文件中进行。

2. Solution B 使用_config.next.yml进行主题配置和同步【需要>hexo5.0】
- [next官方指导doc](https://theme-next.js.org/docs/getting-started/configuration.html)
- 在本地的github.io的工作目录下，创建一个_config.next.yml的文件，把需要更改的主题设置从next/_config.yml复制到_config.next.yml并进行更改，用_config.next.yml进行主题配置和同步。
- 当next主题在配置文件/themes/next/_config.yml增加新的特性时，可随时复制新特性的内容到这个文件_config.next.yml中，实现主题的配置和同步。
- 将本地的修改推送给子项目（github端fork的hexo-theme-next项目）`git subtree push --prefix=themes/next next master`，对github端fork的hexo-theme-next项目进行pull、push操作需要使用`git subtree`命令。
- 平常使用同步同前面介绍的一致。`git add .`,`git commit -m "commit notes"`,`git push`命令同步hexo项目和blog源文件到github端的hexo分支，`hexo clean & hexo g -d`命令部署网站（生成public）并发布到githubio网站（同步到github端的master分支）。
- 当next主题项目下文件更改时，增加了`git subtree push --prefix=themes/next next master`推送本地更改到子项目（github端fork的hexo-theme-next项目）的操作。

3. Solution C 使用source/_data/next.yml进行主题配置和同步
- 使用source/_data/next.yml进行主题配置和同步的方案在next v8版本被告知不支持【20201122】
- 为了避免这种不便，next开发者提供了两种[解决方案](https://github.com/theme-next/hexo-theme-next/blob/master/docs/zh-CN/UPDATE-FROM-5.1.X.md)。
- 把主题配置修改的任务转移到新建的位置：复制一份主题配置文件/themes/next/_config.yml到新建的位置/source/_data/next.yml，并且维持/themes/next/_config.yml的默认值，在/source/_data/next.yml中修改主题配置。当next主题在配置文件/themes/next/_config.yml增加新的特性时，可随时复制新特性的内容到这个新的文件/source/_data/next.yml，这样更新主题时便不需额外处理冲突或者手动替换配置文件了。

- next主题更新【还未验证】
    当next的源项目更新后，希望自己的网站和hexo部署同步更新的操作。
    通过git bash进入本地next主题子项目，更新子项目：`git fetch next master`,`git pull https://github.com/example/hexo-theme-next.git`命令拉取next项目源更新的仓库到本地。
    然后`git add .`,`git commit -m "commit notes"`,`git push`命令推送本地更改到主项目，`git subtree push --prefix=themes/next next master`推送本地更改到子项目。    


# references
1. [hexo配置文档](https://hexo.io/zh-cn/docs/configuration.html)
2. [next](https://theme-next.iissnan.com/)
3. [next主题配置文档](https://theme-next.iissnan.com/theme-settings.html)
