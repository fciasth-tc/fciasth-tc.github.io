---
title: 5款好用的开源Docker工具
date: 2019-12-25 14:20:58
categories: Docker
tags: docker
---

最近，开发者谢哈尔·古拉蒂（Shekhar Gulati）收集了一些他在日常工作中使用的又有趣又实用的 Docker 工具，他称这些工具提升了自己的工作效率，减少了原本需要手工完成的工作。以下就是古拉蒂推荐的五款开源 Docker 工具，以及他选择它们的原因。

<!-- more -->

1. [Watchtower](https://github.com/containrrr/watchtower)：自动更新 Docker 容器

   Watchtower 监视运行容器以及这些容器最初启动时的镜像状态。当 Watchtower 检测到一个镜像已经有变动时，它会使用新镜像自动重新启动相应的容器。我想在我的本地开发环境中尝试最新的构建镜像，所以使用了它。

   Watchtower 本身被打包为 Docker 镜像，因此可以像运行任何其它容器一样运行它。要运行 Watchtower，你需要执行以下命令：

   ```shell
   $ docker run -d --name watchtower --rm -v /var/run/docker.sock:/var/run/docker.sock  v2tec/watchtower --interval 30
   ```

   在上面的命令中，我们使用一个挂载文件（/var/run/docker.sock）启动了 Watchtower 容器。这么做是有必要的，为的是使 Watchtower 可以与 Docker 守护 API 进行交互。我们将 30 秒传递给间隔选项 interval，此选项定义了 Watchtower 的轮询间隔。Watchtower 支持更多的选项，你可以根据[文档](https://github.com/containrrr/watchtower#options)中的描述来使用它们。

2. [Docker-gc](https://github.com/spotify/docker-gc)：容器和镜像的垃圾回收

   Docker-gc 工具通过删除不需要的容器和镜像来帮你清理 Docker 主机。它会删除存在超过一个小时的所有容器。此外，它还删除不属于任何留置容器的镜像。

   你可以将 Docker-gc 作为脚本和容器来使用。若要使用 Docker-gc 来查找所有可以删除的容器和镜像，命令如下：

   ```shell
   $ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -e DRY RUN=1 spotify/docker-gc
   ```

   上述命令中，我们加载了 docker.sock 文件，以便 Docker-gc 能够与 Docker API 交互。我们传递了一个环境变量 DRY_RUN=1 来查找将被删除的容器和镜像。需要注意的是，如果不提供该参数，Docker-gc 会删除所有容器和镜像，所以最好事先确认 Docker-gc 要删除的内容。上述命令的输出如下所示：

   ```shell
   
   [2017-04-28T06:27:24] [INFO] : The following container would have been removed 0c1b3b0972bb792bee508 60c35a4 bc08ba32b527d53eab173d12a15c28deb931/vibrant_ yonath
   [2017-04-28T06:27:24] [INFO] : The following container would have been removed 2a72d41e4b25e2782f7844e188643e395650a9ecca660e7a0dc2b7989e5acc28 
   /friendlyhello_ web
   [2017-04-28T06:27:24] [INFO] : The following image would have been removed sha256:00f017a8c2a6e1 fe2f fd05c281 f27d069d2a99323a8cd514dd35f228ba26d2ff
   [busybox: latest]
   [2017-04-28T06:27:24] [ INFO] : The following image would have been removed sha256 :4a323b466a5ac4ce6524 8dd970b538922c54e535700cafe9448b52a3094483ea
   [hello-world:latest]
   [2017-04-28T06:27:24] [INFO] : The following image would have been removed sha256:4a323b4 66a5ac4ce65248dd970b538922c54e535700cafe9448b52a3094483ea
   [python:2.7-slim]
   ```

   如果你认同 Docker-gc 清理方案， 可以不使用 DRY_RUN 再次运行 Docker-gc 执行清空操作。

   ```shell
   $ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock spotify/docker-gc
   ```

3. [Docker-slim](https://github.com/docker-slim/docker-slim)：面向容器的神奇“减肥药”

   如果你担心你的 Docker 镜像的大小，Docker-slim 可以帮你排忧解难。

   该工具使用静态和动态分析方法来为你臃肿的镜像瘦身。要使用 Docker-slim，可以从 GitHub 下载 Linux 或者 Mac 的二进制安装包。成功下载之后，将它加入到你的系统变量 PATH 中。

4. [Rocker](https://github.com/grammarly/rocker)：突破 Dockerfile 的限制

   大多数使用 Docker 的开发人员都使用 Dockerfile 来构建镜像。Dockerfile 是一种声明式的方法，用于定义用户可以在命令行上调用的所有命令，从而组装镜像。

   Rocker 为 Dockerfile 指令集增加了新的指令。据了解，Rocker 的创建者 Grammarly 正是为了解决他们遇到的 Dockerfile 格式的问题，才创建了 Rocker。要使用 Rocker，首先必须在你的机器上安装。对 Mac 用户来说，就是简单地运行几条 brew 命令：

   ```shell
   $ brew tap grammarly/tap
   $ brew install grammarly/tap/rocker
   ```

   一旦完成安装，你就可以通过传递 Rockerfile 使用 Rocker 来构建镜像了：

   ```shell
   FROM python:2.7-slim
   WORKDIR /app
   ADD . /app
   RUN pip install -r requirements. txt
   EXPOSE 80
   ENV NAME World
   CMD ["python","app.Py"]
   TAG shekhargulati/ friendlyhello:{{ .VERSION }}
   PUSH shekhargulati/friendlyhello:{{ .VERSION }}
   ```

   若要构建一个镜像并将其推送到 Docker Hub，你可以运行以下命令：

   ```shell
   $ rocker d build --push -var VERSION-1.0
   ```

   Rocker 有一组很好的特性，要了解更多信息，请参考它的[文档](https://github.com/grammarly/rocker/blob/master/README.md)。

5. ctop：容器的类顶层接口Ctop 

   是我最近开始使用的一个工具，它能够提供多个容器的实时指标视图。如果你是一个 Mac 用户，可以使用 brew 安装，如下所示：

   ```shell
   $ brew install ctop
   ```

   一旦完成安装，就可以开始使用 ctop 了。你只需要配置 DOCKER_HOST 环境变量。你可以运行 ctop 命令，查看所有容器的状态。若只想查看正在运行的容器，可以使用 ctop -a 命令。

   Ctop 是一个简单的工具，对于了解在你的主机上运行的容器很有帮助。你可以在 ctop 文档中了解更多相关信息。以上就是古拉蒂发现的 5 款实用 Docker 工具。

   原文链接：[5 Docker Utilities You Should Know](https://dzone.com/articles/5-docker-utilities-you-should-know)