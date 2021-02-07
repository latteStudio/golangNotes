# fmt

fmt包，实现了类似c语言中printf和scanf的格式化IO，分为向外输出内容，和获取输入内容2大块

## 向外输出

### Print

Print系列函数，会把内容输出到系统的标准输出，区别是：

- Print函数直接输出内容；
- Printf函数支持格式输出字符串；
- Println函数会在输出内容末尾加一个换行符

三个函数签名如下：

```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```



e.g.

```go
package main

import "fmt"

// /Print 系列函数demo
func main() {
	fmt.Print("哈哈")
	name := "嘻嘻"
	fmt.Printf("名字是 %s\n", name)
	fmt.Println("嘿嘿")
}
$ go run main.go
哈哈名字是 嘻嘻
嘿嘿
```



### Fprint

Fprint系列函数，相比于Print，会多传一个io.Writer类型的参数w，表示向w代表的位置写入信息，如文件、标准输出等，**只要是io.Writer类型的变量都可以写入**

函数签名如下：

```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```



e.g.

```go
package main

import (
	"fmt"
	"os"
)

// Fprint demo

func main() {
	// 写到标准输入
	fmt.Fprintln(os.Stdout, "哈哈")

	fileObj, err := os.OpenFile("./1.txt", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println("打开文件出错了", err)
		return
	}
	name := "xixi"
	fmt.Fprintf(fileObj, "向文件中写入信息： %s", name)
}


$ go run main.go
哈哈

```



### Sprint

**Sprint系列会把传入的数据生成一个字符串，并返回**

函数签名

```go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
```

e.g.

```go
package main

import (
	"fmt"
)

// Sprint demo
func main() {
	s1 := fmt.Sprint("哈哈哈")
	name := "wang"
	age := 24
	s2 := fmt.Sprintf("name :%s, age :%d\n", name, age)

	s3 := fmt.Sprintln("嘻嘻嘻")

	fmt.Println(s1, s2, s3)
}
$ go run main.go
哈哈哈 name :wang, age :24
 嘻嘻嘻

```



### Errorf

**Errorf函数根据format参数生成格式化字符串，并返回一个包含该字符串的错误**

函数签名：

```go
func Errorf(format string, a ...interface{}) error
```

通常使用该方式，定义了一个自定义错误类型，如下

```go
err := fmt.Errorf("一个自定义的错误")
```

go 1.13版本为fmt.Errof函数新加一个%w占位符，用来生成一个可以包裹Error的Wrapping Error

```go
e := errors.New("内置错误")
w := fmt.Errorf("自定义错误里Wrap了一个内置错误%w",e)
```

e.g.

```go
package main

import (
	"errors"
	"fmt"
)

//  Errorf demo

func main() {
	e := errors.New("内置错误")
	err := fmt.Errorf("自定义包含了一个内置错误 %w\n", e)

	fmt.Println(e)
	fmt.Println(err)
}
$ go run main.go
内置错误
自定义包含了一个内置错误 内置错误

```



## 格式化占位符

*printf系列函数都支持format格式化参数，这里按照占位符将被替换的变量类型划分

### 通用占位符

| 占位符 | 说明                               |
| ------ | ---------------------------------- |
| %v     | 值的默认格式输出                   |
| %+v    | 相比%v，输出时结构体时会多个字段名 |
| %#v    | 值的go语法表示                     |
| %T     | 打印值的类型                       |
| %%     | 百分号                             |

e.g.

```go
package main

import "fmt"

//  Errorf demo

func main() {
	fmt.Printf("%v\n", 100)

	fmt.Printf("%v\n", false)

	o := struct{ name string }{"hahaha"}
	fmt.Printf("%v\n", o)

	fmt.Printf("%#v\n", o)
	fmt.Printf("%T\n", o)
	fmt.Printf("100%%\n")
}
$ go run main.go
100
false
{hahaha}
struct { name string }{name:"hahaha"}
struct { name string }
100%

```



### 布尔型

```
%t 是true或false的占位符
```



### 整型

