---
title: Go语言搭配Docker Healthy Check检查
date: 2019-03-17 14:47:43
categories: Go
tags: 
  - docker
  - go
---

在 Docker 1.12 版本后，提供了 HEALTHCHECK 指令，通过指定的一行命令来判断容器内的服务是否正常运作。在此之前大部分都是透过判断程式是否 Crash 来决定容器是否存活，但是这地方有点风险的是，假设服务并非 crash，而是没办法退出容器，造成无法接受新的请求，这就确保容器存活。现在呢我们可以透过在 Dockerfile 内指定 HEALTHCHECK 指令来确保服务是否正常。而用 Go 语言开发的 Web 服务该如何来实现呢？

<!-- more -->

## 建立 /healthz 路由

透过简单的路由 `/healthz` 直接回传 200 status code 即可 (使用 [Gin](https://github.com/gin-gonic/gin) 当例子)。

```go
func heartbeatHandler(c *gin.Context) {
    c.AbortWithStatus(http.StatusOK)
}
```

透過瀏覽器 `http://localhost:8080/healthz` 可以得到空白網頁，但是打開 console 可以看到正確回傳值。[![Snip20180317_4](https://i1.wp.com/farm5.staticflickr.com/4774/26990632808_d800bc3800_z.jpg?w=840&ssl=1)](https://www.flickr.com/photos/appleboy/26990632808/in/dateposted-public/)

## 建立 ping 指令

透过 `net/http` 套件可以快速写个验证接口的函式

```go
func pinger() error {
    resp, err := http.Get("http://localhost:8080/healthz")
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    if resp.StatusCode != 200 {
        return fmt.Errorf("server returned non-200 status code")
    }
    return nil
}
```

## 增加 HEALTHCHECK 指令

在 `Dockerfile` 內增加底下內容:

```shell
HEALTHCHECK --start-period=2s --interval=10s --timeout=5s \
CMD ["/bin/gorush", "--ping"]
```

- **–start-period**: 容器启动后需要等待几秒，预设为 0 秒
- **–interval**: 侦测间隔时间，预设为 30 秒
- **–timeout**: 检查超时时间

重新编译容器，并且启动容器，会看到初始状态为

 [![Snip20180317_5](https://i2.wp.com/farm1.staticflickr.com/788/40861013721_d7327500f9_z.jpg?w=840&ssl=1)](https://www.flickr.com/photos/appleboy/40861013721/in/dateposted-public/)

經過 10 秒後，就會執行指定的指令，就可以知道容器健康與否，最後狀態為 。[![Snip20180317_6](https://i1.wp.com/farm1.staticflickr.com/783/39051186800_ee9a838403_z.jpg?w=840&ssl=1)](https://www.flickr.com/photos/appleboy/39051186800/in/dateposted-public/)

最后可以透过 `docker inspect` 指令来知道容器的状态列表 (JSON 格式)

```shell
$ docker inspect --format '{{json .State.Health}}' gorush | jq
```

[![Snip20180318_8](https://i1.wp.com/farm5.staticflickr.com/4781/40861130401_08ca9e2cce_z.jpg?w=840&ssl=1)](https://www.flickr.com/photos/appleboy/40861130401/in/dateposted-public/)

从上图可以知道每隔 10 秒 Docker 就会自动侦测一次。有了上述这些资料，就可以来写系统报警通知了。