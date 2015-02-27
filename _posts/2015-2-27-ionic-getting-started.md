---
layout: post
title : ionic 教程
tags : [hybird app,cordova,ionic,android,ios]
category : "ionic"
---

> 本文是中文版本的[ionic getting started](http://ionicframework.com/getting-started/).

### 跟随下面的几个步骤使用ionic在几分钟内构建高质量移动app。你可以访问得到一个更深入的了解 [额外视频课程](https://www.youtube.com/watch?v=C-UwOWB9Io4&feature=youtu.be)(youtube链接，需要翻墙)，视频涵盖其他你想了解的方方面面.

1.安装ionic

安装[node.js](http://nodejs.org),然后通过npm安装最新版本的cordova和ionic命令行工具（Command-line tools,简称CLI）。

> `npm -g install cordova ionic`

根据[android](http://cordova.apache.org/docs/en/3.3.0/guide_platforms_android_index.md.html#Android%20Platform%20Guide)和[ios](http://cordova.apache.org/docs/en/3.3.0/guide_platforms_ios_index.md.html#iOS%20Platform%20Guide)平台指引，安装各平台所需的依赖。windows用户可以参考[安装视频](http://learn.ionicframework.com/videos/windows-android/)或者尝试下面提供的整合包.

我们同样提供一个一键[vagrant package](https://github.com/driftyco/ionic-cordova-android-vagrant)安装包。

 > ios开发环境需要Max OS X。通过ionic cli使用IOS模拟器需要ios-sim npm包，你可以使用`npm -g install ios-sim`

2.开始一个工程

使用ionic cli创建一个使用完整demo模版的工程或者空白的工程。

> `ionic start myApp tabs`

![_config.yml]({{ site.baseurl }}/images/ionic/blank-app.png) | ![_config.yml]({{ site.baseurl }}/images/ionic/menu-app.png) | ![_config.yml]({{ site.baseurl }}/images/ionic/tabs-app.png)
---| --- | ---
`ionic start myApp blank` |  `ionic start myApp sidemenu` |  `ionic start myApp tabs`

3.运行它

使用ionic工具来构建，测试和运行你的应用。如果你使用android，请确认将ios替换成android。然后，尝试[ionic view](http://view.ionic.io/)在测试人员和用户之间测试应用。

> `cd myApp`
> `ionic platform add ios`
> `ionic build ios`
> `ionic run ios`

就是这么简单。