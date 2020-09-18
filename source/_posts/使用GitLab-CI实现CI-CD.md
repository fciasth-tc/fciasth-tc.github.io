---
title: 使用GitLab-CI实现CI/CD
date: 2019-09-18 16:34:59
categories: Docker
tags: docker
---

之前有用 `Jenkins` 做过 `CI/CD`，但是太吃内存了，所以尝试下用 `GitLab CI` 做持续集成。

<!-- more -->

### GitLab CI

`GitLab CI` 是 GitLab 提供的持续集成服务(从 `8.0` 版本之后，`GitLab CI` 已经集成在 GitLab 中了)，只要在你的仓库根目录下创建一个`.gitlab-ci.yml` 文件，并为该项目指派一个 Runner，当有合并请求或者 Push 操作时，你写在 `.gitlab-ci.yml` 中的构建脚本就会开始执行，能够自动执行打包，构建，以及部署。

###  环境说明

以下例子是基于：

- gitlab服务：http://git.mfexcel.com/

- GitLab Runner 服务器：`10.0.8.237`，用于`maven打`包，构建`docker`镜像，并推送到`docker`镜像私服

- 部署服务器：`10.0.8.77`，拉取镜像并进行部署

`gitlab`和`gitlab runner`的安装就不再赘述了。

请确保上述2台服务器都已安装`docker`环境，若想使用`kubernetes`进行部署发布，`10.0.8.77`上也有使用`minikube`搭建的`kubernetes`服务，只需简单修改`.gitlab-ci.yml`文件即可。

### 配置maven环境

由于我们需要在`Gitlab runner`服务器`10.0.8.237`的`docker`中进行项目的打包即构建，首先要解决的一个问题就是`maven`的配置问题，因为我们某些第三方包可能需要从`nexus`私服上进行拉取，所以需要对`maven`的配置文件进行修改，定制一个用于`Gitlab runner`进行构建的定制`maven`镜像，里面同时包含了`docker`和`jdk8`的环境，用于之后的打包及镜像构建。

以下为具体的`Dockerfile`

```yml
FROM maven:3.5.4-jdk-8-alpine
MAINTAINER tangchao <tangchao@mfexcel.com>
COPY settings.xml /usr/share/maven/ref/
RUN apk add --no-cache \
        ca-certificates \
        curl \
        openssl
RUN set -x; \
    curl -fSL "https://download.docker.com/linux/static/stable/x86_64/docker-18.03.1-ce.tgz" -o docker.tgz; \
    tar -xzvf docker.tgz; \
    mv docker/* /usr/local/bin/; \
    rmdir docker; \
    rm docker.tgz; \
    docker -v
```

`maven`的`settings`配置文件：继承`nexus`私服以及阿里云仓库

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>/root/.m2/repository</localRepository>
  <servers>
    <server>
      <id>maven-snapshots</id>
      <username>hudan</username>
      <password>_Cio-v482Hgk_6</password>
    </server>
	<server>
      <id>releases</id>
      <username>hudan</username>
      <password>_Cio-v482Hgk_6</password>
    </server>
	<server>
      <id>snapshots</id>
      <username>hudan</username>
      <password>_Cio-v482Hgk_6</password>
    </server>
  </servers>
 <mirrors>
	<mirror>
		<id>alimaven</id>
		<name>aliyun maven</name>
		<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
		<mirrorOf>central</mirrorOf>        
	</mirror>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://192.168.1.98:8081/repository/maven-public/</url>
    </mirror>
  </mirrors>
  
  <profiles>
    <profile>
      <id>nexus</id>
      <repositories>
        <repository>
          <id>maven-public</id>
          <url>http://192.168.1.98:8081/repository/maven-public/</url>
		  <releases><enabled>true</enabled></releases>
		  <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
	  <pluginRepositories>
		<pluginRepository>
			<id>maven-public</id>
			<url>http://192.168.1.98:8081/repository/maven-public/</url>
			<releases><enabled>true</enabled></releases>
			<snapshots><enabled>true</enabled></snapshots>
		</pluginRepository>
	  </pluginRepositories>
    </profile>
  </profiles>

  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
