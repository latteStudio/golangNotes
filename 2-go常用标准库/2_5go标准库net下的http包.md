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

func handleGet(w http.ResponseWriter, r *http.Request) {
	defer r.Body.Close()

	data := r.URL.Query()
	fmt.Println(data.Get("name"))
	fmt.Println(data.Get("age"))
	answer := "recv ok"
	w.Write([]byte(answer))
}
func main() {
	http.HandleFunc("/get/", handleGet)
	http.ListenAndServe("127.0.0.1:9000", nil)

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
	u, err := url.ParseRequestURI(apiUrl)
	if err != nil {
		fmt.Println("parse url failed", err)
	}
	u.RawQuery = data.Encode()
	fmt.Println(u.String())
	resp, err := http.Get(u.String())
	if err != nil {
		fmt.Println("request failed", err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("get resp failed", err)
		return
	}
	fmt.Println(string(b))
}

```

![image-20210202232131748](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210202232131748.png)

## POST请求示例

## 自定义Client

## 自定义Transport

# 服务端

## 简单的server示例

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

## 默认的server示例

## 自定义server

