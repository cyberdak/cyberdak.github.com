---
layout : post
title : offical ionic book
category : "ionic"
tags : [ionic]
---

#The Ionic Book

欢迎来到ionic官方指引教程。我们将通过一下进程来了解ionic安装，相应的依赖安装，创建一个新项目，设计和编译UI，编写业务逻辑，测试和部署应用到移动设备上，发布应用到app store和google play。

1. 什么是ionic
2. 安装
3. 开始你的app
4. 测试你的app
5. 构建app
6. 发布

章节1 ： 关于ionic的一切
===
欢迎来到由ionic的创建者书写的使用ionic框架来构建HTML5移动apps的官方教程。它包含在开始app开发中你所需要了解的一切。

如果你过去曾经使用过其他的移动开发框架，那么你就会发现ionic的使用方法和它们非常相似。但是从任何一个新框架开始接触，都是令人生畏的。所以，我们从简单开始，慢慢扩展它。但是首先，我们需要谈一谈，关于ionic项目本身的东西，为什么它适应开发堆栈(原文是fits dev stack，实在不知道怎么翻译了，字面意思翻译过来。)，以及为什么我们创建它。

什么是ionic，它适应那些场景？
---

ionic是一个以构建混合移动应用为目标的HTML5移动应用开发框架。混合应用是一种运行在app的一个浏览器壳的嵌入式小型网站并且可以通过原生平台api层。混合应用有很多超过原生应用的好处，特别在平台支持，开发速度以及第三方代码的条款上。

想想看，ionic作为前端UI框架，可以处理所有的视觉效果和用户界面。类似“原生的bootstrap”一样，但是ionic拥有范围广泛的通用原生移动组件，华丽的动画以及漂亮的设计。

不像一个响应式的框架，ionic带来了原生风格的UI元素，ionic可以使你在拥有ios或者android的原生sdk时，产生实际的布局文件。在web上，她们是不存在的。ionic同样可以提供一些自用但是有效的方法来使用安装了HTML5开发框架的eclipse来构建移动应用。

ionic是一个HTML5框架，为了像原生应用一样运行，需要一个原生平台包装器，比如 `Cordova` 或者 `PhoneGap`。我们强烈推荐使用适当的`Cordova`来构建您的程序，而且ionic工具包会基于`cordova`。

章节2：安装
===

在这一章，我们将讲解ionic的下载以及它的所有开发依赖的安装。

平台安装
---

首先，我们需要记录使用当前版本ionic开发应用的最小依赖安装。ionic以ios和android的构建为目标。我们支持IOS 6+以及Android 4.0+（但是2.3应该可以正常运行）。无论如何，安卓有很多不同的设备，有可能其中的某些设备不能运行。通常，我们十分欢迎您在[github](https://github.com/driftyco/ionic)上门找到bug和改进他们。

你可以在任何你喜欢的操作系统上门开发ionic应用。实际上，ionic曾经在MAC OS X , Linux和Windows都开发过。现在，你将需要使用命令行来完成这下面的教材。你必须有OS X系统来开发和部署iphone apps，所以如果可能的话推荐使用OS X系统。

如果你使用Windows，确认下载并安装[git for Windows](http://git-scm.com/download/win)和[Console2](http://sourceforge.net/projects/console/)(可选).你可以在git bash里面执行本教程的任何命令。

首先，我们将要安装最近版本的[Apache Cordova](http://cordova.apache.org/),它将把我们的应用封装紧一个原生的包装器来使它变成一个传统的原生app。

在安装cordova之前，确认你已经安装了[node.js](http://nodejs.org/).然后执行

>`sudo npm install -g cordova`

sudo是linux系统切换到管理员权限的命令。如果你使用windows，请去掉sudo。根据你想要安装的不同平台，你需要安装一些不同平台的特性工具。跟随cordova平台的android和ios教程来确保你有在这些平台上开发的一切必备的工具。幸亏，你将只需要做一次这些繁琐的开发环境搭建工作。

`linux android 提示`
`如果你使用64位的ubuntu，你将需要安装32位的库文件，因为安卓在目前只是32位的。`
`$sudo apt-get install ia32-libs`，如果你使用Ubuntu 13.04或者更高的版本，'ia32-libs'已经被移除了。你可以用下面的包的来代替`sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0`

`windows平台java ,ant 和 android`
`开发android的windows用户：你将需要确认你以下被安装和设置。`
`NOTE:当你改变PATH或者其他的系统变量，你需要重启或者打开一个新shell来使path的修改生效。`

 `Java JDK`
 `安装最近版本的Java JDK（不是Java JRE）`
 `然后，创建 JAVA_HOME 系统变量指向JDK的安装目录。所以，如果你安装JDK在 C:\Program Files\Java\jdk7 ,设置 JAVA_HOME 指向这个路径。在这之后，添加JDK的bin目录到 PATH 变量。根据刚才的步骤，你可以使用 %JAVA_HOME%\bin 或者完整路径 C:\Program Files\Java\jdk7\bin`

 `Apache Ant`
 `为了安装ant，从[这里](http://ant.apache.org/bindownload.cgi)下载压缩包，解压它，移动第一个文件到一个安全的位置，更新你的 PATH 变量，使其包含 ant的bin 目录。比如，如果你将ant目录移动到C：/，那么你应该把这个路径添加到 PATH 中：C:\apache-ant-1.9.2\bin`

 `Android SDK`
 `安装android sdk也是必需的，sdk可以提供api库，以及构建，测试以及debug应用所需的必要的开发工具。cordova需要设置 ANDROID_HOME的环境变量。这个变量应该指向 [ANDROID_SDK_DIR]\android-sdk 目录(例如：c:\android\android\sdk)`
`下一步，更新你的 PATH ，使其包含 tools/ 和 platforms-tools` 文件夹。使用 ANDROID_HONE， 你可以itianjia %ANDROID_HOME%/tools和 %ANDROID_HOME%/platform-tools


安装ionic
---

ionic配备一个简便的命令行工具来开始，构建和打包ionic应用。
通过下面这个简单的命令来安装ionic
>`sudo npm install -g ionic`

创建工程
---

现在我们需要创建一个新的cordova项目：
>`ionic start todo blank`
它将创建一个叫做`todo`的文件夹。让我们进入这个目录，看看它的文件列表。这是你的ionic工程外部结构，看起来像这样


>`$ cd todo && ls`
>|--  bower.json		// bower 依赖
>|--  config.xml		// cordova 配置文件
>|--  gulpfile.json 	// gulp 任务
>|--  hooks				// 执行具体命令的自定义cordova hook
>|--  ionic.projects 	// ionic配置文件
>|--  package.json	 	// node 依赖
>|--  platforms 		// ios/android项目文件会生成在这里
>|--  plugins 			// cordova 插件
>|--  scss 				// scss 代码，输出到www/css
>|--  www				// 应用 - 包含js代码，库文件，css， 图片等等。


如果你计划使用任何版本控制工具，你可以添加这个新文件夹。

平台配置
---

现在我们开始启用ios/android平台。提示：除非你使用Mac OS，你才能添加iOS平台：
>`ionic platform add ios`
>`ionic platform add android`

测试一下
---

>`$ ionic serve` --使用浏览器调试
>`$ ionic build android`
>`$ ionic emulate android`(使用模拟器)`$ ionic run android`(使用真机)