---
layout : post
title : win环境下，flask不加载css的问题以及解决
category : "python"
tags : [python,flask]
---

前阵子突然对动态语言来了兴趣，然后又捡起对python的兴趣。

看了半天,优雅的python还是2和3打得飞起。

找了个python的教程看了下，似乎目前已经推荐使用python3.

从一个web从业者的本能来说，一开始就打算直接用来做一个简单的网站就好。

## 选择的版本和框架

python 3.4.3

flask 0.10.1

## 问题

跟着flask的教程，做了一个简单的blog。照猫画虎，很快把功能部分完成了。但是，样式却没有了。调试工具显示加载了css文件。

但是提示`Resource interpreted as Stylesheet but transferred with MIME type application/x-css`，大概意思是，成功用样式表去读取css了，但是文件被以applocation/x-css的mime-type传输回来了。

搜索了一番，发现原来flask这玩意居然是去读取系统的mimetype？！

然而css在windows注册表类型就是`application/x-css`

那么就在运行输入`regedit`，就可以去搜索`css`，然后修改`Content Type`为`text/css`即可。

## 感觉

win用户真是惨，各种新手教程完全忽视win用户。教程要是直接不面向win用户的话，那就很正常。

但是你又支持win用户，又不告诉win用户该注意的tips。

那还做面向win用户的教程干嘛，恶心win用户么？

留下这么多小坑给用户去踩。。