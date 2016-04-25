---
layout : post
title : 分享一个linux查看连接数的脚本
category : "linux"
tags : [shell]
---

`netstat -nat | awk {print $6} | sort | uniq -c | sort` 

返回结果如下

`
 1 established)
 1 Foreign
 3 CLOSE_WAIT
 7 CLOSING
 14 FIN_WAIT2
 25 LISTEN
 39 LAST_ACK
 609 FIN_WAIT1
 882 ESTABLISHED
 10222 TIME_WAIT 
`

之后可以根据连接数来调整thread数量，如果太多的话，会导致thread大量竞争。

这个命令比较长，可以利用alias 来缩短命令。

`alias cn='netstat -nat | awk '\''{print $6}'\'' | sort | uniq -c | sort '`

cn就是count network的意思啦，也可以根据自己的习惯来命名.