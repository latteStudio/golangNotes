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

go1.7加入了一个新标准库context，它定义了Context类型，专用于简化处理单个请求的多个goroutine之间与请求域的数据、取消信号，截止时间等操作，这些操作可以涉及多个api调用；

对传入服务器的请求应该创建上下文，而对服务器的传出调用应该接收上下文，之间的函数链调用必须传递上下文，或可以使用WithCancel WithDeadline WithTimeout WithValue创建的派生上下文，当一个上下文取消时，其派生的上下文也该被取消；





# Context接口

context.Context是一个接口，该接口定义了四个需要实现的方法，如下：

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

- Deadline（）方法返回当前Context被取消的时间，即完成工作的截止时间
- Done()方法许返回一个Channel，该Channel会在当前工作完成或上下文被取消后关闭，
- Err（）方法返回当前Context结束的原因
  - 若当前Context被取消返回Canceled错误
  - 若当前Context超时就返回DeadlineExceed错误
- Value（）方法会从Context中返回键对应的值，

## Background()和TODO()

go内置2个函数，Background（）和TODO（），这2个函数分别返回一个实现了Context接口的background和todo，代码中最开始以这2个内置的上下文对象作为最顶层的partent context，衍生出更多的子上下文对象

Background（）主要用于main函数，初始化以及测试代码中，作为context这个树结构的最顶层的Context，也是根Context

TODO（），目前不知道具体场景，

background和todo本质上都是emptyCtx结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context

# With系列函数

context包中包含了2个with系列函数

## WithCancel

WithCancel返回带有新Done通道的父节点的副本，当调用cancel函数时，将关闭返回上下文的Done通道，

函数签名：

```go
func WithCancel(parent Context) (ctx Context, cacel CancelFunc)
```

e.g.

```go
package main

import (
	"context"
	"fmt"
)

func gen(ctx context.Context) <-chan int {
	// 声明一个无缓冲通道，也叫同步通道，一端发时，另一端必须有人收否则就阻塞
	ch1 := make(chan int)

	n := 1
	// 立即执行函数
	go func() {
		for {
			select {
			// 判断是否在main中执行了cancel函数，执行了这里就会从通道接收到值
			case <-ctx.Done():
				return // 接受到就return，即退出该立即执行函数，随机它的goroutine也就退出了
			case ch1 <- n:
				n++
			}
		}
	}()
	// 这里的return为什么没有直接退出这个立即执行函数，因为ch1是一个无缓冲通道，需要上面的goroutine传过来值才可以返回，
	return ch1
}
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	for i := range gen(ctx) {
		fmt.Println(i)
		if i == 5 {
			break
		}
	}
}


$ ./26context.exe
1
2
3
4
5
```



## WithDeadline

函数签名：

```go
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
```

返回父上下文的副本，并将deadline调整为不迟于d。如果父上下文的deadline已经早于d，则WithDeadline(parent, d)在语义上等同于父上下文。当截止日过期时，当调用返回的cancel函数时，或者当父上下文的Done通道关闭时，返回上下文的Done通道将被关闭，以最先发生的情况为准。

取消此上下文将释放与其关联的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel。

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// 定义一个50毫秒后过期的上下文，得到ctx上下文，和其对应的取消函数
	d := time.Now().Add(50 * time.Millisecond)
	ctx, cancel := context.WithDeadline(context.Background(), d)

	// 尽管这个ctx只有50毫秒就会过期，任何情况下，明确调用它的cancel（）函数都是好的实践
	// 若不这样做，可能会使得其上下文及其父类存活的时间超过必要的时间
	defer cancel()

	select {
	case <-time.After(1 * time.Second):
		fmt.Println("1 second later")
	// 这里的Done不是因为调用了cancel（）函数而有值的，是因为其过期时间到了		
	case <-ctx.Done():
		fmt.Println(ctx.Err())

	}

}
$ ./26context.exe
context deadline exceeded
```

程序会阻塞在select中，因为50毫秒先到，所以会阻塞到先执行第二个case，50毫秒后，ctx过期，ctx.Done()收到内容，然后打印ctx的Err信息；

## WithTimeout

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
LOOP:
	for {
		fmt.Println("db connecting...")
		time.Sleep(time.Millisecond * 10)
		select {
		// 这里会因为50毫秒的过期时间到而退出，而不是因为调用了cancel函数，因为这个函数在5s后才会被调用

		case <-ctx.Done():
			break LOOP
		default:

		}

	}
	fmt.Println("worker done")
	wg.Done()
}
func main() {
	// 得到一个50毫秒生命周期的上下文
	ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
	wg.Add(1)
	go worker(ctx)
	// 睡5s中
	time.Sleep(time.Second * 5)
	// 手动调用cancel（）不会影响，因为在此之前，就因为过期而退出了
	cancel()
	wg.Wait()
	fmt.Println("over")
}
$ ./26context.exe
db connecting...
db connecting...
db connecting...
db connecting...
db connecting...
worker done
over
```



