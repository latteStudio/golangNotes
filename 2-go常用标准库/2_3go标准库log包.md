# log

软件开发时的调试阶段，和软件上线后的运行阶段，都需要记录相应日志

## 使用logger

log包定义了logger类型，该类型提供了格式化输出的方法，也提供预定义的“标准”logger，通过调用函数print系列(print|printf|printfln)，fatal系列(fatal|fatalf|fataln)和panic系列(panic|panicf|panicln)使用

示例：

fatal系列的打印，会将日志输出后，调用os.Exit(1)，而panic系列的打印会在写入日志后panic

```go
package main

import "log"

func main() {
	log.Println("这是一条普通日志")
	v := "普通日志"

	log.Printf("一条普通日志,\n", v)
	log.Fatalln("fatal日志")
	// 执行不到下面的panic日志，遇到fatal就退出了os.Exit(1)
	log.Panicln("panic日志")
}
latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/15log (master)
$ go run main.go
2021/01/25 16:21:57 这是一条普通日志
2021/01/25 16:21:57 一条普通日志,
%!(EXTRA string=普通日志)
2021/01/25 16:21:57 fatal日志
exit status 1
```

panic日志：

```go
package main

import "log"

func main() {
	log.Println("这是一条普通日志")
	v := "普通日志"

	log.Printf("一条普通日志,\n", v)
	// log.Fatalln("fatal日志")
	// 执行不到下面的panic日志，遇到fatal就退出了os.Exit(1)
	log.Panicln("panic日志")
}

latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/15log (master)
$ go run main.go
2021/01/25 16:24:35 这是一条普通日志
2021/01/25 16:24:35 一条普通日志,
%!(EXTRA string=普通日志)
2021/01/25 16:24:35 panic日志
panic: panic日志


goroutine 1 [running]:
log.Panicln(0xc00010ff68, 0x1, 0x1)
        D:/mySoftware/Go/src/log/log.go:365 +0xb3
main.main()
        D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:12 +0x114
exit status 2
```



## 配置logger

### 标准logger的配置

默认情况下，logger只提供日志的时间信息，而若要记录该日志的文件名和行号等信息，`log标准库`提供了定制这些设置的方法；

log标准库中的flags函数会返回标准logger的输出配置，而SetFlags函数设置标准logger的输出配置

```go
func Flags() int
func SetFlags(flag int)
```



### flag选项

log标准库提供了如下的flag选项，是一系列定义好的常量，（用于控制要输出哪些日志信息，而不控制它们的顺序和格式）

```go
const (
	Ldate         = 1 << iota     // the date in the local time zone: 2009/01/23
	Ltime                         // the time in the local time zone: 01:23:23
	Lmicroseconds                 // microsecond resolution: 01:23:23.123123.  assumes Ltime.
	Llongfile                     // full file name and line number: /a/b/c/d.go:23
	Lshortfile                    // final file name element and line number: d.go:23. overrides Llongfile
	LUTC                          // if Ldate or Ltime is set, use UTC rather than the local time zone
	Lmsgprefix                    // move the "prefix" from the beginning of the line to before the message
	LstdFlags     = Ldate | Ltime // initial values for the standard logger
)
```

示例：

```go
package main

import "log"

func main() {

	// 先设置日志格式
	log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)
	// 再打印日志
	log.Println("一条普通日志，")
}
// 输出的日志带有日期和完整的文件路径名
latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/15log (master)
$ go run main.go
2021/01/25 16:40:56.503568 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:10: 一条普通日志，
```



### 配置日志前缀

log标准库还提供了关于日志信息前缀的2个方法：

```go
func Prefix() string // 查看标准logger的输出前缀
func SetPrefix() (prefix string) // 设置logger的输出前缀
```

示例：

```go
package main

import (
	"fmt"
	"log"
)

func main() {

	// 先设置日志格式
	log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)
	// 再打印日志
	fmt.Println(log.Prefix())

	log.Println("一条普通日志")
	// 设置日志前缀
	log.SetPrefix("[一个前缀]")
	fmt.Println(log.Prefix())
    // 查看日志前缀
	log.Println("还是一个普通日志")
}

latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/15log (master)
$ go run main.go

2021/01/25 16:47:34.646835 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:15: 一条普通日志
[一个前缀]
[一个前缀]2021/01/25 16:47:34.657805 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:19: 还是一个普通日志
```



### 配置日志输出位置

配置日志输出位置的函数，默认是标准错误输出

```go
func SetOuptut(w io.Writer)
```

下例将日志输出到同目录下的test.log中

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	outputFile, err := os.OpenFile("./test.log", os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println(err)
		return
	}

    // 设置日志输出位置、日志前缀、日志flag控制输出哪些额外信息
	log.SetOutput(outputFile)
	log.SetPrefix("[test log]")
	log.SetFlags(log.Llongfile | log.Ldate)
	log.Println("一行日志")
	log.Println("一行日志")
	log.Println("一行日志")
}

```

输出到test.log内容如下：

```
[test log]2021/01/25 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:19: 一行日志
[test log]2021/01/25 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:20: 一行日志
[test log]2021/01/25 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:21: 一行日志
[test log]2021/01/25 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:19: 一行日志
[test log]2021/01/25 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:20: 一行日志
[test log]2021/01/25 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:21: 一行日志
```



**写到init（）函数中**

可以将上述步骤写到init()函数中

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func init() {
	outputFile, err := os.OpenFile("./test.log", os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println(err)
		return
	}

	log.SetOutput(outputFile)
	log.SetPrefix("[test log]")
	log.SetFlags(log.Llongfile | log.Ldate)

}

func main() {

	log.Println("一行日志")
	log.Println("一行日志")
	log.Println("一行日志")
}
```



## 创建logger

log标准库提供了构造函数New，可以创建一个新的logger对象，定义如下：

```go
func New(out io.Writer, prefix string, flag int) *Logger {
	return &Logger{out: out, prefix: prefix, flag: flag}
}
```

e.g.

```go
package main

import (
	"log"
	"os"
)

func main() {

	alogger := log.New(os.Stdout, "<my log>", log.Ldate|log.Llongfile)
	alogger.Println("一条新日志")

}
latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/15log (master)
$ go run main.go
<my log>2021/01/25 D:/myCode/Go/src/learngo/basic_grammar/15log/main.go:11: 一条新日志
```



## 总结

go内置log库功能有限，实际项目中常需要用第三方日志库如 [logrus](https://github.com/sirupsen/logrus)、[zap](https://github.com/uber-go/zap)等

