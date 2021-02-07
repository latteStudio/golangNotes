# 为什么需要Context

场景举例：

**go http包的server中，每个请求都有一个goroutine启动一个请求处理函数去处理，而请求处理函数通常会再启动额外的goroutine去访问后端服务，例如连接缓冲、连接数据库等等 （因此一个请求的多个阶段，一般对应多个goroutine在处理，而针对同一个请求，这些处理不同阶段的goroutine通常需要一些公用的信息，比如用户的账户密码信息、验证的token、请求的截至时间等），当一个请求被取消或超时时，处理该请求各个阶段的goroutine都应该一并迅速退出，然后系统释放goroutine所占用的资源**

## 基本示例

下例中，worker函数会一直运行，如何在调用它的main函数中，通知它何时退出呢？

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func worker() {
	defer wg.Done()
	for {
		fmt.Println("worker processing...")
		time.Sleep(time.Second)
	}

}
func main() {
	wg.Add(1)
	go worker()
	wg.Wait()
	fmt.Println("over")
}
$ ./26context.exe
worker processing...
worker processing...
worker processing...
worker processing...
```



## 全局变量方式

通过定义一个全局变量，为是否退出goroutine的标志位，在子goroutine中判断全局变量是否为true，如果为true就退出，在main函数中就可以通过通知全局变量在控制子goroutine退出的时机

如下：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup
var isOut bool // 默认是false

func worker() {
	defer wg.Done()
	for {
		if isOut {
			break
		}
		fmt.Println("worker processing...")
		time.Sleep(time.Second)
	}

}
func main() {
	wg.Add(1)
	go worker()

	time.Sleep(time.Second * 5)
	isOut = true
	wg.Wait()
	fmt.Println("over")
}
$ ./26context.exe
worker processing...
worker processing...
worker processing...
worker processing...
worker processing...
over
```

**存在的问题**

1. 跨包调用时变量名不统一，各个开发者的习惯不同
2. 且多层goroutine嵌套时，控制逻辑很复杂



## 通道方式

通过定义一个消息通知的通道，在子goroutine的运行中，利用一个select中的case分支，每次判断该通道是否发来了消息，如果有消息就退出。main函数可以通过向该通道中传值，来控制子goroutine的是否退出；

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func worker(exitChan chan bool) {

	defer wg.Done()
LOOP:
	for {
		select {
		case <-exitChan:
			break LOOP
		default:
		}

		fmt.Println("worker processing...")
		time.Sleep(time.Second)
	}

}
func main() {
	var exitChan = make(chan bool, 1)
	wg.Add(1)
	go worker(exitChan)

	time.Sleep(time.Second * 5)
	exitChan <- true
	close(exitChan)

	wg.Wait()
	fmt.Println("over")
}
$ ./26context.exe
worker processing...
worker processing...
worker processing...
worker processing...
worker processing...
over
```

问题：

1. 跨包调用需要维护一个公用的channel，不容易规范

## 官方的Context

利用官方的Context实现控制goroutine的退出；

1、一层goroutine

```go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func worker(ctx context.Context) {
	defer wg.Done()
TAG:
	for {
		select {
		// 类似通道方式，ctx的Done方法得到的也是一个通道，具体类型：<-chan struct{}
		case <-ctx.Done():
			break TAG
		default:
		}

		fmt.Println("worker processing...")
		time.Sleep(time.Second)
	}
}
func main() {

	// 拿到ctx用于层层向子goroutine传递，拿到的cancel函数，只需要在main函数中调用cancel()，所有持有与之匹配的ctx的子goroutine都会退出；
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go worker(ctx)

	time.Sleep(time.Second * 5)
	// 调用cancel()方法，在worker中的ctx.Done()通道就会接收到一个值，根据其逻辑，就会跳出循环，进而结束goroutine的函数，进而结束goroutine
	// 补充：goroutine退出的时机，goroutine中的函数，执行完毕时
    // 调用cancel()通知子goroutine退出
	cancel()

	wg.Wait()
	fmt.Println("over")
}
$ ./26context.exe
worker processing...
worker processing...
worker processing...
worker processing...
worker processing...
over
```



2、二层goroutine

ctx可以一直传递，传递多层子goroutine

```go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func worker(ctx context.Context) {
	defer wg.Done()
	// 再开启一层goroutine
	go worker2(ctx)
TAG:
	for {
		select {
		// 类似通道方式，ctx的Done方法得到的也是一个通道，具体类型：<-chan struct{}
		case <-ctx.Done():
			break TAG
		default:
		}

		fmt.Println("worker processing...")
		time.Sleep(time.Second)
	}
}

func worker2(ctx context.Context) {
TAG:
	for {
		select {
		case <-ctx.Done():
			break TAG
		default:
		}
		fmt.Println("worker2 processing...")
		time.Sleep(time.Second)
	}
}
func main() {

	// 拿到ctx用于层层向子goroutine传递，拿到的cancel函数，只需要在main函数中调用cancel()，所有持有与之匹配的ctx的子goroutine都会退出；
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go worker(ctx)

	time.Sleep(time.Second * 5)
	// 调用cancel()方法，在worker中的ctx.Done()通道就会接收到一个值，根据其逻辑，就会跳出循环，进而结束goroutine的函数，进而结束goroutine
	// 补充：goroutine退出的时机，goroutine中的函数，执行完毕时
	cancel()

	wg.Wait()
	fmt.Println("over")
}
$ ./26context.exe
worker processing...
worker2 processing...
worker processing...
worker2 processing...
worker2 processing...
worker processing...
worker2 processing...
worker processing...
worker2 processing...
worker processing...
over
```





# Context简介

# Context接口

## Background()和TODO()

# With系列函数

## WithCancel

## WithDeadline

## WithTimeout

## WithValue

# Context注意事项

# 客户端超时取消示例

## server端

## client端

