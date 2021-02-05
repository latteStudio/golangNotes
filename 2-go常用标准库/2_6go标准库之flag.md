# os.Args

go内置的flag包实现了命令行参数的解析，flag包使得开发命令行工具更为简单

使用示例：

os.Args返回的是一个字符串类型的切片，其第一个元素为二进制执行文件的名称；然后就是在命令行中传入的各个参数

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// os.Args是一个[]string
	if len(os.Args) > 0 {
		for index, arg := range os.Args {
			fmt.Printf("参数%d, 参数值 %s\n", index, arg)
		}
	}
}

```



运行测试：

```
$ ./24flag.exe a b c haha
参数0, 参数值 C:\myworkstation\mygocode\src\learngo\basic_grammar\24flag\24flag.exe
参数1, 参数值 a
参数2, 参数值 b
参数3, 参数值 c
参数4, 参数值 haha
```

## 导入flag包

```go
import "flag"
```



## flag参数类型

flag包支持的命令行参数类型有：bool int int64 uint uint64 float float64 string duration

![image-20210205155341968](https://gitee.com/boogie96/pic-go-bed/raw/master/img/image-20210205155341968.png)

## 定义命令行flag参数

定义命令行参数有如下方法

### flag.Type()

基本格式如下：

```go
flag.Type(flag名,默认值，帮助信息) *Type

name := flag.String("name", "lisi" , "姓名")
age := flag.Int("age", 18, "年龄")
married := flag.Bool("married", false, "婚否")
delay :=flag.Duratiom("d", 0, "时间间隔")

此时，name age married delay ，均为对应类型的指针
```



### flag.TypeVar()

基本格式如下：

```go
flag.TypeVar(Type指针， flag名， 默认值，帮助信息)
var name string
var age int
var married bool

flag.StringVar(&name, "name", "lisi", "姓名")
flag.IntVar(&age, "age", 18, "年龄")
flag.BoolVar(&married, "married", false, "婚否")
```



## flag.Parse()

通过以上2中方式定义好flag参数后，通过调用flag.Parse()来对命令行参数进行解析

支持的命令行参数格式有下面几种：

- -flag xxx
- -- flag xxx
- -flag=xxx
- --flag=xxx

**其中bool类型的参数必须以等号方式指定**

**flag解析在第一个非flag参数之前停止，或者在终止符号- 之后停止**



## flag其他函数

```
flag.Args() 返回命令行参数后的其他参数，以[]string类型
flag.Narg() 返回命令行参数后的其他参数个数
flag.NFlag()返回使用的命令行参数个数
```



## 完整示例

### 定义

```go
package main

import (
	"flag"
	"fmt"
	"time"
)

// flag demo

func main() {
	// 定义四个变量，用于存储4个flag参数
	var name string
	var age int
	var married bool
	var delay time.Duration

	// 四个参数分别为，要存放到哪个变量的指针，flag字段名（即传参时--后的字符串），默认值，以及打印--help时,该字段的解释信息
	flag.StringVar(&name, "personName", "老王", "输入的人员姓名")
	flag.IntVar(&age, "personAge", 18, "人的年龄")
	flag.BoolVar(&married, "isMarried", false, "结婚状态")
	flag.DurationVar(&delay, "howlong", 3600000000000, "结婚多久的时间")

	// 解析命令行参数
	flag.Parse()
	// 打印解析后的flag的值
	fmt.Println(name, age, married, delay)

	// 打印命令行参数后的其他参数
	fmt.Println(flag.Args())
	// 打印命令行参数后的其他参数个数
	fmt.Println(flag.NArg())
	// 打印命令行参数的个数
	fmt.Println(flag.NFlag())

}

```



### 使用

```
打印提示信息，输出了4个flag字段的字段名，类型，字段说明，和默认值

$ ./24flag.exe --help
Usage of C:\myworkstation\mygocode\src\learngo\basic_grammar\24flag\24flag.exe:
  -howlong duration
        结婚多久的时间 (default 1h0m0s)
  -isMarried
        结婚状态
  -personAge int
        人的年龄 (default 18)
  -personName string
        输入的人员姓名 (default "老王")
        



填入四个flag参数，看到对应的输出，四个flag参数后没有参数，所以flag.Narg()个数是0，flag.Args()字符串切片是空
$ ./24flag.exe -howlong=24h -isMarried=true -personAge=24 -personName=shuaibi
shuaibi 24 true 24h0m0s
[]
0
4




四个flag参数后加了3个参数，所以flag.Narg()个数是3flag.Args()字符串切片是这三个字符串， a b c
ten@DESKTOP-7ICOPRG MINGW64 /c/myworkstation/mygocode/src/learngo/basic_grammar/24flag (master)
$ ./24flag.exe -howlong=24h -isMarried=true -personAge=24 -personName=shuaibi a b c
shuaibi 24 true 24h0m0s
[a b c]
3
4


```

