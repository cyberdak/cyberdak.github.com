---
layout: post
title: ionic 更新库
tags: [ionic]
category: 'ionic'
---

最新ionic发布到了rc2版本，正好需要把之前一个使用beta14版本app升级到最新的lib。

ionic CLI 提供了`ionic lib update` 来升级ionic库文件。但是实际在命令行中执行完这个命令，ionic默认又直接去找bower.json中的依赖了，而json文件中的依赖还是0.0.14beta.

所以，删除app根目录下面的bower.json，担心的话，直接改名就好。删除www/lib/ionic/下的bower.json和.bower.json。

这时再运行`ionic lib update` 提示

>Unable to load ionic liv version information

>Are you sure you want to replace `d:/code/ionic/app/www/lib/ionic` with an updated version of ionic.

这时直接输入yes，因为墙的问题，如果失败了就在重试一次就行了。

之后程序会下载最新版本的库文件，然后就更新好了。