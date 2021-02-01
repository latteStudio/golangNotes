# time包

go内置的time提供了时间相关的函数

## 时间类型

time.Time类型表示时间，

**通过time.Now()获得当前时间的时间对象，然后利用该对象的各种方法，获取年月日等信息**

e.g.

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	// 获得当前的时间对象

	nowTimeObj := time.Now()

	// 利用对象的各种方法，获得年月日等信息

	year := nowTimeObj.Year()
	month := nowTimeObj.Month()
	day := nowTimeObj.Day()

	hour := nowTimeObj.Hour()
	min := nowTimeObj.Minute()
	sec := nowTimeObj.Second()

	fmt.Printf("%d-%02d-%02d %02d:%02d:%02d\n", year, month, day, hour, min, sec)
}
latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/14time (master)
$ go run main.go
2021-01-22 17:23:12
```





## 时间戳

时间戳是1970年，1月1至今的秒数，又叫做unixTimestamp

1、时间格式转为时间戳

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	// 获得当前的时间对象

	nowTimeObj := time.Now()

	// 时间戳获得
	timeStamp1 := nowTimeObj.Unix()
	timeStamp2 := nowTimeObj.UnixNano() // 精确到纳秒

	fmt.Println(timeStamp1)
	fmt.Println(timeStamp2)

}

latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/14time (master)
$ go run main.go
1611307814
1611307814421148300
```



2、时间戳转为时间格式

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	// time.Unix将时间戳转为时间对象
	time1 := time.Unix(1611307814, 0)

	// 获取时间对象的年和月
	year := time1.Year()
	month := time1.Month()

	fmt.Printf("%d-%02d\n", year, month)
}

latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/14time (master)
$ go run main.go
2021-01

```



## 时间间隔

time包中的Duration类型代表时间间隔，表示2个时间点之间的时间间隔，**单位是纳秒**

定义：

```go
type Duration int64
// 本质就是基于int64定义的自定义类型，存储形式还是int64的整型

const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)
```

`一个Duration就是1纳秒，time.Duration就是1纳秒；time.Second就是1s，time.Hour就是1小时`

## 时间操作

### Add

语法：

```go
func (t Time) Add(d Duration) Time
```

求一个小时后的时间

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	now := time.Now()

	fmt.Println(now)

	// add
	later := now.Add(time.Hour)

	fmt.Println(later)

	
}

latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/14time (master)
$ go run main.go
2021-01-25 15:04:55.9893984 +0800 CST m=+0.002014401
2021-01-25 16:04:55.9893984 +0800 CST m=+3600.002014401
```



### Sub

语法：求2个时间点之间的间隔

```go
func (t Time) Sub(u Time) Duration
// 调用某个时间对象的Sub方法，再传入一个时间类型，返回的Duration，就是这2个时间点的时间间隔
// 若要获取t-d这个时间点，用t.Add(-d)就行
```



### Equal

语法：

```go
func (t Time) Equal(u Time) bool
// 可以判断t和传入的u，2个时间点是否相等，是则返回true，（且会考虑事时区的因素
```



### Before

语法：

```go
func (t Time) Before(u Time) bool
// t若在u时间点之前，则返回true
```



### After

语法：

```go
func (t Time) After(u Time) bool
// t在u时间点之后，返回true
```



## 定时器

使用`time.Tick时间间隔`可以设置定时器，定时器本质是一个通道(channel)

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	ticker := time.Tick(time.Second)
	// 设置一个间隔一秒的定时器，效果是每秒都会执行打印任务
	for i := range ticker {
		fmt.Println(i)
	}
}

latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/14time (master)
$ go run main.go
2021-01-25 15:24:15.9191352 +0800 CST m=+1.002174601
2021-01-25 15:24:16.9195181 +0800 CST m=+2.002557501
2021-01-25 15:24:17.9198929 +0800 CST m=+3.002932301
2021-01-25 15:24:18.9194266 +0800 CST m=+4.002466001
2021-01-25 15:24:19.920871 +0800 CST m=+5.003910401
```



## 时间格式化

**go中时间格式化的格式表示，不是用Y-m-d 这些，而是用go的诞生时间2006年的1月2号15时04分，是周一，（记忆口诀为2006 1 2 3（下午三点） 4**

例子：

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	now := time.Now()

	// now这个时间对象，按照Format中传入的时间格式，打印
	// 秒后面的000，表示，将当前时间的毫秒数也打印
	fmt.Println(now.Format("2006-01-02 15:04:00.000 Mon Jan"))
	// 按照12小时制输出
	fmt.Println(now.Format("2006 01 02 03:04:00.000 PM Mon Jan"))
	fmt.Println(now.Format("2006 01 02"))
	fmt.Println(now.Format("15:04:00 2006:01:02"))
}
latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/14time (master)
$ go run main.go
2021-01-25 15:35:00.982 Mon Jan
2021 01 25 03:35:00.982 PM Mon Jan
2021 01 25
15:35:00 2021:01:25
```



### 解析字符串格式的时间

时间对象之间做操作时，比如取间隔，**要考虑到时区的因素，可以用time.LoadLocation加载需要的时区**

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	now := time.Now()
	fmt.Println(now)

	// 加载时区为上海，之前操作的时间字符串，都可以传入该loc，将其当作亚洲东八区来处理
	loc, err := time.LoadLocation("Asia/Shanghai")
	if err != nil {
		fmt.Println(err)
		return
	}

	// 将中间的时间字符串，按照第一个参数格式解析，按照第三个参数时区处理
	tommorrowTimeObj, err := time.ParseInLocation("2006/01/02 15:04:00", "2021/01/26 15:45:00", loc)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(tommorrowTimeObj)
	fmt.Println(tommorrowTimeObj.Sub(now))

}

```

# 练习

1、获取当前时间，格式化输出为2017/06/19 20:30:05`格式。

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	now := time.Now()
	fmt.Println(now)

	fmt.Println(now.Format("2006/01/02 15:04:00"))

}

```





2、编写程序统计一段代码的执行耗时时间，单位精确到微秒。

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	begin := time.Now()
	fmt.Println(begin)
	sum := 0
	for i := 0; i < 10000000; i++ {
		sum += i
	}

	end := time.Now()
	fmt.Println(end)
	fmt.Println(end.Sub(begin))

}
latteplus@LAPTOP-00EFC09V MINGW64 /d/myCode/Go/src/learngo/basic_grammar/14time (master)
$ go run main.go
2021-01-25 16:09:21.9936318 +0800 CST m=+0.001995701
2021-01-25 16:09:22.0066199 +0800 CST m=+0.014983801
12.9881ms
```



