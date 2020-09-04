---
title: 接入disconf使用docker部署jar包启动无法指定配置参数
date: 2019-10-04 11:21:46
categories: Docker
tags: docker
---

接入disconf使用docker部署jar包启动无法指定配置参数，官方其实已经指出解决方案

<!-- more -->

![image-20200904112807411](https://i.loli.net/2020/09/04/p4f89U2JjWQtvLM.png)

这对于直接用java -jar 来启动该springboot项目是OK的，但是构建到docker里就不行了，dockerfile是提前写好的，里面已经写死了启动命令了，后期我们只能去修改docker的启动端口什么的，无法再传入上面的java -Ddisconf.XXX参数了（或者会的话告诉我一下，就不用下面的步骤了）。那么怎么在不同的环境下动态设置disconf.env参数呢，在使用同一个docker镜像的情况下。

查看源码，我们可以知道：执行顺序是这样的，先读取disconf.properties里的所有属性，然后赋值，譬如将配置文件里的disconf.env定义的rd取出来，赋给变量env。然后再去读取系统环境变量，System.getProperty(name)，如果也有值，就覆盖从properties里读取的值，这样就是官方说的从java命令行输入参数就能直接动态覆盖配置文件。

根据这个特性我们就能来定制env了，对的，就是使用环境变量。我们只需要在项目启动时加载disconf.env的环境变量，就能动态指定env了。在docker下，环境变量是很容易设置的。

解决方案如下：

```java
package com.xy.onlineteam.HSEI.config;

import com.baidu.disconf.client.DisconfMgrBean;
import com.baidu.disconf.client.DisconfMgrBeanSecond;
import org.springframework.context.EnvironmentAware;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;

/**
 * @ClassName DisConfig
 * @Description TODO
 * @Author tangchao@mfexcel.com
 * @Date 2020/8/24 15:43
 * @Version 1.0
 */
@Configuration
public class DisConfig implements EnvironmentAware {

    @Bean(destroyMethod = "destroy")
    public DisconfMgrBean getDisconfMgrBean() {
        DisconfMgrBean disconfMgrBean = new DisconfMgrBean();
        disconfMgrBean.setScanPackage("com.xy.onlineteam.HSEI");
        return disconfMgrBean;
    }

    @Bean(destroyMethod = "destroy", initMethod = "init")
    public DisconfMgrBeanSecond getDisconfMgrBean2() {
        return new DisconfMgrBeanSecond();
    }

    @Override
    public void setEnvironment(Environment environment) {
        String env = environment.getProperty("disconf.conf_server_host");
        if(env != null) {
            System.setProperty("disconf.conf_server_host", env);
        }
    }
}
```

docker run 启动时，-e 指定disconf环境参数