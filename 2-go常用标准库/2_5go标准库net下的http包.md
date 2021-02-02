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

```



## POST请求示例

## 自定义Client

## 自定义Transport

# 服务端

## 默认的server

## 默认的server示例

## 自定义server

