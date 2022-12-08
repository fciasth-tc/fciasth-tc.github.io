---
title: docker和k8s常见问题记录
date: 2022-06-15 15:27:04
categories: K8s
tags:
  - k8s
---

docker 和 k8s 相关问题记录

<!-- more -->

### 1. k8s 或 docker 中运行 arthas 出现"Unable to get pid of LinuxThreads manager thread"的问题

在镜像中添加 init 功能，就是将[tini](https://github.com/krallin/tini)添加到镜像中

```dockerfile
FROM openjdk:8-jdk-alpine
COPY ./app.jar /opt/
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["java","-jar","/opt/app.jar"]
```

### 2. 强制删除 pod

```bash
kubectl delete pod app_pod --grace-period=0 --force
```
