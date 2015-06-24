---
layout : post
title : npm 切换源的工具 nrm
category : "nodejs"
tags : [nodejs,npm]
---

npm 默认的源`register.npmjs.org`因为墙的问题，经常访问不到/速度太慢，install一次就花个几十分钟，实在受不了。

npm可以通过命令行来手动切换源，但是手动切换很麻烦，而且说不一定那天又开始抽风。所以可以使用工具`nrm`来管理npm源。

>npm install -g nrm

使用`nrm ls`查看npm源列表 

```bash
$ nrm ls
  npm ---- https://registry.npmjs.org/
* cnpm --- http://r.cnpmjs.org/
  taobao - http://registry.npm.taobao.org/
  eu ----- http://registry.npmjs.eu/
  au ----- http://registry.npmjs.org.au/
  sl ----- http://npm.strongloop.com/
  nj ----- https://registry.nodejitsu.com/
  pt ----- http://registry.npmjs.pt/
```

使用`nrm test`查看源服务器的响应速度
```bash
  npm ---- https://registry.npmjs.org/
* cnpm --- http://r.cnpmjs.org/
  taobao - http://registry.npm.taobao.org/
  eu ----- http://registry.npmjs.eu/
  au ----- http://registry.npmjs.org.au/
  sl ----- http://npm.strongloop.com/
  nj ----- https://registry.nodejitsu.com/
  pt ----- http://registry.npmjs.pt/
```

使用`nrm use taobao`来切换源服务器。


>`$ nrm use cnpm`


ps：切换到taobao npm库，安装某些库依然会安装不上，把dns server从8.8.8.8切换到114.114.114.114不知道会不会改善。