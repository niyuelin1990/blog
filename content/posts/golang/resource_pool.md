---
title: "Golang简单的对象池"
date: 2018-04-25T16:05:28+08:00
draft: false
categories: ["golang"]
keywords: [ "golang", "go" ]
tags: [ "golang" ]
---

* 复用的好处
    -  减少gc压力
    -  减少不必要的内存分配


```golang

import (
    "fmt"
	"sync"
)

var bufPool sync.Pool

type buf struct {
	b []byte
}

func main() {
	for {
		var bf *buf
		// 从池中取数据
		v := bufPool.Get()
		if v == nil {
			//若不存在buf，创建新的
			fmt.Println("no buf ,create!")
			bf = &buf{
				b: make([]byte, 10),
			}
		} else {
			// 池里存在buf,v这里是interface{}，需要做类型转换
			bf = v.(*buf)
		}
		fmt.Println("使用数据", bf)
		// bf使命完成，放入池中
		bufPool.Put(bf)
	}
}

```