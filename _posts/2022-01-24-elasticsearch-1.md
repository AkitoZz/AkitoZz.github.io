---
layout: post
title: elasticseach杂谈-1
categories: [efk, elasticsearch]
description: elasticsearch杂谈
keywords: 内存相关, heap, es7.5
---

---
提纲

es的内存为什么重要
es的内存申请主要用作于
不同的内存使用如何查看
熔断器
---


由于最近工作内容一直在使用elasticsearch，并且遇到了不少问题，本文就es所使用的内存情况，做一个分析

环境说明
    - es使用的7.5.1版本
    - 运行在k8s环境
    - 内存分配4G，heap设置2G

#### ES内存的重要性
    对于elasticsearch来说，从大到小的概念为cluster node index segment shard doc，也就是集群 节点 索引 段 分片 文档，elasticsearch是一个搜索存储引擎，实际存储的数据在是在doc这个单位上，但是将搜索条件映射到doc却并非关系型数据库般对应列值是否满足条件这么简单，也并非是内存完全的内存存储，es通过分词来实现搜索，通过缓存来提高查询速度，这也是为什么es虽然并非内存存储引擎，然而内存对齐十分重要的原因。
    
    lucene倒排索引 词典 doc segment，这几个东西可以说是es的核心，lucene倒排索引是es最核心的技术，但是笔者了解并不够深入，简单地将其理解为一个特殊的hash函数，词典通过这个hash函数能映射到对应的文档列表，这就是查询文档的逻辑，而词典是es的分词器去将源数据分析得到的，segment是一个完整的倒排索引，索引是由shard分片存储，而shard中主要就是存储segment数据，由于segment


    



内存分类
    - 堆外内存
    - 堆内存

