---
layout : post
title : jvm ygc越来越慢排查
category : "jvm"
tags : [jvm,ygc]
---

之前猎人因为调用的服务过多，而某些应用超时的情况下，无法获取到业务所需要的特征，导致业务逻辑产生错误的结果，专门封闭解决超时，包括优化服务，增加缓存优化避免过多的服务调用。项目完成时，成功将每天的超时量降低到了1000。

然后出现了一个很奇怪的现象来了，当程序运行了一段时间以后，超时量就开始逐渐变多。表现就是今天超时500，明天600，后天700，这样慢慢往上涨，这种随jvm启动而导致程序rt升高以及吞吐量降低的问题，第一时间考虑到了是GC影响的问题。

因为YGC要扫描所有的gc roots，而gc roots其实包含了很多坑。比如old gen，string table，classloader。

正巧之前淘宝的bluedavy也排查过一些ygc越来越慢的问题，那个case是因为XStream的构造函数有bug，导致会不断新建classloader导致的。在排查一番以后，我们的程序中并没有使用XStream。参考case : http://hellojava.info/?p=441  http://hellojava.info/?p=454

StringTable的问题也一样，某些场景不合理使用`String.intern()`也会导致YGC越来越长。case 看 这里.https://mp.weixin.qq.com/s/z2d4w9he88qG4pa8jSYqmg

ygc的问题其实是最难排查的，因为 parNew ygc的日志量太少了，就一行语句，不会打印各个阶段的详情。所以为了能够查看到底是ygc的哪个阶段出了问题，使用jdk8u40以后版本的话，直接切换到G1GC，G1GC的日志十分丰富。

切换到G1GC以后，可以明显从日志上看到 Pre 耗时比较长，因此把并行处理开关设置为true，观察了一段时间，发现问题依旧。

再次在参数上加上 print Proc ，这次可以看到整个GC里面耗时最长的部分在`WeakReference`上。

![微信图片_20190417001136.png](https://i.loli.net/2019/04/17/5cb5fee4e4ca5.png)

放狗搜索一番，得到以下链接。。

http://k.sina.com.cn/article_1708729084_65d922fc034002aen.html

确认问题，是在JDK1.8中使用js导致的问题。

![fd8b-fyhneam3577013.jpg](https://i.loli.net/2019/04/17/5cb5ff7368f1d.jpg)


同时搜索一下 bugs.openjdk.com 看看这个bug的情况

https://bugs.openjdk.java.net/browse/JDK-8177098

![微信图片_20190417002509.png](https://i.loli.net/2019/04/17/5cb602005fea9.png)

同时也给了一个workaround，每次eval的时候，都用一个新的实例。

```
import jdk.nashorn.api.scripting.NashornScriptEngineFactory; 
import javax.script.ScriptEngine; 
public class ReproduceBug { 
   public static void main(String[] args) throws Exception { 
      ScriptEngine engine = new NashornScriptEngineFactory().getScriptEngine(); 
      while (true) engine.eval("var x = 5;"); 
   } 
} 
```


修复方式
```
import jdk.nashorn.api.scripting.NashornScriptEngineFactory; 
import javax.script.ScriptEngine; 
public class Workaround { 
   public static void main(String[] args) throws Exception { 
      while (true) { 
         ScriptEngine engine = new NashornScriptEngineFactory().getScriptEngine(); 
         engine.eval("var x = 5;"); 
      } 
   } 
} 
```


附上gceasy的分析报告

https://gceasy.io/my-gc-report.jsp?p=c2hhcmVkLzIwMTkvMDQvMTYvLS1jaGVjay0xMzQubG9nLjMtLTE2LTQ2LTUy&channel=WEB

上面说
![微信图片_20190417004939.png](https://i.loli.net/2019/04/17/5cb607b6acf62.png)
，显然他们也没有处理过这种case。否则可以根据日志中`WeakReference`耗时不断上涨来给出诊断意见。 

这里总结一下几种会导致ygv越来越长的case。

+ Xstream 使用无参构造器，每次使用都会创建一个classloader，导致GC ROOTs变多。YGC扫描变慢。相关链接:http://hellojava.info/?p=441  http://hellojava.info/?p=454
+ jackson序列化json时，会对key使用String.intern()来降低对内存的占用，这本来是个优化，但是当使用场景的key是可变的时候，就会导致StringTable 越变越大.
+ jdk 8 ,9 重用`ScriptEngine`类，不断调用eval方法。
