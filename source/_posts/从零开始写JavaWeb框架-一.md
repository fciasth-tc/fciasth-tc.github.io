---
title: 从零开始写JavaWeb框架(一)-从一个简单的Web应用开始
date: 2018-05-07 13:44:57
tags:
  - java
  - web
categories: Java
toc: true
---

 正所谓“工欲善其事，必先利其器”，在正式开始设计并开发我们的轻量级Java Web框架之前，有必要先掌握以下技能：1.使用IDEA搭建并开发Java项目，2.使用Maven自动化构建Java项目，3.使用Git管理项目代码

<!-- more -->

## 使用IDEA创建Maven项目

### 创建IDEA项目

我们无须单独下载Maven，更不用将其继承到IDEA中，因为IDEA默认已经将其整合了。

在IDEA中创建Maven项目非常简单，只需要按照以下步骤进行即可：

1. 单击 “File” 按钮,选择 “New” ，选择“Project”,弹出 New Project 对话框。

2. 选择Maven选项，单击 ”Next“ 按钮。

3. 输入GroupId，ArtifactId，Version，单击 ”Next“ 按钮。

4. 输入Project name、Project location，单击 “Finish” 按钮。

   (Maven相关知识，请自行了解。)

按照以上步骤，IDEA就创建了一个基于Maven的目录结构，如图：

![项目目录结构](/images/QQ截图20180507140912.png)

### 调整Maven配置

打开项目中的pom.xml文件，添加配置如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.fciasth</groupId>
    <artifactId>chapter1</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 统一源代码的编码方式，否则使用Maven编译源代码时会出现相关警告 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <build>
        <plugins>
            <!-- 除了需要统一源代码编码方式外，还需要统一源代码与编译输出的JDK版本，此项目中使用JDK1.8 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <!-- 可选配置，当使用Maven打包时，想跳过单元测试，可添加如下插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </build>


</project>
```

至此，一个Maven项目就搭建完毕了，下面我们就要在这个项目中添加有关Java Web的代码。

## 搭建Web项目框架

### 转为Java Web项目

目前，在pom.xml中还没有任何的Maven依赖(dependency)，随后会添加一些Java Web所需的依赖，只不过在添加这些依赖之前，有必要先将这个Maven项目调整为Web项目结构。

只需按照以下三步即可实现：

1. 在main目录下，添加webapp目录。
2. 在webapp下，添加WEB-INF目录。
3. 在WEB-INF目录下，添加web.xml文件。

此时，IDEA给出一个提示。

![Web项目提示](/images/QQ截图20180507140922.png)

表示IDEA已经识别出目前我们使用了Web框架(即Servlet框架)，需要进行一些配置才能使用。单击 “Configure” 按钮，会看到一个确认框，单击 “OK” 按钮即可将当前项目变为Web项目。

这里打算使用Servlet3.0框架，所以在web.xml中添加如下代码:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmln="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
         http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">

</web-app>

```

实际上，使用Servlet3.0框架是可以省略web.xml文件的，因为Servlet无须在web.xml中进行配置，只需通过注解的形式来配置即可，下面绘描述具体的方法。

### 添加Java Web的Maven依赖

由于Web项目是需要打包成war包的，因此必须在pom.xml文件中设置packaging为war(默认为jar)，配置如下

```xml
<packaging>war</packaging>
```

接下来就需要添加Java Web所需的Servlet、JSP、JSTL等依赖了，添加如下配置：

```xml
<dependencies>
        <!-- Servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>
        <!-- JSP -->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <!-- JSTL -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
    </dependencies>
```

需要说明的是：

1. Maven依赖的“三坐标”(groupId、artifactId、version)必须提供。
2. 如果某些依赖只需参与编译，而无需打包(例如Tomcat自带了Servlet与JSP所对应的jar包)，可将其scope设置为provided。
3. 如果某些依赖只是运行时需要，但无需参与编译（例如JSTL的jar包），可将其scope设置为runtime。

为了方便阅读，以下是pom.xml的完整代码:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.fciasth</groupId>
    <artifactId>chapter1</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>war</packaging>

    <!-- 统一源代码的编码方式，否则使用Maven编译源代码时会出现相关警告 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- Servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>
        <!-- JSP -->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <!-- JSTL -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- 除了需要统一源代码编码方式外，还需要统一源代码与编译输出的JDK版本，此项目中使用JDK1.8 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <!-- 可选配置，当使用Maven打包时，想跳过单元测试，可添加如下插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>

            <!-- Tomcat -->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <path>/${project.artifactId}</path>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

现在一个基于Maven的Java Web项目已经搭建完毕。