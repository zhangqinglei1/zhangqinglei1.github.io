---
layout: post
title:  "Spark消费kafka"
date:   2020-10-10 18:14:54
categories: 大数据
tags: 大数据
excerpt: Spark消费提交kafka
mathjax: true
---

* content
{:toc}


## 地址

http://spark.apache.org/docs/latest/streaming-kafka-0-10-integration.html#storing-offsets

## 说明

需要注意的是，Spark和Kafka的版本，必须要集成一致，一般来说，我们目前所使用的kafka的版本都需要在0.10以上

注意Streaming-kafka包

```
groupId = org.apache.spark
artifactId = spark-streaming-kafka-0-10_2.12
version = 3.2.0
```



