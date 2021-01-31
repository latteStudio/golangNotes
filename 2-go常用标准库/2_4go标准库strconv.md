# strconv包

strconv包的作用：**实现了基本数据类型和其字符串表示的相互转换的方法**

如“100”转为int 的100（从文件中读取的100往往就是字符串类型，存储到变量时，应该先转为int类型

[官方文档](https://golang.org/pkg/strconv/)



## string和int类型转换

### Atoi()

将字符串类型的整数转为int类型，函数定义如下：

```go
func Atoi(s string) (i int, err error)
```

若传入的字符串参数无法转为int类型，就会返回错误

```go
s1 := "100"
i1, err := strconv.Atoi(s1)

if err != nil {
    fmt.Println("can`t convert to int")
} else {
    fmt.Printf("type: %T value: %#v \n", i1, i1) // type:int value:100
}
```



### Itoa()

将int类型数据转为对应的字符串表示，函数定义如下：

```go
func Itoa(i int) string

```

e.g.

```go
i2 := 200
s2 := strconv.Itoa(i2)
fmt.Printf("type:%T value:%#v\n", s2 ,s2) // type:string value :"200"
```





### a的由来

a在c语言中表示字符数据array，**而c中没有字符串，只有字符数组，而go又很像c，因此Itoa，a就是代表字符串的意思**



## Parse系列函数

Parse系列函数用于将字符串转为给定类型的值；有：

- ParseBool()
- ParseFloat()
- ParesInt()
- ParseUint()

### ParseBool()

函数定义

```go
func ParseBool (str string) (value bool ,err error)
```

返回的value是传入的str所代表的bool值，可以接收的str有 1、 0、 t、 f、 T、 F、 true、false、True、False、TRUR、FALSE

### ParseInt()

```go
func ParseInt(s string, base int, bitSize int) (i int64 , err error)
```

- 返回的i为字符串所代表的整数值，接受正负号
- base指定进制2到36，若base是0，则从字符串的前缀判断，“ox”是16进制、“o”是八进制、否则是10进制
- bitSiez指定结果必须能无溢出赋值的整数类型，0 8 16 32 64 分别 int、in8 、int16、 int32、int64
- 返回的err是*NumErr类型，若语法有误，err.Error=ErrSynat，如果结果超出类型范围，err.Error = ErrRange
- 

### ParseUnit()

```go
func ParseUint (s string, base int, bitSize int ) (n uint64, err error)
```

ParseUint类似ParseInt，但不接收正负号，用于无符号整型

### ParseFloat()

```go
func ParseFloat(s string, bitSize int) (f float64 , err error)
```

- 解析一个表示浮点数的字符串，并返回其值
- 若s合乎语法规则，函数会返回最为接近s表示值的一个浮点数
- bitSize指定了期望的接收类型，32代表float32,64代表float64
- 返回值err是*NumErr类型，语法有误，err.Error = ErrSyntax，结果超出表示范围的，返回值为正负Inf，err.Error = ErrRange

### 代码示例

```go
b, err := strconv.ParseBool("true")

f, err := strconv.ParseFloat("3.1415", 64)

i, err := strconv.ParseInt("-2", 10, 64)

u, err := strconv.ParseUint("2", 10, 64)
```

这些函数都有2个返回值，第一个是转换后的值，第二个是转化失败的化的提示信息

## Format系列函数

Format系列函数，实现了将给定类型数据格式化为string类型数据的功能

### FormatBool()

```go
func FormatBool(b bool) string
```

根据b的值，返回“true”或“false”

### FormatInt()

```go
func FormatInt(i int64 , base int) string
```

返回i的base进制的字符串表示，base必须在2到36之间，结果中会使用小写字母“a”到“z”表示大于10 的数字

### FormatUint()

```go
func FormatUint( i uint64, base int) string
```

FormatInt()的无符号整数版本

### FormatFloat()

```go
func FormatFloat(f float64, fmt byte ,prec, bitSize int) string
```

- 函数将浮点数表示为字符串并返回
- bitSize表示f的类型，32对应float32,64对应float64
- fmt表示格式，
- prec控制精度
- 参考官方解释：https://golang.org/pkg/strconv/#FormatFloat

### 代码示例

```go
s1 := strconv.FormatBool(true)
s2 := strconv.FormatFloat(3.1415, 'E', -1, 64)
s3 := strconv.FormatInt(-2, 16) 
s4 := strconv.FormatUint(2, 16)
```



## 其他

### isPrint()

```go
func IsPrint(r rune) bool
```

返回一个字符是否是可以打印的，和unicode.IsPrint一样，r必须是：字母（广义）、数字、标点、符号、ascll空格

### CanBackquote()

```go
func CanBackquote(s string) bool
```

返回字符串s是否可以不被修改的表示一个单行的、没有空格和tab之外控制字符的反引号字符串

### 其他

除了以上函数外，strconv包，还有Appen系列，Quote系列等，[参见官方的标准库介绍](https://golang.org/pkg/strconv/)





