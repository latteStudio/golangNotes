# goWeb示例

## net/http包实现简单web

- 读取简单的html文件并返回；

### 直接响应特定字符串

1、利用内置的net/http包实现一个简单的web server

```go
package main

import (
	"fmt"
	"net/http"
)

func sayHello(w http.ResponseWriter, r *http.Request) {
	_, _  = fmt.Fprint(w, "<h1> hello Golang <h1> <h2> good good coding <h2>")
}
func main() {
	http.HandleFunc("/hello", sayHello)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		fmt.Printf("http server started failed : %v\n", err)
		return
	}
}

```

2、启动后访问

先启动：

```
D:\myCode\Go\src\ginDemo>ginDemo.exe

```

![image-20210215180729540](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210215180729540.png)

### 响应从文件中读取的内容



1、新建一个html文件

![image-20210215181135569](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210215181135569.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>this is hello.html</title>
</head>
<body>

<h1>hello Golang</h1>
<h2>coding is cool</h2>

<img id="img1" src="https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2FGMzxtRIo9ge0wbvkaUWhfQ26tw22sqWOlMJvKMOik5T5r1547871437818.jpg&thumbnail=650x2147483647&quality=80&type=jpg" alt="">
<button id="b1">点击切换图片</button>
<script>
    document.getElementById("b1").onclick=function () {
        document.getElementById('img1').src='http://n.sinaimg.cn/sinakd20210215ac/716/w877h1439/20210215/ddc0-kkciesq5349785.jpg'
    }
</script>
</body>
</html>
```

2、将sayHello函数改为从html文件中读取内容后，然后响应给前端

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func sayHello(w http.ResponseWriter, r *http.Request) {
	// 从二进制所在目录找到hello.html文件，读取后得到[]byte类型
	respStr, err := ioutil.ReadFile("./hello.html")
	// 对可能的错误进行处理
	if err != nil {
		fmt.Println("read from ./hello.html failed, please check the file.")
		return
	}
	// 将读取的内容，写到w中，从而可以被客户端接收到，一般是浏览器，（一定要转为string类型，否则无法正常展示页面）
	_, _  = fmt.Fprint(w, string(respStr))
}
func main() {
	http.HandleFunc("/hello", sayHello)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		fmt.Printf("http server started failed : %v\n", err)
		return
	}
}

```



3、再次访问，就得到了hello.html解析后的界面

![image-20210215183133025](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210215183133025.png)

点击按钮，通过js注册的按钮动作，可以实现更换图片

![image-20210215183212159](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210215183212159.png)



# gin框架项目

## gin简介

https://www.liwenzhou.com/posts/Go/Gin_framework/

## gin示例

## gin的restfulApi示例

## template模版与渲染

https://www.liwenzhou.com/posts/Go/go_template/

### go原生模版渲染示例



