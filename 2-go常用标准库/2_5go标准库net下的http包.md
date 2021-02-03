# net/http介绍

go内置的net/http包提供了http客户端和服务端的实现

## http协议

http超文本传输协议，提供了发送和接收html页面的方法

# http客户端

## http/https请求

http包内置了实现GET POST HEAD...这些方法的方法，分别对应http.Get http.Post http.PostForm

e.g.

```go
resp, err := http.Get("http://example.com/")
...
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})
```

**得到的resp对象，在使用完毕后，必须关闭**

```go
resp, err ;= http.Get("http://www.example.com/")
if err != nil {
    // handle error
}
// 注册一个关闭语句
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)
// ...
```



## GET请求示例

利用`net/http`包实现一个简单的发送http请求的客户端，如下：

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	// 得到一个resp对象，是*Response
	resp, err := http.Get("https://boogie96.gitee.io")
	if err != nil {
		fmt.Println("get failed,", err)
		return
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("read from resp.Body failed, err :%v\n", err)
		return
	}
	fmt.Println(string(body))
}

```

上述代码，可以获取boogie96.gitee.io的网站首页文件，并以纯文本方式打印到终端，浏览器其实也是http协议客户端，收发http数据，并把收到的文本文件，按照css，html，的规则渲染出来

## 带参数的GET请求

1、server端

```go
package main

import (
	"fmt"
	"net/http"
)

// handleGet 接收一个w写回复类型，接收一个r请求类型
func handleGet(w http.ResponseWriter, r *http.Request) {
	defer r.Body.Close()
	// 先注册一个请求的body关闭语句

	// Values map[string][]string 通过URL.Query()方法，得到客户端发来的参数，是个map[string][]string类型，
	// 然后用get取出各个参数值，再返回客户端一个回复
	data := r.URL.Query()
	fmt.Println(data.Get("name"))
	fmt.Println(data.Get("age"))
	answer := "recv ok"
	w.Write([]byte(answer))
}
func main() {
	// 把url处理函数写在注册监听之前；对于/get url，调用handleGet处理，
	http.HandleFunc("/get/", handleGet)
	http.ListenAndServe("127.0.0.1:9000", nil)

	// 像下面这样，先监听，后调用url处理函数，就会出现，如下图失败案例所示：访问时出现404 page not found的情况，（一定要先把所有的url处理函数写在监听之前

	// http.ListenAndServe("127.0.0.1:9000", nil)
	// http.HandleFunc("/get/", handleGet)
}

```



2、client端

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
)

func main() {
	apiUrl := "http://127.0.0.1:9000/get"
	data := url.Values{}
	data.Set("name", "wang")
	data.Set("age", "24")

	// 对要访问的apiUrl进行编码处理
	u, err := url.ParseRequestURI(apiUrl)
	if err != nil {
		fmt.Println("parse url failed", err)
	}
	// 对要传入的参数进行编码，然后添加到编码后的apiUrl中
	u.RawQuery = data.Encode()
	fmt.Println(u.String())

	// 发起访问
	resp, err := http.Get(u.String())
	if err != nil {
		fmt.Println("request failed", err)
		return
	}
	defer resp.Body.Close()
	// 读取服务端的响应
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("get resp failed", err)
		return
	}
	fmt.Println(string(b))
}

```

成功访问：

![image-20210202232131748](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210202232131748.png)

失败示例：

![image-20210203101325384](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210203101325384.png)

## POST请求示例

1、服务端

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func postHandler(w http.ResponseWriter, r *http.Request) {
	/*
		1 先defer注册一个r的Body的关闭操作
		2 然后根据传入的http请求，contenttype的类别选择不同的处理逻辑，并从中取出post传入的数据
		3 然后返回客户端响应数据

	*/

	defer r.Body.Close()
	// 请求类型是application/x-www-form-urlencodeed时，解析数据

	// 解析在编码进url的参数数据并打印
	r.ParseForm()
	// fmt.Println(r.PostForm)
	// fmt.Println(r.PostForm.Get("name"), r.PostForm.Get("age"), "haha")

	// 请求类型是application/json时，从r.Body读数据
	b, err := ioutil.ReadAll(r.Body)
	if err != nil {
		fmt.Println("read request from client failed, ", err)
		return
	}
	fmt.Println(string(b))

	// 返回响应数据给客户端
	answer := "recv your data ,status is ok"
	w.Write([]byte(answer))

}

