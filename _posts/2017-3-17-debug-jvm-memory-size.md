---
layout : post
title : 一个排查jvm内存占用的案例
category : "jvm"
tags : [jvm,linux,pmap]
---

同事碰到一个jvm占用内存大量超过xmx的案例，找我帮忙排查为什么内存占用过多。

目标程序是一个java调用jni做图像计算的程序。

参数设置是 xmx6g xms6g xmn4g 使用cmsjvm。

用jstat和jmap查看显示堆占用在2g左右。

top结果，pid24907就是我们要查看的进程

![_config.yml]({{ site.baseurl }}/images/jvm-memory-debug/top.png) 

RES 就是实际占用的内存，占用了 6.9g.

再看看 jstat 和 jps 的数据。

![_config.yml]({{ site.baseurl }}/images/jvm-memory-debug/jps&jstat.png) 

平时会认为jvm占用的内存就是heap+perm，实际上还会有nio direct memory以及jni申请的native memory以及code cache等区域。

首先这个case先怀疑下过多的部分是属于nio的direct memory还是jni申请的native memory。

java本身并没有提供一个直接查看进程申请direct memory的api，正好有撒加写的一个工具可以用来查看。https://gist.github.com/rednaxelafx/1593521

保存到服务器上，编译下就可以用了。

查看结果显示并没有太多的nio direct memory占用。


![_config.yml]({{ site.baseurl }}/images/jvm-memory-debug/directmemorysize.png)

于是把方向转到jni的怀疑上。

使用pmap来查看进程的内存分布。 

这次终于发现了一个占用5g大小的内存块，从而确定问题在jni上。同时还能看到一些带有 nvidia 标签的内存块

![_config.yml]({{ site.baseurl }}/images/jvm-memory-debug/pmap.png)