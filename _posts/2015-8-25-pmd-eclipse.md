---
layout : post
title : 使用PMD来分析代码质量
category : "java"
tags : ['pmd','']
---


PMD是一个用来分析代码缺陷,code style的工具。同时还附带了一个CPD工具，用来检测重复的代码。

通常在项目中大量的复制粘贴代码，极有可能会导致一些bug以及这些拥有共性的代码往往可以被抽取出来做一个新的方法。

所以这里我们借助CPD来帮我们找出那些重复的代码。

PMD的项目管理似乎比较混乱。

我在pmd站点找到了两个eclipse的插件。一个安装之后编码错误不能用，一个安装之后直接报错。

这里说一下解决的问题的思路。

1.第一个安装的坑：通过搜索pmd，找到的文章里面的提到的eclipse安装地址都有`http://pmd.sf.net/eclipse`\`http://pmd.sourceforge.net/eclipse/`，然而都不能用。

只好去google 了一下pmd，找到了github io上面的站点。

找到了一个新的连接

>`http://sourceforge.net/projects/pmd/files/pmd-eclipse/update-site/`

谢天谢地，这个可以用，不过sf大陆连接的速度感人，只好开了VPN然后安装。

见过漫长的安装之后，终于安装完成了。

2.第二个坑

之后启动eclipse，项目上门右键，出现了PMD的菜单，点击“Find Suspect Cut and Copy”，然后选择一下语言，还有重复的行数，以及输出报告的格式，然后就可以检查重复了。

但是..点完就直接报错了。。

报错的信息大概是直接把汉字报出来了。把异常google了一下，发现也有人碰到这个问题。

问题描述如下
link:http://sourceforge.net/p/pmd-eclipse/discussion/457016/thread/c2476d3b/
bugs link:https://sourceforge.net/p/pmd-eclipse/bugs/20/

`refer to https://sourceforge.net/p/pmd-eclipse/bugs/20/
An internal error occurred during: "DetectCutAndPaste".
Exception Stack Trace : net.sourceforge.pmd.ast.TokenMgrError: Lexical
error at line XXX, column YYY. Encountered: "\r" (13), after :ZZZ
I found it's because of the encoding of *.java it not the same as default
encoding base upon locale.
for example,my locale is zh_CN,so the default encoding is GBK,but my java
file encoding is UTF-8,so an error occured.if my java file encoding is
GBK,the CPD runs rightly.
but there is no setting in the perferrence of eclipse to tell PMD "what's
my file encoding is".
the only way to use CPD is click the "Lunch CPD..." button in the eclipse
menu 'window--perference--PMD--CPD preference'.and in the dialog input
"UTF-8" in the "file encoding(defaults base upon locale)"`

文中大致提到是因为编码的问题无法启动，java文件是UTF-8时报错，GBK时正常。
唯一能够正常使用CPD的途径是'window--perference--PMD--CPD preference'，然后点击'Lunch CPD...'按钮。
之后出现的一个SWT界面，这个界面可以选择编码，把编码改成UTF-8就可以正常使用了。但是这个办法启动还是比较麻烦，而且SWT界面太丑。

既然它没有提供修改编码的地方，那就动手给他加上一个修改编码的地方不就完了。

然后开始动手，把插件的jar包解压了，开始搜索相应的关键字。

搜索了一下GBK没结果，想起来jar都是封装的class文件。肯定不行了

这时候只能打开google找一下这个插件的源码

万幸代码托管在github:`https://github.com/pmd/pmd-eclipse-plugin`

之后把代码下载下来，导入到eclipse，搜索了一下菜单的关键字“Find Suspect Cut and Copy”,得到得到preporties中的key=`action.checkcpd`,搜索这个key，得到plugin.xml，应该是一个类似android布局文件的一样的东西，里面定义了每个菜单的具体属性。

这里的菜单明显就是触发`net.sourceforge.pmd.eclipse.ui.actions.CPDCheckProjectAction`这个文件，然后找一下这个文件。

把代码一行一行看完，发现是通过`DetectCutAndPasteCmd`这个类来执行实际的命令。

然后在这个类里面发现了元凶

`    private CPD newCPD() {
        CPDConfiguration config = new CPDConfiguration();
        config.setMinimumTileSize(minTileSize);
        config.setLanguage(language);
        config.setEncoding(System.getProperty("file.encoding"));
        return new CPD(config);
    }
`

这里的编码是通过系统变量file.encoding来获取的，然而windows平台下面，在修改了工作区间编码为UTF-8的eclipse里面运行如下代码
`		public static void main(String[] args) {
			System.out.println(System.getProperty("file.encoding"));
		}
`
会得到`utf-8`

但是，如果直接通过 `java xxx` 来运行的话，结果居然是GBK。

所以现在问题明显了，只要修改了运行eclipse的jvm的参数，把file.encodind修改成UTF-8就ok了

在eclipse.ini加入下列语句

`-Dfile.encoding=utf-8`

然后CPD就可以正常跑起来了