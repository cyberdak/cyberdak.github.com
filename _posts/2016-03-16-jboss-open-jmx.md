---
layout : post
title : jboss-4.2.3开启jxm监控
---


准备给老系统的jboss加上jvm监控，查看内存泄漏和gc次数，方便优秀。
因为项目以前跑了很久了。

而且由于是老项目，用的还是ejb+jboss-4.2.3+jdk6..

```
vim ${JBOSS_HOME}/bin/run.conf
```

在文件最后，加上如下语句

```
JAVA_OPTS="-Djava.rmi.server.hostname=ip(填一个服务器的ip) $JAVA_OPTS"
JAVA_OPTS="-Dcom.sun.management.jmxremote $JAVA_OPTS"
JAVA_OPTS="-Dcom.sun.management.jmxremote.port=12345(自定义端口) $JAVA_OPTS"
JAVA_OPTS="-Dcom.sun.management.jmxremote.authenticate=false $JAVA_OPTS"
JAVA_OPTS="-Dcom.sun.management.jmxremote.ssl=false $JAVA_OPTS"

JAVA_OPTS="$JAVA_OPTS -Djboss.platform.mbeanserver"
JAVA_OPTS="$JAVA_OPTS -Djavax.management.builder.initial=org.jboss.system.server.jmx.MBeanServerBuilderImpl"
```

然后重启就可以在jmc里面访问到了

但是5.4的jmc提示我6还不支持某些新方法。得更新到7U4以后才能完全支持，猜测是因为7u4更新了完整的G1的缘故。