</settings>
```

使用 `docker build -t registry.bizport.cn/ali-maven-docker:3.5.4-jdk-8-alpine .` 构建镜像

发布到私服 `docker push registry.bizport.cn/ali-maven-docker:3.5.4-jdk-8-alpine`

### 注册Gitlab runner

```shell
[root@localhost]# gitlab-runner register   #1 注册gitlab runner
Runtime platform                                    arch=amd64 os=linux pid=18471 revision=738bbe5a version=13.3.1
Running in system-mode.                            
                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://git.mfexcel.com/  #2 连接的gitlab服务地址
Please enter the gitlab-ci token for this runner:
FzPmqp4A22zf4ZSQR_z6   #3 具体项目的token，可在项目的setting中查看
Please enter the gitlab-ci description for this runner:
[localhost.localdomain]: xy-docker runner   #4 描述，随便写
Please enter the gitlab-ci tags for this runner (comma separated):
xy-docker runner   #5 tag，用于后续 .gitlab-ci的配置中使用
Registering runner... succeeded                     runner=FzPmqp4A
Please enter the executor: virtualbox, docker-ssh+machine, docker-ssh, ssh, parallels, shell, docker+machine, kubernetes, custom, docker:
docker  #6 采用docker方式进行执行构建
Please enter the default Docker image (e.g. ruby:2.6):
registry.bizport.cn/ali-maven-docker:3.5.4-jdk-8-alpine   #7 runner中执行构建时使用的docker镜像，使用上述定制的maven镜像
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```

### 修改config.toml

进入`/etc/gitlab-runner`目录下，找到`config.toml`，修改刚刚注册`runner`

```toml
[[runners]]
  name = "xy-docker runner"
  url = "http://git.mfexcel.com/"
  token = "93ea6f5168ad29a2708eaeca189abf"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
  [runners.docker]
    tls_verify = false
    image = "registry.bizport.cn/ali-maven-docker:3.5.4-jdk-8-alpine"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache", "/data/apache-maven-repository:/root/.m2/repository"]
    shm_size = 0
    pull_policy = "if-not-present"
```

添加 maven 库目录的本地映射，这样每次打包时不需要重下依赖包。另外末尾加上 `pull_policy = "if-not-present"`，这样不会每次都拉镜像。

### 编写 .gitlab-ci.yml

```yaml
variables:  #1 变量
  DOCKER_HOST: tcp://10.0.8.237:2375 # 2 gitlab-runner的docker service地址
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ''
  MAVEN_OPTS: '-Dmaven.repo.local=/root/.m2/repository'
  IMAGE_NAME: 'registry.bizport.cn/xy-third-syn-system'
  # 部署命令
  RUN_SCRIPT: 'docker run --name xy-third-syn-system -d -p 8099:8080 $IMAGE_NAME'

services:  #3 在docker中运行docker的一种工具
  - docker:dind
  
stages:  #4  Pipline的整个流程
  - package
  - build
  - deploy
  
before_script:
  - export TAG=$IMAGE_NAME:$CI_COMMIT_SHA
  - export DEPLOY=$RUN_SCRIPT:$CI_COMMIT_SHA

package:  #5  项目打包
  stage: package
  tags:
    - xy-docker runner
  script:
    - mvn clean package -Dmaven.test.skip=true
  artifacts:
    paths:
      - target/*.jar #6   输出上传jar包，用于下一个job

build_image:  #7 构建镜像
  stage: build
  image: docker:dind 
  tags:
    - xy-docker runner
  script:
    - cp target/*.jar ./
    - docker build -f src/main/docker/Dockerfile -t $TAG .
    - docker push $TAG
  only:
    - master
    
deploy:  #8 部署
  stage: deploy
  image: ictu/sshpass:latest
  tags:
    - xy-docker runner
  script:
    # ssh免密登录，IP和密码配在gitlab ci的环境变量里
    - sshpass -p "$DEPLOY_HOST_PASSWORD" ssh -o StrictHostKeyChecking=no -tt root@$DEPLOY_HOST_IP $DEPLOY
  only:
    - master
```

### 效果

![](https://i.loli.net/2020/09/18/gyrmc4Yof6SQTnG.png)

`pipline`执行成功，并在部署服务器上成功部署。

![](https://i.loli.net/2020/09/18/u7xXNopJr6gtYvf.png)