| 占位符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| %b     | 表示二进制                                                   |
| %c     | 该值对应的unicode编码                                        |
| %d     | 十进制                                                       |
| %o     | 二进制                                                       |
| %x     | 十六进制 a-f用小写                                           |
| %X     | 十六进制，A-F用大写                                          |
| %U     | 表示为Unicode格式：U+1234，等价于”U+%04X”                    |
| %q     | 该值对应的单引号括起来的go语法字符字面值，必要时会采用安全的转义表示 |

e.g.

```go
package main

import "fmt"

func main() {
	n := 65

	fmt.Printf("%b\n", n)
	fmt.Printf("%d\n", n)
	fmt.Printf("%o\n", n)

	fmt.Printf("%x\n", n)
	fmt.Printf("%X\n", n)
	fmt.Printf("%o\n", n)

	fmt.Printf("%c\n", n)
	fmt.Printf("%U\n", n)
	fmt.Printf("%q\n", n)

}
$ go run main.go
1000001
65
101
41
41
101
A
U+0041
'A'
```



### 浮点数与复数

| 占位符 | 说明                                                   |
| ------ | ------------------------------------------------------ |
| %b     | 无小说部分，二进制指数表示的科学计数法，如-123456p-78  |
| %e     | 科学计数法，如-1234.456e+78                            |
| %E     | 科学计数法，如-1234.456E+78                            |
| %f     | 有小数部分但无指数部分，如123.456                      |
| %F     | 等价于%f                                               |
| %g     | 根据实际情况采用%e或%f格式（以获得更简洁、准确的输出） |
| %G     | 根据实际情况采用%E或%F格式（以获得更简洁、准确的输出） |



e.g.

```go
package main

import "fmt"

func main() {

	f := 12.34

	fmt.Printf("%e\n", f)
	fmt.Printf("%E\n", f)
	fmt.Printf("%b\n", f)
	fmt.Printf("%f\n", f)
	fmt.Printf("%F\n", f)
	fmt.Printf("%g\n", f)
	fmt.Printf("%G\n", f)

}
$ go run main.go
1.234000e+01
1.234000E+01
6946802425218990p-49
12.340000
12.340000
12.34
12.34
```



### 字符串和[]byte

| 占位符 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|   %s   |                   直接输出字符串或者[]byte                   |
|   %q   | 该值对应的双引号括起来的go语法字符串字面值，必要时会采用安全的转义表示 |
|   %x   |           每个字节用两字符十六进制数表示（使用a-f            |
|   %X   |          每个字节用两字符十六进制数表示（使用A-F）           |



```go
package main

import "fmt"

func main() {

	s := "小王"
	fmt.Printf("%s\n", s)
	fmt.Printf("%q\n", s)
	fmt.Printf("%x\n", s)
	fmt.Printf("%X\n", s)

}
$ go run main.go
小王
"小王"
e5b08fe78e8b
E5B08FE78E8B
```



### 指针

| 占位符 |              说明              |
| :----: | :----------------------------: |
|   %p   | 表示为十六进制，并加上前导的0x |

e.g.

```go
package main

import "fmt"

func main() {

	a := 10
	fmt.Printf("%p\n", &a)
	fmt.Printf("%#p\n", &a)

}
$ go run main.go
0xc0000120c0
c0000120c0
```



### 宽度标识符

输出的宽度由跟在百分号后的一个十进制数表示，（可选：宽度后加. 点号之后再加一个十进制数表示精度）

若没有给定宽度标识位或没有精度标识位，则表示采用默认的值，如下：

| 占位符 |        说明        |
| :----: | :----------------: |
|   %f   | 默认宽度，默认精度 |
|  %9f   |  宽度9，默认精度   |
|  %.2f  |  默认宽度，精度2   |
| %9.2f  |    宽度9，精度2    |
|  %9.f  |    宽度9，精度0    |

```go
package main

import "fmt"

func main() {

	n := 12.34
	fmt.Printf("%f\n", n)
	fmt.Printf("%9f\n", n)
	fmt.Printf("%.2f\n", n)
	fmt.Printf("%9.2f\n", n)
	fmt.Printf("%9.f\n", n)

}
$ go run main.go
12.340000
12.340000
12.34
    12.34
       12
```



### 其他flag

