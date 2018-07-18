---
layout : post
title : es 在磁盘故障时的故障表现以及排查方式
category : "es"
tags : [es,disk,crash]
---

在续[es 分配策略解读](http://cyberdak.github.io/2018-7-8-es-disk-allocate-strategy.md) 之后，笔者将es集群从5.4.2升级到5.6.8，然后将多盘的机器开始配置RAID 0，从而避免多个data path的数据分布不均匀。

在某一个节点由运维同学完成RAID 0 配置，开始加入集群做 rebalance 的时候，另外一个节点突然down掉。这时后台正在做一个forcemerge，查看了jvm crash的日志，同样显示是merge thread上导致的crash。

crash logs : 

```
---------------  T H R E A D  ---------------

Current thread (0x00007fc63819e000):  JavaThread "elasticsearch[xnode-18][[strategy-101-20180713][1]: Lucene Merge Thread #0]" daemon [_thread_in_Java, id=21778, stack(0x00007fb3896dd000,0x00007fb3897de000)]

siginfo: si_signo: 7 (SIGBUS), si_code: 2 (BUS_ADRERR), si_addr: 0x00007e4f65d5f000

Registers:
RAX=0x00007e4f65d5f400, RBX=0x0000000000000000, RCX=0x00007fd147499920, RDX=0xffffffffffffff88
RSP=0x00007fb3897dc340, RBP=0x00007fb3897dc340, RSI=0x00007fd147499528, RDI=0x00007e4f65d5f3f8
R8 =0x00000000000751f3, R9 =0x000000000006f1f3, R10=0x00007fd2585e3700, R11=0x00007e4f65d5f000
R12=0x00007fc9d7fff000, R13=0x00007fd147498ee0, R14=0x00007e4f65d5f400, R15=0x00007fc63819e000
RIP=0x00007fd2585e2a50, EFLAGS=0x0000000000010286, CSGSFS=0x0000000000000033, ERR=0x0000000000000004
  TRAPNO=0x000000000000000e

Top of Stack: (sp=0x00007fb3897dc340)
0x00007fb3897dc340:   0000000000000410 00007fd25a78e431
0x00007fb3897dc350:   00007fd147498ea0 00007fd147499518
0x00007fb3897dc360:   0000600000000400 00007fcb25344c80
0x00007fb3897dc370:   00007fd147498e48 00007f89df740000
0x00007fb3897dc380:   00007fd100000000 00007fcb25344cc0
0x00007fb3897dc390:   001be0d00ca6a55a 00007fd0a40af0d0
0x00007fb3897dc3a0:   00007fd1474992e8 00007fd259886f7e
0x00007fb3897dc3b0:   00007fd100000057 00007fd147498ea0
0x00007fb3897dc3c0:   0000000000000000 00007fd25ca74428
0x00007fb3897dc3d0:   00007fd147498f38 00000000000751e3
0x00007fb3897dc3e0:   0000000000006000 ede934a300000400
0x00007fb3897dc3f0:   00007fd147498f58 00007fd1000000e6
0x00007fb3897dc400:   00007fd147494fd8 00007fd147498458
0x00007fb3897dc410:   00007fd147498358 0000000000000000
0x00007fb3897dc420:   00007fd147498ea0 00007fd25c96b2f8
0x00007fb3897dc430:   00007fcb25344c80 00007fd147498f38
0x00007fb3897dc440:   0000000000000000 00007fca22e80000
0x00007fb3897dc450:   0000000000000001 00007fd147498300
0x00007fb3897dc460:   0000000000000000 00007fcb2b6e92f8
0x00007fb3897dc470:   0000000024a1509d 00007fd25b53e724
0x00007fb3897dc480:   0074b70000000000 00007fd147498358
0x00007fb3897dc490:   0000000000000000 00007fd147494e38
0x00007fb3897dc4a0:   00007fca3d011b00 d91ac4ca00000000
0x00007fb3897dc4b0:   00007fd147494d80 00007fd0a6000670
0x00007fb3897dc4c0:   000000010000000d 00007fd0a0d61650
0x00007fb3897dc4d0:   0000000a00000a5a 00007fcb2b6f6340
0x00007fb3897dc4e0:   00007fd147494d80 00007fd259617a6c
0x00007fb3897dc4f0:   00007fd0a6000780 00007fd147494d80
0x00007fb3897dc500:   00007fd147494e00 00007fd147494e38
0x00007fb3897dc510:   00007fd0a673f698 00007fca3d011ea0
0x00007fb3897dc520:   00007f89df740000 0000000000000007
0x00007fb3897dc530:   0000000000000007 00007f89df740000 

Instructions: (pc=0x00007fd2585e2a50)
0x00007fd2585e2a30:   da e9 32 00 00 00 48 8b 44 d7 08 48 89 44 d1 08
0x00007fd2585e2a40:   48 ff c2 75 f1 48 33 c0 c9 c3 66 0f 1f 44 00 00
0x00007fd2585e2a50:   c5 fe 6f 44 d7 c8 c5 fe 7f 44 d1 c8 c5 fe 6f 4c
0x00007fd2585e2a60:   d7 e8 c5 fe 7f 4c d1 e8 48 83 c2 08 7e e2 48 83 

Register to memory mapping:

RAX=0x00007e4f65d5f400 is an unknown value
RBX=0x0000000000000000 is an unknown value
RCX=0x00007fd147499920 is pointing into object: 0x00007fd147499518
[B 
 - klass: {type array byte}
 - length: 1024
RDX=0xffffffffffffff88 is an unknown value
RSP=0x00007fb3897dc340 is pointing into the stack for thread: 0x00007fc63819e000
RBP=0x00007fb3897dc340 is pointing into the stack for thread: 0x00007fc63819e000
RSI=0x00007fd147499528 is pointing into object: 0x00007fd147499518
[B 
 - klass: {type array byte}
 - length: 1024
RDI=0x00007e4f65d5f3f8 is an unknown value
R8 =0x00000000000751f3 is an unknown value
R9 =0x000000000006f1f3 is an unknown value
R10=0x00007fd2585e3700 is at begin+0 in a stub
StubRoutines::unsafe_arraycopy [0x00007fd2585e3700, 0x00007fd2585e373b[ (59 bytes)
R11=0x00007e4f65d5f000 is an unknown value
R12=0x00007fc9d7fff000 is an unknown value
R13=0x00007fd147498ee0 is an oop
java.nio.DirectByteBufferR 
 - klass: 'java/nio/DirectByteBufferR'
R14=0x00007e4f65d5f400 is an unknown value
R15=0x00007fc63819e000 is a thread


Stack: [0x00007fb3896dd000,0x00007fb3897de000],  sp=0x00007fb3897dc340,  free space=1020k
Native frames: (J=compiled Java code, j=interpreted, Vv=VM code, C=native code)
v  ~StubRoutines::jlong_disjoint_arraycopy
J 15271 C2 org.apache.lucene.store.ByteBufferIndexInput.readBytes([BII)V (189 bytes) @ 0x00007fd25a78e431 [0x00007fd25a78e1c0+0x271]
J 27711 C2 org.apache.lucene.store.ChecksumIndexInput.seek(J)V (72 bytes) @ 0x00007fd25ca74428 [0x00007fd25ca74140+0x2e8]
J 27715 C2 org.apache.lucene.codecs.CodecUtil.checksumEntireFile(Lorg/apache/lucene/store/IndexInput;)J (114 bytes) @ 0x00007fd25c96b2f8 [0x00007fd25c96b1e0+0x118]
J 26742 C2 org.apache.lucene.codecs.lucene60.Lucene60PointsWriter.merge(Lorg/apache/lucene/index/MergeState;)V (615 bytes) @ 0x00007fd25b53e724 [0x00007fd25b53e100+0x624]
J 29495 C2 org.apache.lucene.index.SegmentMerger.merge()Lorg/apache/lucene/index/MergeState; (882 bytes) @ 0x00007fd25cfb3c80 [0x00007fd25cfb0ac0+0x31c0]
J 27600 C2 org.apache.lucene.index.IndexWriter.mergeMiddle(Lorg/apache/lucene/index/MergePolicy$OneMerge;Lorg/apache/lucene/index/MergePolicy;)I (2152 bytes) @ 0x00007fd25bb0bb4c [0x00007fd25bb09720+0x242c]
J 31072 C1 org.apache.lucene.index.IndexWriter.merge(Lorg/apache/lucene/index/MergePolicy$OneMerge;)V (411 bytes) @ 0x00007fd25d403454 [0x00007fd25d402620+0xe34]
J 29879 C2 org.elasticsearch.index.engine.ElasticsearchConcurrentMergeScheduler.doMerge(Lorg/apache/lucene/index/IndexWriter;Lorg/apache/lucene/index/MergePolicy$OneMerge;)V (777 bytes) @ 0x00007fd25d11a444 [0x00007fd25d119a80+0x9c4]
J 29544 C2 org.apache.lucene.index.ConcurrentMergeScheduler$MergeThread.run()V (252 bytes) @ 0x00007fd25cca5138 [0x00007fd25cca4f00+0x238]
v  ~StubRoutines::call_stub
V  [libjvm.so+0x68bc46]  JavaCalls::call_helper(JavaValue*, methodHandle*, JavaCallArguments*, Thread*)+0x1056
V  [libjvm.so+0x68c151]  JavaCalls::call_virtual(JavaValue*, KlassHandle, Symbol*, Symbol*, JavaCallArguments*, Thread*)+0x321
V  [libjvm.so+0x68c5f7]  JavaCalls::call_virtual(JavaValue*, Handle, KlassHandle, Symbol*, Symbol*, Thread*)+0x47
V  [libjvm.so+0x7233b0]  thread_entry(JavaThread*, Thread*)+0xa0
V  [libjvm.so+0xa68fff]  JavaThread::thread_main_inner()+0xdf
V  [libjvm.so+0xa6912c]  JavaThread::run()+0x11c
V  [libjvm.so+0x91cc68]  java_start(Thread*)+0x108
C  [libpthread.so.0+0x7dc5]  start_thread+0xc5

```


开始怀疑是因为HDD导致的压力问题，merge 压力过大，所以直接启动了该节点开始恢复。

集群从红色变回黄色。

但是过了几分钟，集群再次变回红色，一看 es 进程小时了，而且`logs`文件中没有异常数据，同时文件夹中也没有jvm crash 文件生成。怀疑是 uncatchError 。使用非后台运行方式启动es:`bin/elasticsearch`.(后台启动方式是`bin/elasticsearch -d`,这样会忽略java console，导致不能捕获到没有经过catch的异常错误信息)

此时es挂掉的错误信息为

```
[2018-07-18T19:55:16,294][ERROR][o.e.b.ElasticsearchUncaughtExceptionHandler] [xnode-18] fatal error in thread [elasticsearch[xnode-18][generic][T#12]], exiting
java.lang.InternalError: a fault occurred in a recent unsafe memory access operation in compiled Java code
at org.apache.lucene.codecs.CodecUtil.checkHeader(CodecUtil.java:195) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.apache.lucene.util.bkd.BKDReader.(BKDReader.java:62) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.apache.lucene.codecs.lucene60.Lucene60PointsReader.(Lucene60PointsReader.java:105) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:55:25]
at org.apache.lucene.codecs.lucene60.Lucene60PointsFormat.fieldsReader(Lucene60PointsFormat.java:108) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:55:25]
at org.apache.lucene.index.SegmentCoreReaders.(SegmentCoreReaders.java:134) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.apache.lucene.index.SegmentReader.(SegmentReader.java:74) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.apache.lucene.index.ReadersAndUpdates.getReader(ReadersAndUpdates.java:145) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.apache.lucene.index.ReadersAndUpdates.getReadOnlyClone(ReadersAndUpdates.java:197) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.apache.lucene.index.StandardDirectoryReader.open(StandardDirectoryReader.java:103) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.apache.lucene.index.IndexWriter.getReader(IndexWriter.java:467) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.apache.lucene.index.DirectoryReader.open(DirectoryReader.java:103) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.apache.lucene.index.DirectoryReader.open(DirectoryReader.java:79) ~[lucene-core-6.6.1.jar:6.6.1 9aa465a89b64ff2dabe7b4d50c472de32c298683 - varunthacker - 2017-08-29 21:54:39]
at org.elasticsearch.index.engine.InternalEngine.createSearcherManager(InternalEngine.java:329) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.index.engine.InternalEngine.(InternalEngine.java:175) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.index.engine.InternalEngineFactory.newReadWriteEngine(InternalEngineFactory.java:25) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.index.shard.IndexShard.newEngine(IndexShard.java:1602) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.index.shard.IndexShard.createNewEngine(IndexShard.java:1584) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.index.shard.IndexShard.internalPerformTranslogRecovery(IndexShard.java:1027) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.index.shard.IndexShard.skipTranslogRecovery(IndexShard.java:1048) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.indices.recovery.RecoveryTarget.prepareForTranslogOperations(RecoveryTarget.java:360) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.indices.recovery.PeerRecoveryTargetService$PrepareForTranslogOperationsRequestHandler.messageReceived(PeerRecoveryTargetService.java:330) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.indices.recovery.PeerRecoveryTargetService$PrepareForTranslogOperationsRequestHandler.messageReceived(PeerRecoveryTargetService.java:324) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.transport.TransportRequestHandler.messageReceived(TransportRequestHandler.java:33) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.transport.RequestHandlerRegistry.processMessageReceived(RequestHandlerRegistry.java:69) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.transport.TcpTransport$RequestHandler.doRun(TcpTransport.java:1556) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingAbstractRunnable.doRun(ThreadContext.java:674) ~[elasticsearch-5.6.8.jar:5.6.8]
at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:37) ~[elasticsearch-5.6.8.jar:5.6.8]
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) ~[?:1.8.0_66]
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) ~[?:1.8.0_66]
at java.lang.Thread.run(Thread.java:745) [?:1.8.0_66]
```

报错的节点依然是lucene，此时怀疑是lucene数据在上次jvm crash的时候已经损害，导致es读取到相应的错误数据crash。

首先去了es的github上搜索了一下carsh的issue，没有相关的信息。大多数在怀疑硬盘、内存由问题。

自查了一下，磁盘和内存都有大量可用值。

接着提了自己的issue，提供了相应的jvm crash 日志和 es crash日志。

此时，怀疑是否是因为硬盘损坏的缘故，因为是多盘系统。当前的data path 设置如下：

```
path:
  logs: /data/eslog
  data:
    - /data/es
    - /data1/es
    - /data2/es
    - /data3/es
    - /data4/es
    - /data5/es
    - /data6/es
    - /data7/es
    - /data8/es
    - /data9/es
    - /data10/es
```

用二分法快速注释掉相应的 path，排查出来是  /data4 导致的问题。

此时查看系统日志 `dmesg`,发现相应的错误日志:

```
[6663179.508007] sd 0:0:5:0: [sdf] FAILED Result: hostbyte=DID_OK driverbyte=DRIVER_SENSE
[6663179.508018] sd 0:0:5:0: [sdf] Sense Key : Medium Error [current] [descriptor] 
[6663179.508022] sd 0:0:5:0: [sdf] Add. Sense: Unrecovered read error
[6663179.508025] sd 0:0:5:0: [sdf] CDB: Read(16) 88 00 00 00 00 02 80 02 a2 08 00 00 01 00 00 00
[6663179.508028] blk_update_request: critical medium error, dev sdf, sector 10737590840
[6663182.792073] mpt2sas0: log_info(0x31080000): originator(PL), code(0x08), sub_code(0x0000)
[6663182.792091] sd 0:0:5:0: [sdf] FAILED Result: hostbyte=DID_OK driverbyte=DRIVER_SENSE
[6663182.792099] sd 0:0:5:0: [sdf] Sense Key : Medium Error [current] [descriptor] 
[6663182.792103] sd 0:0:5:0: [sdf] Add. Sense: Unrecovered read error
[6663182.792107] sd 0:0:5:0: [sdf] CDB: Read(16) 88 00 00 00 00 02 80 02 a2 38 00 00 00 08 00 00
[6663182.792110] blk_update_request: critical medium error, dev sdf, sector 10737590840
[6664570.572432] mpt2sas0: log_info(0x31080000): originator(PL), code(0x08), sub_code(0x0000)
[6664570.572438] mpt2sas0: log_info(0x31080000): originator(PL), code(0x08), sub_code(0x0000)
[6664570.572444] mpt2sas0: log_info(0x31080000): originator(PL), code(0x08), sub_code(0x0000)
[6664570.572452] mpt2sas0: log_info(0x31080000): originator(PL), code(0x08), sub_code(0x0000)
[6664570.572470] sd 0:0:5:0: [sdf] FAILED Result: hostbyte=DID_OK driverbyte=DRIVER_SENSE
[6664570.572476] sd 0:0:5:0: [sdf] Sense Key : Medium Error [current] [descriptor] 
[6664570.572479] sd 0:0:5:0: [sdf] Add. Sense: Unrecovered read error
[6664570.572482] sd 0:0:5:0: [sdf] CDB: Read(16) 88 00 00 00 00 02 80 02 a2 08 00 00 01 00 00 00
```

之后在`path.data`中注释掉`/data4`，es成功稳定运行，开始在集群中恢复数据。


相关链接:
[error-javalanginternalerror-a-fault-occurred-in-kafka](https://community.hortonworks.com/content/supportkb/199827/error-javalanginternalerror-a-fault-occurred-in-a.html)
[crash on lucene merge](https://github.com/elastic/elasticsearch/issues/32163)