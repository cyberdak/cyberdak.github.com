---
layout : post
title : 一个排查jvm内存占用的案例
category : "jvm"
tags : [jvm,linux,pmap]
---

同事碰到一个jvm占用内存大量超过xmx的案例，找我帮忙排查为什么内存占用过多。

目标程序是一个java调用jni做图像计算的程序。

参数设置是 xmx6g xms6g xmn4g 使用cmsgc。

用jstat和jmap查看显示堆占用在2g左右。