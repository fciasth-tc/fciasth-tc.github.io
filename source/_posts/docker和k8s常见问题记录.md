---
title: docker和k8s常见问题记录
date: 2022-06-15 15:27:04
categories:
  - Docker
  - K8s
tags: 
  - docker
  - k8s
---

docker和k8s相关问题记录

<!-- more -->

### 1. k8s或docker中运行arthas出现"Unable to get pid of LinuxThreads manager thread"的问题

在镜像中添加init功能，就是将[tini](https://github.com/krallin/tini)添加到镜像中

```dockerfile
FROM openjdk:8-jdk-alpine
COPY ./app.jar /opt/
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["java","-jar","/opt/app.jar"]
```



### 2. 强制删除pod

```bash
kubectl delete pod app_pod --grace-period=0 --force
```

