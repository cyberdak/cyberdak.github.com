---
layout : post
title : 使用google chrome frame
category : "前端"
tags : [chrome,web,front]
---


之前浏览别人网页发现这么一句话

><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">

字面意思就是优先使用chrome，没有chrome的情况下，使用ie。

按字面理解，我以为这是一个国产特性，各种混合chrome和ie的浏览器。在这种情况下强制浏览器走chrome。

放狗搜索了一番，发现不是这么回事。

一下内容摘录自wiki：

### 谷歌浏览器內嵌框架

>Google Chrome Frame（中国大陆官方译名：谷歌浏览器內嵌框架，台湾、香港官方译名：Google Chrome內嵌框架）是专为Internet Explorer设计的一个插件。相应的开源计划为Chromium的一部分，其采用BSD许可证授权并开放源代码。

>这插件可运行于Windows 7、Vista、XP SP2或更高版本操作系统中的Internet Explorer 6、7、8、9，使Internet Explorer可以基于谷歌浏览器中的Webkit引擎及V8引擎进行排版及运算，即能获得比原有Internet Explorer更快、更安全网页浏览体验[4]，并且令Internet Explorer 6、7、8支持HTML5代码。

>浏览使用支持Chrome Frame的Google服务会自动使用Chrome内核渲染，如Youtube、Google文档、Orkut等已经打开了对Chrome Frame的支持。

>开发原意是使不支持HTML5的Internet Explorer也能浏览Google Wave及其它使用了HTML5代码的Google服务[5]。

>此项目已于2014年2月25日正式停止维护与更新.

### 网页开发

`网页设计员可以在网页中加入以下代码使网站能以谷歌浏览器 Frame浏览。`

><meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1">

`若浏览者有安装Chrome Frame，且浏览者的IE浏览器版本为IE8或更低，此代码会自动引导浏览器激活插件进行排版及运算；但若浏览者并没有安装插件或IE版本为IE9或更高，则不会进行任何动作。
浏览者亦可以在网址前增加gcf:前缀来要求Internet Explorer以插件进行排版及运算。`


然并卵，安装chrome的人还是太少了