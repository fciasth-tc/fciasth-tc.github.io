---
title: Go语言struct methods该使用pointer或value传值?
date: 2019-09-11 11:26:47
categories:Go
tags:go
---

在 Go 语言如何区分 `func (s *MyStruct)` 及 `func (s MyStruct)`，底下我们先来看看简单的 Struct 例子

<!-- more -->

```go
package main

import "fmt"

type Cart struct {
    Name  string
    Price int
}

func (c Cart) GetPrice() {
    fmt.Println(c.Price)
}

func main() {
    c := &Cart{"bage", 100}
    c.GetPrice()
}
```



上面是个很简单的 Go struct 例子，假设我们需要动态更新 Price 值，可以新增 `UpdatePrice` method。[线上执行范例](https://play.golang.org/p/MPU3W-qR26)

```go
package main

import "fmt"

type Cart struct {
    Name  string
    Price int
}

func (c Cart) GetPrice() {
    fmt.Println("price:", c.Price)
}

func (c Cart) UpdatePrice(price int) {
    c.Price = price
}

func main() {
    c := &Cart{"bage", 100}
    c.GetPrice()
    c.UpdatePrice(200)
    c.GetPrice()
}
```

上面可以看到输出的结果是 100，只用 value 传值是无法改 Struce 内成员。我们可以用另外方式绕过。[线上执行范例](https://play.golang.org/p/sckO_D1ImM)

```go
package main

import "fmt"

type Cart struct {
    Name  string
    Price int
}

func (c Cart) GetPrice() {
    fmt.Println("price:", c.Price)
}

func (c Cart) UpdatePrice(price int) *Cart {
    c.Price = price
    return &c
}

func main() {
    c := &Cart{"bage", 100}
    c.GetPrice()
    c = c.UpdatePrice(200)
    c.GetPrice()
}
```

从上面范例可以发现，将 struct 回传，这样就可以正确拿到修改的值。但是这解法不是我们想要的。来试试看用 Pointer 方式 [线上执行范例](https://play.golang.org/p/euf_D2cE15)

```go
package main

import "fmt"

type Cart struct {
    Name  string
    Price int
}

func (c Cart) GetPrice() {
    fmt.Println("price:", c.Price)
}

func (c Cart) UpdatePrice(price int) {
    fmt.Println("[value] Update Price to", price)
    c.Price = price
}

func (c *Cart) UpdatePricePointer(price int) {
    fmt.Println("[pointer] Update Price to", price)
    c.Price = price
}

func main() {
    c := &Cart{"bage", 100}
    c.GetPrice()
    c.UpdatePrice(200)
    fmt.Println(c)
    c.UpdatePricePointer(200)
    fmt.Println(c)
}
```

只要使用 pointer 方式传值就可以正确将您需要改变的值写入，所以这边可以结论就是，如果只是要读值，可以使用 Value 或 Pointer 方式，但是要写入，则只能用 Pointer 方式。其实在 Go 语言官方有[整理 FAQ](https://golang.org/doc/faq#methods_on_values_or_pointers)，竟然之前都没发现，参考底下官方给的建议。

### 写入或读取

如果您需要对 Struct 内的成员进行修改，那请务必使用 Pointer 传值，相反的，Go 会使用 Copy struct 方式来传入，但是用此方式你就拿不到修改后的资料。

### 效能

假设 Struct 内部成员非常的多，请务必使用 Pointer 方式传入，这样省下的系统资源肯定比 Copy Value 的方式还来的多。

### 一致性

在开发团队内，如果有人使用 Pointer 有人使用 Value 方式，这样写法不统一，造成维护效率非常低，所以官方建议，全部使用 Pointer 方式是最好的写法。