func main() {
	http.HandleFunc("/post", postHandler)
	http.ListenAndServe("127.0.0.1:8080", nil)
}

```



2、客户端

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"strings"
)

func main() {
	/*
		先定义要访问的url
		然后定义要传入的post参数
		然后将编码后的url和参数，传入http.Get
		获得响应，判断是否有错误，没有就打印成功，并接收客户端的响应
		其中编码post要传入的参数时，有2种方式
			1，把参数和Url编码到一起
			2，把参数编码到请求体的body中

	*/

	apiURL := "http://127.0.0.1:8080/post"
	// 表单数据
	// contentType 是 application/x-wwww-form-urlencoded

	// contentType := "application/x-www-form-urlencodeed"
	// data := `{"name":"wang", "age":24}`

	// json传参
	contentType := "application/json"
	data := `{"name":"wang", "age":24}`
	resp, err := http.Post(apiURL, contentType, strings.NewReader(data))
	if err != nil {
		fmt.Println("post failed,", err)
		return
	}

	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("recv from server failed,", err)
		return
	}
	fmt.Println(string(b))

	// json类型
}

```

3、运行截图

![image-20210203151920808](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210203151920808.png)

## 自定义Client

可以通过自定义client，管理http客户端的头部、重定向策略和其他设置

```go
client := &http.Client{
    CheckRedirect :redirectPolicyFunc,
}
resp, err := client.Get("http://example.com")
req, err := http.NewRequest("GET", "http://example.com", nil)
req.Header.Add("If-None-Match", `W/"wyzzy"`)
resp, err := client.Do(req)
```



## 自定义Transport

可以通过自定义Transport，管理代理、tls配置、keep-alive，压缩等配置

```go
tr := &http.Transport {
    TLSClientConfig ； &tls.Config{RootCAs: pool}
    DisableCompression :true,
}
client := &http.Client{Transport: tr}
resp ,err := client.Get("https://example.com")
```

Client和Transport类型都可以安全的被多个goroutine同时使用，考虑效率，应该一次建立，尽量复用

# 服务端

## 简单的server示例

服务端代码，在本地监听了127.0.0.1的9000端口，并实现了一个url的处理函数，f1，负责处理访问/get这个url的逻辑，**通过建立连接后获取的responseWrite对象，的Write方法向客户端写一个字符串**

```go
package main

import (
	"net/http"
)

func f1(w http.ResponseWriter, r *http.Request) {
	str := "this is test html"
	w.Write([]byte(str))
}
func main() {

	http.HandleFunc("/test/", f1)
	http.ListenAndServe("127.0.0.1:9000", nil)
}

```

然后启动编译后的server.exe，然后在浏览器访问本地的127.0.0.1:9000/test/

![image-20210202225557387](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210202225557387.png)

## 默认的server

ListenAndServe使用指定的监听地址和处理器启动一个http服务端。处理器参数通常是nil，表示用包变量DefaultServeMux作为处理器

Handle和HandleFunc函数可以向DefaultServeMux添加处理器

```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request){
    fmt.Fprintf(w, "hello, %q", html.EscapeString(r.URL.Path))
})

log.Fatal(http.ListenAndServe(":8080", nil))
```



## 自定义server

**要管理服务端的行为，可以创建一个自定义的server**

```go
s := &http.Server{
    Addr : ":8080",
    Handler: myHandler,
    ReadTimeout: 10*time.Second
    WriteTimeout: 10 * time.Second
    MaxHeaderBytes: 1<<20
}
log.Fatal(s.ListenAndServe)
```



