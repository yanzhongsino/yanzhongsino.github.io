---
title: glup
date: 2018-06-12 14:53:00
categories:
- computer
- web
tags: gulp
description: gulp基础知识
---

# glup

用来压缩打包html、css、img等。

gulp的工作方式是**流（stream）**，使用pipe作为传输管道。

**gulp的使用流程一般是**：首先通过gulp.src()方法获取到想要处理的文件流，然后把文件流通过pipe方法导入到gulp的插件中，最后把经过插件处理后的流再通过pipe方法导入到gulp.dest()中，gulp.dest()方法则把流中的内容写入到文件中。

获取流  
gulp.src(global[,options])

写文件  
gulp.dest(path[,options])

监视文件  
gulp.watch(global[,options],tasks)  
或 gulp.watch(global[,options,cb])

定义任务  
gulp.task(name[, deps], fn)  
name为default的task会在执行gulp没有指定task的name时执行。

执行任务  
gulp.run()