## WithValue

**没看明白？**

函数签名：

```go
func WithValue(parent Context, key, val interface{}) Context
```

e.g.

```go
package main

import (
	"context"
	"fmt"
	"sync"

	"time"
)

// context.WithValue

type TraceCode string

var wg sync.WaitGroup

func worker(ctx context.Context) {
	key := TraceCode("TRACE_CODE")
	traceCode, ok := ctx.Value(key).(string) // 在子goroutine中获取trace code
	if !ok {
		fmt.Println("invalid trace code")
	}
LOOP:
	for {
		fmt.Printf("worker, trace code:%s\n", traceCode)
		time.Sleep(time.Millisecond * 10) // 假设正常连接数据库耗时10毫秒
		select {
		case <-ctx.Done(): // 50毫秒后自动调用
			break LOOP
		default:
		}
	}
	fmt.Println("worker done!")
	wg.Done()
}

func main() {
	// 设置一个50毫秒的超时
	ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
	// 在系统的入口中设置trace code传递给后续启动的goroutine实现日志数据聚合
	ctx = context.WithValue(ctx, TraceCode("TRACE_CODE"), "12512312234")
	wg.Add(1)
	go worker(ctx)
	time.Sleep(time.Second * 5)
	cancel() // 通知子goroutine结束
	wg.Wait()
	fmt.Println("over")
}


$ ./26context.exe
worker, trace code:12512312234
worker, trace code:12512312234
worker, trace code:12512312234
worker, trace code:12512312234
worker, trace code:12512312234
worker done!
over
```



# Context注意事项

- 推荐以参数的方式显式传递Context
- 以Context作为参数的函数方法，应该将Context作为第一个参数
- 给一个函数传递Context时，不要传递nil，若不知道传递什么，传递context.TODO()
- Context的Value相关方法应该传递请求域的必要数据，不应该用于传递可选参数
- Context是线程安全的，可以放心的在多个goroutine中传递

# 客户端超时取消示例

调用服务端的api时，如何在客户端实现超时控制；

## server端

1. 正常的编写一个httpHandler函数
2. 在其中加入一个随机数，制造随机的响应延迟，模拟服务端的响应延迟

代码：

```go
package main

import (
	"fmt"
	"math/rand"
	"net/http"
	"time"
)

func indexHandler(w http.ResponseWriter, r *http.Request) {

	// 生成一个随机数
	number := rand.Intn(2)
	if number == 0 {
		time.Sleep(time.Second * 5)
		fmt.Fprintf(w, "slow response")
		return
	}
	fmt.Fprintf(w, "normal response")
}
func main() {
	http.HandleFunc("/", indexHandler)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("listen failed ", err)
		return
	}

}

```



## client端

1. 先定义一个全局的结构体，respData用于存储要收到的响应，包含2个字段
   1. resp *http.Response类型
   2. err 可能的error类型
2. 在main函数中，定义一个100毫秒的超时，获得对应生命周期的ctx和其cancel（）函数
3. 注册一个defer，显式的调用cancel（）
4. 调用客户端请求函数，子函数中
   1. 定义一个通道rChan，类型是*respData类型，
   2. http.Client函数，构造一个client对象
   3. http.NewRequest函数，构造一个req对象
   4. 利用req的WithContext方法传入ctx上下文，得到，有超时时长的新req
   5. 然后开启 一个goroutine，利用client.Do(req)去发起请求
      1. 若正常请求到了，没超时，把得到resp和err组成一个*respData类型，丢到rChan中，下面的select会收到并执行对应的正常分支
      2. 若没请求到，超时了，无法得到resp和err，也就无法构造*respData类型，丢不到rChan中，此时一旦过了50毫秒的时间，ctx过期，下面的select ctx.Done()分支会收到，并执行对于的超时分支
   6. 程序继续向下走，在select中定义2个分支，
      1. 要么ctx.Done()中得到值，表示请求超时了，超时的提示超时信息
      2. 要么resp的通道中得到值，表示正常未超时，没超时的把响应正常打印