| 占位符 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|  ’+’   | 总是输出数值的正负号；对%q（%+q）会生成全部是ASCII字符的输出（通过转义）； |
|  ’ ‘   | 对数值，正数前加空格而负数前加负号；对字符串采用%x或%X时（% x或% X）会给各打印的字节之间加空格 |
|  ’-’   | 在输出右边填充空白而不是默认的左边（即从默认的右对齐切换为左对齐）； |
|  ’#’   | 八进制数前加0（%#o），十六进制数前加0x（%#x）或0X（%#X），指针去掉前面的0x（%#p）对%q（%#q），对%U（%#U）会输出空格和单引号括起来的go字面值； |
|  ‘0’   | 使用0而不是空格填充，对于数值类型会把填充的0放在正负号后面； |

```go
s := "小王子"
fmt.Printf("%s\n", s)
fmt.Printf("%5s\n", s)
fmt.Printf("%-5s\n", s)
fmt.Printf("%5.7s\n", s)
fmt.Printf("%-5.7s\n", s)
fmt.Printf("%5.2s\n", s)
fmt.Printf("%05s\n", s)

小王子
  小王子
小王子  
  小王子
小王子  
   小王
00小王子
```



## 获取输入

fmt包中：有fmt.Scan fmt.Scanf fmt.Scanln三个函数，可以在程序运行中从标准输入获取用户输入

### fmt.Scan

函数签名

```go
func Scan(a ...interface{}) (n int, err error)
```

- Scan从标准输入扫描文本，读取由空白符分隔的值，保存到传递给本函数的参数中，换行符视为空白符
- 返回成功扫描的数据个数，和遇到的错误，

e.g.

```go
package main

import "fmt"

func main() {
	var (
		name    string
		age     int
		married bool
	)

	fmt.Scan(&name, &age, &married)

	fmt.Printf("扫描结果： name: %s age : %d married: %t \n", name, age, married)
}
$ ./25fmt.exe
xiaowang 24 false
扫描结果： name: xiaowang age : 24 married: false

$ ./25fmt.exe
haha 25
true
扫描结果： name: haha age : 25 married: true
中间有换行符不影响，
```



### fmt.Scanf

函数签名：

```go
func Scanf(format string, a ...interface{}) ( n int, err error)
```

- 从标准输入扫描文件，根据format的格式，去读取由空白符分隔的值，保存到传递给本函数的参数中

e.g.

**可以看到，必须按照format中的格式，把除去占位符的部分也输入，才可以正常获取到值**

```go
package main

import "fmt"

func main() {
	var (
		name    string
		age     int
		married bool
	)

	fmt.Scanf("1:%s 2:%d 3:%t", &name, &age, &married)
	fmt.Printf("name : %s age :%d married :%t\n", name, age, married)
}

$ ./25fmt.exe
haha 18 true
name :  age :0 married :false


$ ./25fmt.exe
1: wang 2:18 3:false
name : wang age :18 married :false
```



### fmt.Scanln

函数签名

```go
func Scanln(a ...interface{}) (n int, err error)
```

- Scanln类似Scan，它在遇到换行时才停止扫描。最后一个数据后面必须有换行或者到达结束位置。

e.g.

```go
package main

import "fmt"

func main() {
	var (
		name    string
		age     int
		married bool
	)

	fmt.Scanln(&name, &age, &married)
	fmt.Printf("name : %s age :%d married :%t\n", name, age, married)
}

遇到回车符就结束，没有输入的对应值，为默认的零值
$ ./25fmt.exe
wang
name : wang age :0 married :false


$ ./25fmt.exe
wang          18      false
name : wang age :18 married :false
```



### bufio.NewReader

若想要输入的内容中，包含空格，**此时用Scanln就非常不便**，此时可以用bufio包实现

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	// 从标准输入获取一个读对象reader
	reader := bufio.NewReader(os.Stdin)
	fmt.Print("请输入内容")
	// 利用读对象的ReadString方法，按行读取
	content, _ := reader.ReadString('\n')
	fmt.Printf("%#v\n", content)
}
$ ./25fmt.exe
请输入内容xixi haha shuaiqi lueluelue
"xixi haha shuaiqi lueluelue\r\n"
```



### Fscan系列

该系列函数类似于Scan系列，但不是从标准输入中读取，而是从io.Reader类型的对象中读取

```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
```



### Sscan系列

该系列函数类似于Scan系列，但不是从标准输入，而是从指定字符串中读取数据

```go
func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
```





