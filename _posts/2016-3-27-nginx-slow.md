--- 
layout : post
title : 线上服务器访问慢的一次故障诊断
date : 2016-03-27 20:06:08 +08:00
tags : "nginx"
---

一次在线上服务器发布更新的时候，发现经过nginx反代之后，访问十分的缓慢，开了一下chrome的调试器，发现访问一个是在访问ico的时候就挂了，然后导致其他的也被阻塞。

![访问超时图.png](https://ooo.0o0.ooo/2016/03/27/56f7d601db8cb.png)

访问ico的时候502了。因为是全局通过nginx 来反代，所以考虑去查看一下nginx的配置文件。

nginx配置文件如下

```
location / {
            proxy_pass http://localhost:8080;
            include proxy.conf;
        }
```

发现`/`目录是指向tomcat 的 8080 端口。于是访问了一下tomcat，果然是没有开启。

然后尝试启动tomcat。
然后得到了一下错误。

```
SEVERE: StandardServer.await: create[8005]: 
java.net.BindException: Cannot assign requested address
        at java.net.PlainSocketImpl.socketBind(Native Method)
        at java.net.AbstractPlainSocketImpl.bind(AbstractPlainSocketImpl.java:353)
        at java.net.ServerSocket.bind(ServerSocket.java:336)
        at java.net.ServerSocket.<init>(ServerSocket.java:202)
        at org.apache.catalina.core.StandardServer.await(StandardServer.java:406)
        at org.apache.catalina.startup.Catalina.await(Catalina.java:676)
        at org.apache.catalina.startup.Catalina.start(Catalina.java:628)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
```

初步怀疑是8005端口被占用，然后查看了一下，也没有，而且反应过来，这里的提示也不是端口被占用。

然后只好去bing了一下，发现都说是hosts的问题。原因在于这个绑定端口的工作是在localhost上执行。。并不是在0.0.0.0或者127.0.0.1上面。

然后去看了一眼`/etc/hosts`文件，果然localhost被指向了`192.168.0.1`这个不存在的ip上，所以绑定自然也就失败了。

修改完hosts，重启一下果然就ok了。这次的排错算是一个比较简单的东西，但是有趣的是，ico加载失败居然会frame的其他部分也被阻塞，也倒是令我觉得十分奇怪。