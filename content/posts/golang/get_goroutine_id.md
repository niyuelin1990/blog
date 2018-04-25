---
title: "Golang获取goroutine ID"
date: 2018-04-25T16:28:29+08:00
draft: false
categories: ["golang"]
keywords: [ "golang", "go" ]
tags: [ "golang" ]
---


golang本身不提供获取goroutineID的接口，如果要获取goroutineID可以使用下面的方法

``` golang

package main

import (
    "bytes"
    "fmt"
    "runtime"
    "strconv"
)

func main() {
    fmt.Println(getGID())
}

func getGID() uint64 {
    b := make([]byte, 64)
    b = b[:runtime.Stack(b, false)]
    b = bytes.TrimPrefix(b, []byte("goroutine "))
    b = b[:bytes.IndexByte(b, ' ')]
    n, _ := strconv.ParseUint(string(b), 10, 64)
    return n
}

```



