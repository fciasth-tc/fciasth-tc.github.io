---
title: apollo使用docker-compose部署
toc: true
date: 2019-10-12 11:30:50
categories: Dcoker
tags: docker
---

apollo使用docker-compose部署

<!-- more -->

```yml
version: '2.2'
services:
  apollo:
    image: idoop/docker-apollo:latest
    ports:
      - "8070:8070"
      - "8180:8180"
      - "8190:8190"
      - "8181:8181"
      - "8191:8191"
      - "8182:8182"
      - "8192:8192"
      - "8183:8183"
      - "8193:8193"
    environment:
      # 开启Portal,默认端口: 8070
      PORTAL_DB: jdbc:mysql://10.0.200.34:3306/ApolloPortalDB?characterEncoding=utf8
      PORTAL_DB_USER: dev
      PORTAL_DB_PWD: 123456
      DEV_DB: jdbc:mysql://10.0.200.34:3306/ApolloConfigDB_DEV?characterEncoding=utf8
      DEV_DB_USER: dev
      DEV_DB_PWD: 123456
      DEV_ADMIN_PORT: 8190
      # 与对应环境库的ServerConfig表中的eureka.service.url，IP和端口号保持一致
      DEV_CONFIG_PORT: 8180
      # 开启fat环境, 默认端口: config 8081, admin 8091
      FAT_DB: jdbc:mysql://10.0.200.34:3306/ApolloConfigDB_FAT?characterEncoding=utf8
      FAT_DB_USER: dev
      FAT_DB_PWD: 123456
      FAT_ADMIN_PORT: 8191
      # 与对应环境库的ServerConfig表中的eureka.service.url，IP和端口号保持一致
      FAT_CONFIG_PORT: 8181
      # 开启uat环境, 默认端口: config 8082, admin 8092
      UAT_DB: jdbc:mysql://10.0.200.34:3306/ApolloConfigDB_UAT?characterEncoding=utf8
      UAT_DB_USER: dev
      UAT_DB_PWD: 123456
      UAT_ADMIN_PORT: 8192
      # 与对应环境库的ServerConfig表中的eureka.service.url，IP和端口号保持一致
      UAT_CONFIG_PORT: 8182
      # 开启pro环境, 默认端口: config 8083, admin 8093
      PRO_DB: jdbc:mysql://10.0.200.34:3306/ApolloConfigDB_PRO?characterEncoding=utf8
      PRO_DB_USER: dev
      PRO_DB_PWD: 123456
      PRO_ADMIN_PORT: 8193
      # 与对应环境库的ServerConfig表中的eureka.service.url，IP和端口号保持一致
      PRO_CONFIG_PORT: 8183
      # 指定注册IP
      eureka.instance.ip-address: 10.0.8.77
```

