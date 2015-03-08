---
layout : post
title : eclipse常用快捷小贴士
category : "eclipse"
tags : [eclipse]
---

eclipse提供的模版/快捷输入功能，可以使我们只输入少量代码，就能自动补充成完成的代码。比如经常使用的`syso`，在按下Alt+/之后，就会自动变成`System.out.println()`.这就是eclipse提供的自定义模版功能。

我们经常在开发中写一些固定格式的代码，所以可以把它们在eclipse里面定义好，方便以后使用。
每次新建的代码，以往都要手动添加作者信息和添加时间。这里也可以去修改代码模版，来达到自动添加信息的功能。
固定代码模版:
>window->preferences->Java->Editor->Code Style->Code Templates


新建类自动添加作者和时间信息：选择Types,点击Edit，添加以下内容:
```java
/**
 * @author 姓名（这里换成开发者自己的姓名）
 * 时间:${date}${time}
 * ${tags}
 */
```
    

自定义缩写：
>window->preferences->Java->Editor->Templates

点击右上角的New..新建一个模版。`Context`选择`Java`,选择`Automatically insert`自动插入。`Description`填写模版的描述

`Logger`Context选择`Java`,下面的代码是Log4j:
```java
private static final Logger logger = LoggerFactory.getLogger(${enclosing_type}.class);
```
现在输入logger，按下alt+/，就会自动补全代码了，不过这样不会自动导包,可以使用Ctrl+Alt+O导入所有需要的包。


`ControllerPage`非responsebody的controller方法，`Context`选择`Java type members`.
```java
@RequestMapping(${value})
public  String ${name} (ModelMap map){
	return "";
}
```

`ControlelrJson` 返回ResponseBody的controller.`Context`选择`java type members`.
```ControllerJson 
@RequestMapping("")
@ResponseBody
public ${return_type} ${method}(){
	return ${return_type};
}
```