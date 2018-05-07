---
title: "一秒开始OpenTracing"
date: 2018-05-02T16:43:14+08:00
draft: false
bigimg: [{src: "https://res.cloudinary.com/shaocongcong/image/upload/v1525252394/blog/trace/opentracing/tracing_kenan.jpg", desc: "tracing"}]
categories: ["trace"]
keywords: [ "监控", "go" ]
tags: [ "监控" ]
---

### 背景
随着应用架构的演变，从单体系统渐渐的转变为微服务架构，其中业务调用关系也转变为服务与服务之间的调用与请求。分布式监控系统也应运而起，随着Google大爷的Dapper论文的发布，市面上出现了一大批优秀的监控系统，当然它们各有优缺点。

- Dapper(Google) : 各 tracer 的基础 
- StackDriver Trace (Google) 
- Zipkin(twitter) 
- Appdash(golang) 
- 鹰眼(taobao) 
- 谛听(盘古，阿里云云产品使用的Trace系统) 
- 云图(蚂蚁Trace系统) 
- sTrace(神马) 
- X-ray(aws)
- jaeger(Uber)

当然本文的重点还是在opentracing。

### OpenTracing
opentracing是一套分布式追踪协议，与平台，语言无关，统一接口，方便开发接入不同的分布式追踪系统。

一个完整的opentracing调用链包含 Trace + span + 无限极分类

- Trace：追踪对象，一个Trace代表了一个服务或者流程在系统中的执行过程，如：test.com，redis，mysql等执行过程。一个Trace由多个span组成
- span：记录Trace在执行过程中的信息，如：查询的sql，请求的HTTP地址，RPC调用，开始、结束、间隔时间等。
- 无限极分类：服务与服务之间使用无限极分类的方式，通过HTTP头部或者请求地址传输到最低层，从而把整个调用链串起来。

当然opentracing只是一套标准，要玩转还需要配合Jaeger。
Jaeger 是 Uber 推出的一款开源分布式追踪系统，兼容 OpenTracing API。

### Jaeger架构

![Jaeger服务之间关系](https://res.cloudinary.com/shaocongcong/image/upload/v1525423400/blog/trace/opentracing/jaeger-architecture.png)


### Jaeger Docker部署
用docker起一个示例Jaeger服务。

    docker run -d -p 5775:5775/udp -p 16686:16686 jaegertracing/all-in-one:latest

访问 [http://127.0.0.1:16686](http://127.0.0.1:16686)

如下图成功安装示例
![](https://res.cloudinary.com/shaocongcong/image/upload/v1525410176/blog/trace/opentracing/jaeger_ui.png)


好了废话说了一堆直接上代码如何使用Opentracing

``` go
package main

import (
	"log"
	"time"

	"github.com/opentracing/opentracing-go"
	"github.com/uber/jaeger-client-go"
	"github.com/uber/jaeger-client-go/config"
)

func main() {
	cfg := config.Configuration{
		Sampler: &config.SamplerConfig{
			Type:  "const",
			Param: 1,
		},
		Reporter: &config.ReporterConfig{
			LogSpans:            true,
			BufferFlushInterval: 1 * time.Second,
			LocalAgentHostPort:  "127.0.0.1:5775", // 数据上报地址
		},
	}
	tracer, closer, err := cfg.New(
		"Hello Jaeger",
		config.Logger(jaeger.StdLogger),
	)
	if err != nil {
		log.Panic(err)
	}
	opentracing.SetGlobalTracer(tracer)
	defer closer.Close()

	someFunction()

}

func someFunction() {
	parent := opentracing.GlobalTracer().StartSpan("hello")
	defer parent.Finish()

	child := opentracing.GlobalTracer().StartSpan("world", opentracing.ChildOf(parent.Context()))
	defer child.Finish()
}

```

    go run main.go

    2018/05/04 13:08:11 Initializing logging reporter
    2018/05/04 13:08:11 Reporting span 3e27061246d7ffe6:3f98ef24d2fb1144:3e27061246d7ffe6:1
    2018/05/04 13:08:11 Reporting span 3e27061246d7ffe6:3e27061246d7ffe6:0:1


然后到Jaeger去查询刚刚的调用，如下图：
![span_hello_2](https://res.cloudinary.com/shaocongcong/image/upload/v1525410983/blog/trace/opentracing/jaeger_hello_2.jpg)
![span_hello_4](https://res.cloudinary.com/shaocongcong/image/upload/v1525410983/blog/trace/opentracing/jaeger_hello_4.jpg)

OK ! 简单的示例就到这里结束了，这里Jaeger是通过docker部署的，官方在对于非docker部署方式的文档基本是无的，后面我会写一篇如果在Linux下通过二进制部署Jaeger，See you! 