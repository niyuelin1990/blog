---
title: "以太坊ipc实现方式以及golang有用的库---npipe"
date: 2018-04-27T15:02:50+08:00
draft: false
categories: ["blockchain"]
keywords: ["blockchain", "Npipe"]
tags: ["blockchain", "Npipe"]
---


## 背景
以太坊rpc有4种实现方式分别是**inproc**，**ipc**，**http**，**ws**。inproc是进程内部调用，为console使用；http是以http接口方式提供访问；ws是以websocket的方式提供访问。

**ipc**便是进程间通信，以npipe为底层实现，上层采用Json-Rpc为消息格式，并使用go的reflect包实现对内部Api的调用。  ipc通信是用于同一个主机间钱包等客户端与geth之间的通信。

## npipe
npipe是以Windows命名为管道(pipes)的一个go语言库。

[**pipes**](http://msdn.microsoft.com/en-us/library/windows/desktop/aa365780):管道是一种使用共享内存进行通讯的方式。创建管道的过程称为管道服务器。连接到管道的进行时管道客户端。一个进程将信息写入到管道，然后另一个进程从管道读取信息。

**[npipe](https://github.com/natefinch/npipe)**：npipe是基于stdlib包实现，并实现了 Dial, Listen, 和Accept方法，同时分装了net.Conn 和 net.Listener。可以以net来实现rpc调用。

**[stdlib](https://zh.wikipedia.org/wiki/Stdlib.h)**：stdlib.h是C标准函数库的头文件，声明了数值与字符串转换函数, 伪随机数生成函数, 动态内存分配函数, 进程控制函数等公共函数。 C++程序应调用等价的cstdlib头文件。这里主要是使用到了内存相关函数。

## npipe例子
npipe首先创建服务端，再用客户端调用。
c首先是创建服务端 pipeserver.go

``` go
import (
	"fmt"
	"github.com/natefinch/npipe"
	"bufio"
	"net"
)

func main() {

	//endpoint := fmt.Sprintf("test-ipc-%d-%d", os.Getpid(), rand.Int63())

	//if runtime.GOOS == "windows" {
	//	endpoint = `\\.\pipe\` + endpoint
	//} else {
	//	endpoint = os.TempDir() + "/" + endpoint
	//}
	// 开启连接
	endpoint:=`\\.\pipe\test-ipc-22244-5577006791947779410`
	fmt.Println(endpoint)

	server(endpoint)
}

//创建服务端
func server(endpoint string )  {
	ln, err := npipe.Listen(endpoint)
	if err != nil {
		panic(err)
	}
	func() {
		for {
			conn, err := ln.Accept()
			if err != nil {
				// handle error
				continue
			}
			// handle connection like any other net.Conn
			go func(conn net.Conn) {
				r := bufio.NewReader(conn)
				msg, err := r.ReadString('\n')
				if err != nil {
					// handle error
					return
				}
				fmt.Println(msg)
				fmt.Fprintln(conn, "Hi client!")
			}(conn)
		}
	}()
}
```
###再次创建客户端pipeclient.go

``` go
import (
	"fmt"
	"github.com/natefinch/npipe"
	"bufio"
)

func main() {
	endpoint:=`\\.\pipe\test-ipc-22244-5577006791947779410`
	conn, err := npipe.Dial(endpoint)
	if err != nil {
		// handle error
	}
	if _, err := fmt.Fprintln(conn, "Hi server!"); err != nil {
		// handle error
		fmt.Println(err)
	}
	r := bufio.NewReader(conn)
	msg, err := r.ReadString('\n')
	if err != nil {
		// handle eror
		fmt.Println(err)
	}
	fmt.Println(msg)
}
```
### 输出

``` go
Hi client!
```
``` go
Hi server!
```

## npipe在以太坊应用
以太坊中ipc位于rpc包中，以下是启动方式：
ipc.go
``` go
//开启连接
func ipcListen(endpoint string) (net.Listener, error) {
	return npipe.Listen(endpoint)
}

//ipc 服务端接收连接
func (srv *Server) ServeListener(l net.Listener) error {
	for {
		//接收连接
		conn, err := l.Accept()
		if err != nil {
			return err
		}
		log.Trace(fmt.Sprint("accepted conn", conn.RemoteAddr()))
		//解析json数据
		go srv.ServeCodec(NewJSONCodec(conn), OptionMethodInvocation|OptionSubscriptions)
	}
}
``` go
客户端调用方法：

``` go
func (c *Client) CallContext(ctx context.Context, result interface{}, method string, args ...interface{})
```
server端最终调用底层的方法
``` go
//server.go 311行调用内部方法
reply := req.callb.method.Func.Call(arguments)
```


## 总结
以往不同进程间通信我们一般会采用tcp或者http的方式调用，但是如果用管道通信库npipe可以减去网卡的通信；直接基于内存的通信性能和内存的开销达到最低。
