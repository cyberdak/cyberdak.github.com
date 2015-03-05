---
layout : post
title : ionic结合ngCordova调用原生平台api
tags : [ionic,ngCordova]
category : "ionic" 
---

ionic这样的html5移动框架，在开发移动应用时，提供了很大的便利。但是如果需要结合android或者ios本身的系统特性来实现某些功能呢？这时候只靠ionic就没法实现这些功能了。cordova就是这样的框架，提供了相应的插件api来操纵android或者ios。cordova提供了原生的js接口来操纵移动系统。但是ionic本身已经使用了angularjs，使用原生cordova插件的话，就没法使用angularjs了。所以ionic团队提供了一个使用angularjs的cordova版本:[ngCordova](http://ngCordova.com)

ngCordova
---
ionic 团队提供的angularjs版本的cordova插件框架集合，目前有63+的插件支持。ngcordova的官网提供了[文档](http://ngocrdova.com/docs)和[下载](https://github.com/driftyco/ng-cordova/archive/master.zip)。

ngCordova的官网提供了安装的教程和使用的教程。还提供了一个插件列表，但是可能官网的维护不给力的原因，官网的插件列表没有github库里面的插件多。所以这里推荐读者直接去ngcordova的github库去查看目前支持的插件列表。

ngCordova目前的一个问题，重写了很多原来插件的api，但是又没有具体的文档，导致查阅文档时，有点无头苍蝇的感觉。（有空尽量补齐一些插件的api。）

安装ngCodeova
===

获得代码
---

命令行下，进入相应的ionic项目目录

>`bower install ngCordova`

>bower是一个依赖nodejs的js依赖管理工具。使用`npm install -g bower`来安装bower。

在index.html添加一下语句，ng-cordova.js要放在cordova之前，angularjs之后。

```html
<script src="lib/ngCordova/dist/ng-cordova.js"></script>
<script src="cordova.js"></script>
```

注入angular依赖
---

在你的angular模块注入ngCodova

> `angular.module('myApp',[ngCordova])`

用`deviceready`事件包裹每一个插件调用。 - 重要!
---

在每个插件调用之前，你必须检查设备是否完全载入。

	document.addEventListener('deviceready',function(){
		$cordovaPlugin.someFunction().then(success,error);
	}, false);

	// 或者使用ionic

	$ionicPlatform.ready(function(){
		$cordovaPlugin.someFunction().then(success,error);
	});

使用cordova cli添加插件到你的项目
---

现在你可以使用cordova的api来添加插件到你的项目里面。	

>cordova plugin add ...	