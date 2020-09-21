---
layout : post
title : prometheus 2.21.0 版本新特性
category : "prometheus"
tags : [prometheus]
---


prometheus 目前发展的趋势非常快，基本保持一个月发版一次的节奏，[2.21.0](https://github.com/prometheus/prometheus/releases/tag/v2.21.0)版本在9月18号发布,该版本基于[2.20.0](https://github.com/prometheus/prometheus/releases/tag/v2.20.0)，带来一个新特性和bug修复。

+ 现在可以在promQL、配置文件和命令行中使用 `1h30m` 这种的混合时间表达式了。

+ 两个新的服务发现: `Eureka` 和 `Hetzner`。 Kubernetes 服务发现现在支持EndpointSlices

+ tsdb 命令行工具目前称为promtool的一部分。


更多的[changelog](https://github.com/prometheus/prometheus/releases/tag/v2.21.0)