```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"time"
)

// 定义一个类型，用于存放响应数据
type respData struct {
	resp *http.Response
	err  error
}

func doReq(ctx context.Context) {
	// 定义个chan，用于传递响应数据
	var rChan = make(chan *respData, 1)

	// 构造client,是短链接类型
	transport := &http.Transport{
		DisableKeepAlives: true,
	}
	client := &http.Client{
		Transport: transport,
	}

	// 构造请求Request,其类型是*http.Request
	req, err := http.NewRequest("GET", "http://127.0.0.1:8080", nil)
	if err != nil {
		fmt.Println("new request failed,", err)
		return
	}
	// 利用*http.Request类型的WithContext方法，将得到的req，变成一个带有ctx的50毫秒超时时长的新req对象
	req = req.WithContext(ctx)

	// 开启goroutine执行向服务端发起请求
	go func() {
		resp, err := client.Do(req)
		if err != nil {
			fmt.Println("request to server failed,")
			return
		}
		// 如果正常请求，构造得到的resp和err
		respData1 := &respData{
			resp: resp,
			err:  err,
		}
		rChan <- respData1
	}()

	// 上面的goroutine后台启动一个子goroutine执行，这里继续用select判断
	// 会先阻塞，直到超时时间到达，或者超时时间内，得到正常的resp
	select {
	case <-ctx.Done():
		fmt.Println("req is timeout")
	case result := <-rChan:
		fmt.Println("req is normal")
		fmt.Printf("resp:%v , err:%v\n", result.resp, result.err)
	}

}

func main() {
	// 得到一个50毫秒超时时间的ctx和其对于的cancel（）
	ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
	// 显式调用cancel（）
	defer cancel()
	doReq(ctx)

}

```

client优化：

1. 利用WaitGroup对goroutine进行同步
2. 得到的resp，应该调用其Body.Close()方法进行关闭

```go
package main

import (
	"context"
	"fmt"
	"io/ioutil"
	"net/http"
	"sync"
	"time"
)

// 定义一个类型，用于存放响应数据
type respData struct {
	resp *http.Response
	err  error
}

func doReq(ctx context.Context) {
	// 定义个chan，用于传递响应数据
	var rChan = make(chan *respData, 1)

	// 构造client,是短链接类型
	transport := &http.Transport{
		DisableKeepAlives: true,
	}
	client := &http.Client{
		Transport: transport,
	}

	// 构造请求Request,其类型是*http.Request
	req, err := http.NewRequest("GET", "http://127.0.0.1:8080", nil)
	if err != nil {
		fmt.Println("new request failed,", err)
		return
	}
	// 利用*http.Request类型的WithContext方法，将得到的req，变成一个带有ctx的50毫秒超时时长的新req对象
	req = req.WithContext(ctx)

	// 开启goroutine执行向服务端发起请求
	var wg sync.WaitGroup
	wg.Add(1)
	defer wg.Wait()
	go func() {
		defer wg.Done()
		resp, err := client.Do(req)
		if err != nil {
			fmt.Println("request to server failed,")
			return
		}
		// 如果正常请求，构造得到的resp和err
		respData1 := &respData{
			resp: resp,
			err:  err,
		}
		rChan <- respData1

	}()

	// 上面的goroutine后台启动一个子goroutine执行，这里继续用select判断
	// 会先阻塞，直到超时时间到达，或者超时时间内，得到正常的resp
	select {
	case <-ctx.Done():
		fmt.Println("req is timeout")
	case result := <-rChan:
		// 注册关闭resp body的代码
		defer result.resp.Body.Close()
		data, _ := ioutil.ReadAll(result.resp.Body)
		fmt.Printf("resp : %v \n", string(data))
	}

}

func main() {
	// 得到一个50毫秒超时时间的ctx和其对于的cancel（）
	ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
	// 显式调用cancel（）
	defer cancel()
	doReq(ctx)

}

```



## 运行效果：

![image-20210208165707882](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210208165707882.png)

优化后效果：

![image-20210208171726236](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210208171726236.png)