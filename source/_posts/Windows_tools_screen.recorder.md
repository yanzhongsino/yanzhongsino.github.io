---
title: 录屏软件
date: 2022-09-22
categories:
- computer
- Windows
- tools
tags:
- screen recorder
- video
- Xbox
- OBS Studio
- Captura
- Windows

description: 介绍了录屏软件Xbox，OBS Studio，Captura。软件录制视频的使用和设置。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=105136&auto=1&height=32"></iframe></div>

# 1. 录屏软件
尝试了多种录屏方式，也遇到一些问题。记录下来，供参考。

如果在window上简单录制，则优先window+G使用xbox即可。如果是需要录课程等非游戏视频，推荐开源免费的OBS Studio，类似的还有Captura。

# 2. Xbox
Xbox是window10自带的录制游戏的软件，不需下载。

Xbox只能录制应用程序，不能录制桌面。

## 2.1. Xbox使用
1. window+G调出Xbox录屏界面
2. 点击屏幕上方的“捕获”调出捕获小组件
3. 建议点击调出来的捕获小组件右上角的小图钉，在桌面固定捕获小组件
4. 简单地点击“录制”和“停止录制”开关按钮即可开始或停止录制

## 2.2. Xbox设置
1. 把需要录制的软件选为游戏。这是为了避免录制时其他软件的干扰。
- window+G调出Xbox界面
- 点击屏幕上方的“设置”调出设置窗口
- 在弹出的窗口选择“常规”
- 有一个选项：“记住这是一款游戏”，下面会实时显示在最上层的软件，把需要录制的软件放在最上层，选中“记住这是一款游戏”。
2. 后台录制。这是为了避免录制时其他软件的干扰。
- “设置”界面选择“正在捕获”
- 在“正在录制”的下面选择“当我玩游戏时进行后台录制”
3. 音频选择
- 在“正在捕获”界面的“要录制的音频”下选择录制那些音频：游戏、所有、无。
- 我选择的是“所有”，包括游戏和已启用麦克风，系统和其他应用程序。
4. 麦克风
- 在右上角的“查看我的捕获”界面，有个麦克风开关。可以根据需要选择，如果录课则关闭，需要录自己声音则打开。

# 3. OBS Studio
## 3.1. OBS Studio简介
OBS Studio是开源软件，完全免费，适用windows，macOS和Liunx系统，可用于视频录制和推流。

OBS Studio主页：https://obsproject.com/

## 3.2. OBS Studio使用
以录制腾讯会议为例
1. 打开软件
2. 在“来源”中添加需要录制的源
- 点击“+”号，有非常多种来源可选择：包括“窗口采集”，“显示器采集”，“浏览器”，“游戏源”，“媒体源”等等。
- 这里录制腾讯会议，选择“窗口采集”
- 弹出创建源窗口：命名来源，点击确定
- 弹出属性窗口：在“窗口”中选择“腾讯会议”，“显示鼠标指针”可以根据需要选择，点击确定。
3. 使源窗口全屏
- 添加的源未必刚好覆盖录制的全屏，如果这样开始录制，视频会包含大量的黑框部分。
- 可以直接拉伸添加的源外面的红框，使其覆盖全屏录制区域。
- 也可以简单地在“编辑”-“变换”中选择“比例适配屏幕”来最大化源窗口使其适配录制屏幕【推荐】，或者选择“拉伸到全屏”来调整源窗口比例，从而实现完全全屏。
4. 录制
- 在“来源”中出现新建的源，选中这个源，右边的“控件”中使用“开始录制”即可开始，“停止录制”即停止。

## 3.3. OBS Studio设置
### 3.3.1. 打开“暂停录制”功能
软件默认设置是不支持“暂停录制”，可通过修改设置来打开“暂停录制”的按钮。
1. “文件”-“设置”，弹出设置窗口
2. “输出”，下面有显示“当录像质量设为与串流画质相同时，无法暂停录制”。所以需要把“录像”-“录像质量”中选择其他的，我这里选择“高质量”。
3. “输出”-“输出模式”中选择“高级”，然后选择“录像”，下面有显示“当录像编码器设为使用推流编码器时，无法暂停录制。”所以在“录制设置”-“编码器”中选择其他的，我这里选择“x264”。
4. 应用和保存设置。
5. 这时，如果点击“开始录制”，右边会出现一个暂停按钮，暂停功能就打开了。

### 3.3.2. 输出设置
1. “文件”-“设置”，弹出设置窗口
2. “输出”-“录像”这里可以设置录像的质量，格式，编码器等。

# 4. Captura
Captura也是开源的录屏软件，网上看到很多推荐，界面简单。

Captura下载：https://mathewsachin.github.io/Captura/

但是我自己用下来遇到几个问题。
1. 首先FFmpeg解码器我用不了也无法自动下载，自己下载导入后也用不了，就没继续折腾了。
2. 选择窗口录屏，这个窗口一定要一直保持在最上方，如果有其他软件覆盖，就会录到其他软件。
3. 音频录制的选择，要不就没声音，要不就把麦克风录进去，反正没调对。

应该是挺不错的软件，可能是和我电脑哪里不兼容，还是我没设置对，反正最后弃了。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>