---
title: 离线转录视频/音频成文本的软件
date: 2025-07-16
categories: 
- software
- tools
- transcribe
tags: 
- software
- transcribe
- translate
- Open AI
- Whisper
- Buzz

description: 介绍把语音或视频转录成文本的开源软件 Open AI的Whisper，以及基于Whisper开发的Buzz。
---  

<div align="middle"><iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/6qwU563KYfEPIW6IMhAJos?utm_source=generator" width="50%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe></div>

# 1. 背景
- 把视频或语音转录成文本，以及实时转录的软件有非常多，许多商业公司开发的软件前期都免费，但后期开始收费，如科大讯飞、剪映、网易见外等。
- 现在手机大部分支持录音转文本，但是录音很占内存，有时使用电脑处理更方便。

# 2. Windows自带转录功能
如果想要马上上手，可以使用Windows给视频/音频添加同步字幕的软件（快捷键：Windows+Ctrl+L），以及实时转换语音成文本的软件（快捷键：Windows+H）

# 3. 离线转录软件Whisper/Buzz
## 3.1. 简介Whisper/Buzz
1. Open AI在2022年9月21日开源了号称其英文语音辨识能力已达到人类水准的Whisper神经网络，且它亦支持其它99种语言的自动语音辨识。
2. Whisper系统所提供的自动语音辨识（Automatic Speech Recognition，ASR）模型是被训练来运行语音辨识与翻译任务的，它们能将各种语言的语音变成文本，也能将这些文本翻译成英文。
3. 但是Whisper的安装和配置稍微复杂，可以用其他基于Whisper开发的软件替代，比如Buzz。
4. Buzz是一款可以离线运行的语音识别软件。它有两个模式，一个是录音转文字，一个是实时语音识别，同时还支持翻译。它的底层还是使用的whisper的语音识别功能。

## 3.2. Buzz下载
1. Buzz可在github下载，https://github.com/chidiwilliams/buzz，支持Windows、Mac、Linux等系统。
2. Window系统下载exe文件，双击安装即可使用。
3. 第一次转录的时候会下载模型，下载完后即可离线运行。

## 3.3. 使用步骤
Buzz支持转录视频/音频文件成文本，也支持实时转录。如果要转录视频/音频，点击加号来导入音频/视频文件。如果要实时转录，点击话筒符号。

1. 点击"File",再点击"Import File"，导入音频或者视频文件。或者点击话筒符号进行实时转录。
2. 然后在对话框中选择
- 模型（Whisper/Whisper.cpp/Faster Whisper/Hugging Face/Open AI Whisper API）
- Whisper模型包括：Large-V3-Turbo,Large-V3,Large-V2,Large,Medium,Small,Base,Tiny），模型越大消耗的CPU和GPU越多，转录速度越快。
- 任务（转录Transcribe/翻译Translate）
- 语言（English、Chinese等99种语言，也可以让模型检测语言）
- 高级设置中还可以选择启动AI翻译，支持gpt-3.5-turbo等模型
- 并支持选择**逐词识别**
- 导出文件支持TXT,SRT,VTT格式。SRT是具有时间戳的文本文件，是常见的字幕文件格式。用PotPlayer等视频播放器导入SRT文件即可显示字幕。
3. 转录结束后，会自动保存到视频/音频文件所在目录下。或者点击双箭头符号，可以手动保存文本文件。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>