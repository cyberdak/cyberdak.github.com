---
layout : post
title : es 配置多个 data path 时的分配策略
category : "es"
tags : [es,allocate,disk]
---

es 基于磁盘的可用容量来决定 shard 的分配情况，在哪个节点，在节点的哪块硬盘。

基本的策略称为[Disk-based shard Allocation](https://www.elastic.co/guide/en/elasticsearch/reference/current/disk-allocator.html).但是节目的页面比较简单，所以本文的目的就是详细地探究配置了多个`data.path`时，shard如何分配。


## 2.0 版本

多个`data.path`只能从node级别上统计。比如

```
    node1 :
        - /path1 10gb  (可用空间)
        - /path2 20gb
        - /path3 1tb
    可用空间:1.03tb
    node2 :
        - /path1 500gb
        - /path2 500gb
        - /path3 500gb
    可用空间:1.5tb
```

此时node2有1.5tb可用空间，node1有1.03tb可用空间，shard会被随机分配到node2上面，反而不会分配到最空闲的`node1.path3`上。这种分配策略严重影响了磁盘的利用率。所以在之后的版本采用了每个dish独立计算的方式。

结论：
+ 2.0 在多data path上的使用体验非常痛苦，需要尽快升级到5.x版本。

## 5.x-6.0:


```
    node1 :
        - /path1 10g  
        - /path2 20g
        - /path3 1tb
    
    node2 :
        - /path1 500g
        - /path2 500g
        - /path3 500g
```

因为策略升级到每个`disk`单独统计，现在的情况变成了所有的新shard都会被分配到`/path3`，写入压力全部集中到`/path3`上，从而导致`/path3`io压力过大，iowait过大，极端情况会导致linux宕机.同时也很容易导致node1因为压力大而进入GC死亡螺旋（解释见文章结尾。）。5.x版本需要在index上设置``

结论：
1. 5.x-6.0版本需要需要搭配`index.routing.allocation.total_shards_per_node: "2"`来保证shard在不同节点之间分配；依然会出现shard都堆在一个path上，导致单盘io过高的问题。
2. 可以在index创建之前主动设置集群`rebalance`为`true`,帮忙集群同步shard。建议定期运行该策略来平衡shard分布。

## 6.1

6.1 增加了一个关键的特性[#26654](https://github.com/elastic/elasticsearch/pull/26654),新特性使得index的多个shard可以在不用的`data path`上平衡.

假设现在有10shard需要分配:

6.1 之前的效果

+ /path1 10 shards
+ /path2 0 shard
+ /path3 0 sahrd

6.1 之后的效果

+ /path1 4 shards
+ /path2 3 shards
+ /path3 3 sahrds

结论:

+ 6.1之后的版本有比较的shard平衡能力，依然建议在index上设置`index.routing.allocation.total_shards_per_node: "2"`.


一些其他有用的资料

### gc死亡螺旋

gc死亡螺旋的概念来源《Google SRE》一书中，第22章。
es这种的java服务往往面临着大量的对象创建和回收，gc会大幅度影响es集群的运行状况。使用forcemerge可以大幅度降低data node的内存占用，降低服务器的load。

```
假设如下场景：
1. 某JAVA前端服务器GC参数没有被调优
2. 在高负载（但是在期待范围内）情况下，情断由于GC问题导致CPI不足
3. CPU不足导致请求处理变慢
4. 同时处理的请求增多导致内存使用上升。
5. 内存压力上升，同时由于固定内存分配比例的原因，用于缓存的内存数量减少
6. 缓存数量降低意味着缓存中键值数量下降，从而导致命中率下降
7. 缓存命中率下降导致更多的请求被发往后端进行处理。
8. 后端服务器CPU或者线程不足。
9. CPU不足导致健康检查失败，从而触发了连锁故障。
```

## 参考资料

+ [由于集群硬盘大小不一致导致的分配不均匀问题](
https://blog.csdn.net/cdpac/article/details/78623308)
+ [5.3.0 中关于多data path的bug](https://www.elastic.co/blog/multi-data-path-bug-in-elasticsearch-5-3-0)
+ [根据shard大小平衡es集群](http://engineering.simplymeasured.com/dev-blog/2015/07/08/balancing-elasticsearch-cluster-by-shard-size.html)