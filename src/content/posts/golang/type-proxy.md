---
title: Go动态代理
published: 2025-09-08
description: '一次随手的 Go 动态代理小笔记'
image: ''
tags: [go,golang,go-type-proxy,go-func-proxy]
category: 'golang'
draft: false 
lang: ''
---

今天闲逛QQ群，发现群友发了一个挺有意思的截图，算是 Go 里的一种“动态代理”小写法，记录一下

## 示例代码

```go
package main

import "fmt"

type ProxyFunc func(hello string)

func (pf *ProxyFunc) proxy(hele string) {
    (*pf)("hello") // 先执行原始函数
    fmt.Println(hele) // 再加上额外逻辑
}

func main() {
    var pf ProxyFunc

    pf = func(hele string) {
        fmt.Println(hele)
    }

    pf.proxy("world")

    pf.proxy("fuck")
}
````

## 运行效果

执行结果大概是这样的：

```
hello
world
hello
fuck
```

也就是说，通过 `proxy` 方法，可以在调用原函数时额外插入逻辑，某种意义上类似“代理模式”或者“中间件包装”。

## 随手感想

* 这种写法有点 hack 味道，不过也挺 Go 的：**函数就是一等公民**，你可以把它挂在结构里，也能给它加上包装。
* 如果把这种模式用在日志、监控、限流等地方，可能还能玩出点花样。
* 不过这种方式可读性一般，生产里要慎用，哈哈。

先记一笔，后面有空再研究下。