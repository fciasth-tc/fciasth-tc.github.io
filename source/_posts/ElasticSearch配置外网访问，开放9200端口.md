---
title: ElasticSearch配置外网访问，开放9200端口
date: 2018-05-27 09:25:26
tags: 
   - elasticsearch
   - linux
categories: ElasticSearch
toc: true
---

#### 前言

最近在学习springboot整合elasticsearch，在腾讯云的服务器上用docker装上es后，发现elasticsearch默认不开放外网访问。所以折腾了一下。记录一下配置流程。

<!-- more -->

#### 配置流程

**1. 进入elasticsearch主目录下**

```shell
vim config/elasticsearch.yml 
```

**2. 添加下面内容**

```properties
 network.host: 0.0.0.0
 http.port: 9200
```

注：前面没有#注释，还有就是顶头加一个空格，冒号后也要加空格。 ==> 空格 network.host:空格 0.0.0.0