---
layout: post
title: tomcat开启jmx监控以及飞行记录器
description: Some Description
date: 2016-03-25 23:30:41 +08:00
tags: "java tomcat jmx"
---

之前把现在服务器里面的jboss都开启了jmx了，然后开始着手准备把剩下的tomcat也都开jmx。

现在公司用的基本都是tomcat7+tomcat8 / jdk 7 + jdk8。
在jmc里面还看到一个新玩意，叫飞行记录器。googe了一下，发现在7u4以后，jdk出了一个新的功能叫飞行记录器，在jmc（java mission control）中提供。

所以这次就顺手把jmx和飞行记录器都打开。

查看了一下tomcat的文档，发现其实tomcat提供了默认环境变量的默认文件`setenv.bat`(在*nix上面则是setenv.sh),相应的代码如下:

```
rem Get standard environment variables
if not exist "%CATALINA_BASE%\bin\setenv.bat" goto checkSetenvHome
call "%CATALINA_BASE%\bin\setenv.bat"
goto setenvDone
:checkSetenvHome
if exist "%CATALINA_HOME%\bin\setenv.bat" call "%CATALINA_HOME%\bin\setenv.bat"
:setenvDone
```

基本就是检测bin目录下是否存在setenv.bat，如果存在就执行。

使用setenv.bat 的好处在于能够避免侵入性，不修改原来的代码，毕竟如果直接修改catalina.bat的话，还需要在execute之前修改，对于公司的新手来说，可能会导致将配置写在错误的地方，导致配置不生效。

在bin目录下新建`setenv.bat`

在其中填入一下内容
```
set CATALINA_OPTS=%CATALINA_OPTS% -Dcom.sun.management.jmxremote
set CATALINA_OPTS=%CATALINA_OPTS% -Dcom.sun.management.jmxremote.port=6666
set CATALINA_OPTS=%CATALINA_OPTS% -Dcom.sun.management.jmxremote.ssl=false
set CATALINA_OPTS=%CATALINA_OPTS% -Dcom.sun.management.jmxremote.authenticate=false
set JAVA_OPTS=%JAVA_OPTS% -XX:+UnlockCommercialFeatures -XX:+FlightRecorder
exit /b 0
```

完事重启一下就ok了，tomcat配置jmx比jboss4.2.3少了一个hostname。还是比较简单的。
然后打开jmc就可以连接jmx和飞行记录器了。