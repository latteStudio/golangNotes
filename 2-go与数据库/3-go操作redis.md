# redis介绍

## 参考文档

https://pkg.go.dev/github.com/go-redis/redis

Redis是一个开源的内存数据库，Redis提供了多种不同类型的数据结构，很多业务场景下的问题都可以很自然地映射到这些数据结构上。除此之外，通过复制、持久化和客户端分片等特性，我们可以很方便地将Redis扩展成一个能够包含数百GB数据、每秒处理上百万次请求的系统。

## redis支持的数据结构

redis支持的数据结构体有：

- strings字符串
- hashes哈希值
- lists列表
- sets集合
- sorted sets带范围查询的排序集合
- bit maps位图
- hyperloglogs
- geospatial indexes带半径查询和流的地理空间索引

## redis应用场景

- 缓存系统：减轻数据库的压力，如mysql
- 计数场景，如微博、抖音的关注数
- 热门排行榜，如热搜，使用Zset
- 消息队里，利用list可以实现队列功能

## redis搭建

安装

```
[root@host1 ~]# yum install -y redis
[root@host1 ~]# systemctl enable redis
[root@host1 ~]# systemctl start redis
```

修改配置，修改为对外监听，并添加访问密码

```
[root@host1 ~]# vim /etc/redis.conf 

61   bind 127.0.0.1 192.168.80.100

480  requirepass 123456


[root@host1 ~]# systemctl restart redis
[root@host1 ~]# ss -nlt|grep 6379
LISTEN     0      128    192.168.80.100:6379                     *:*                  
LISTEN     0      128    127.0.0.1:6379                     *:*   
```

连接测试

```
[root@host1 ~]# redis-cli -a 123456 -h 192.168.80.100
192.168.80.100:6379> 
```



# go-redis库

## 安装

[官方仓库地址](https://github.com/go-redis/redis)，且go-redis支持哨兵和集群模式的redis集群，[redigo](https://github.com/gomodule/redigo)也是个连接redis的go第三方库

这里采用的是go-redis库的v8版本；[参考文档](https://redis.uptrace.dev/)

1、利用go module方式管理，初始化一个module

```
$ go mod init github.com/latteplus/redis-cli-goDemo
go: creating new go.mod: module github.com/latteplus/redis-cli-goDemo
```

2、安装依赖包

```

go get github.com/go-redis/redis/v8
```



## 连接

### v8版本的连接示例

直接用官方的文档：https://github.com/go-redis/redis#quickstart

```go
package main

import (
	"context"
	"fmt"

	"github.com/go-redis/redis/v8"
)

var ctx = context.Background()

func ExampleClient() {
	rdb := redis.NewClient(&redis.Options{
		Addr:     "192.168.80.100:6379",
		Password: "123456", // redis password set
		DB:       1,        // use  DB 1
	})

	err := rdb.Set(ctx, "key", "value", 0).Err()
	if err != nil {
		panic(err)
	}

	val, err := rdb.Get(ctx, "key").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("key", val)

	val2, err := rdb.Get(ctx, "key2").Result()
	if err == redis.Nil {
		fmt.Println("key2 does not exist")
	} else if err != nil {
		panic(err)
	} else {
		fmt.Println("key2", val2)
	}
	// Output: key value
	// key2 does not exist
}

func main() {
	ExampleClient()
}

```

运行结果：

```
$ ./redis-cli-goDemo.exe
key value
key2 does not exist
```

```redis
192.168.80.100:6379> select 1
OK
192.168.80.100:6379[1]> KEYS *
(empty list or set)
192.168.80.100:6379[1]> KEYS *
1) "key"
192.168.80.100:6379[1]> get key
"value"
```



## V8版本相关

### 连接redis哨兵模式

https://redis.uptrace.dev/sentinel/

```go
func initClient()(err error){
	rdb := redis.NewFailoverClient(&redis.FailoverOptions{
		MasterName:    "master",
		SentinelAddrs: []string{"x.x.x.x:26379", "xx.xx.xx.xx:26379", "xxx.xxx.xxx.xxx:26379"},
	})
	_, err = rdb.Ping().Result()
	if err != nil {
		return err
	}
	return nil
}
```



### 连接redis集群

https://redis.uptrace.dev/cluster/

```go
func initClient()(err error){
	rdb := redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: []string{":7000", ":7001", ":7002", ":7003", ":7004", ":7005"},
	})
	_, err = rdb.Ping().Result()
	if err != nil {
		return err
	}
	return nil
}
```



## 基本使用

### set/get

```go
package main

import (
	"context"
	"fmt"

	"github.com/go-redis/redis/v8"
)

var ctx = context.Background()
var rdb *redis.Client

func initClient() {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "192.168.80.100:6379",
		Password: "123456", // redis password set
		DB:       1,        // use  DB 1
	})

}

func redisDemo() {
	err := rdb.Set(ctx, "score", 100, 0).Err()
	if err != nil {
		fmt.Printf("set score failed, err:%v\n",err)
		return
	}
	val, err := rdb.Get(ctx,"score").Result()
	if err != nil {
		fmt.Printf("get socre failed ,err:%v\n",err)
		return
	}

	fmt.Println("score:",val)

	val2, err := rdb.Get(ctx,"name").Result()
	if err == redis.Nil{
		fmt.Println("name not exist")
	}else if err != nil {
		fmt.Println("get name failed,",err)
		return
	}else {
		fmt.Println("name",val2)
	}
}
func main() {
	initClient()
	fmt.Println("init redis client success!")

	redisDemo()
}

```

执行结果：

```
D:\myCode\Go\src\learngo\database\redis>redis-cli-goDemo.exe
init redis client success!
score: 100
name not exist

192.168.80.100:6379[1]> KEYS *
2) "score"
3) "key"
192.168.80.100:6379[1]> get score
"100"
```



### zset

```go
package main

import (
	"context"
	"fmt"

	"github.com/go-redis/redis/v8"
)

var ctx = context.Background()
var rdb *redis.Client

func initClient() {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "192.168.80.100:6379",
		Password: "123456", // redis password set
		DB:       1,        // use  DB 1
	})

}

func redisDemo() {
	// 构造zset中的键名，和其成员
	zsetKey := "language_rank"
		languages := []*redis.Z{
			&redis.Z{Score: 90.0, Member: "Golang"},
			&redis.Z{Score: 98.0, Member: "Java"},
			&redis.Z{Score: 95.0, Member: "Python"},
			&redis.Z{Score: 97.0, Member: "JavaScript"},
			&redis.Z{Score: 99.0, Member: "C/C++"},
		}

//	Zadd

	num, err := rdb.ZAdd(ctx,zsetKey,languages...).Result()
	if err != nil {
		fmt.Printf("zadd failed, err : %v\n", err)
		return
	}

	fmt.Printf("zadd %d success!", num)

//	把Golang的分数加10
	newScore, err := rdb.ZIncrBy(ctx, zsetKey, 10, "Golang").Result()
	if err != nil {
		fmt.Printf("zincrby failed, err :%v\n", err)
		return
	}
	fmt.Printf("Golang`s score is %f now.", newScore)

//	取分数最高的三个

	ret, err := rdb.ZRevRangeWithScores(ctx, zsetKey, 0, 2).Result()
	if err != nil {
		fmt.Printf("zrevrange failed, err :%v\n", err)
		return
	}

	for _, z := range ret{
		fmt.Println(z.Member, z.Score)
	}


//	取95-100分的
	op := redis.ZRangeBy{
		Min: "95",
		Max: "100",
	}
	ret, err = rdb.ZRangeByScoreWithScores(ctx, zsetKey, &op).Result()
	if err != nil {
		fmt.Printf("zrangbyScore failed, err:%v\n", err)
		return
	}
	for _, z := range ret{
		fmt.Println(z.Member, z.Score)
	}

}

func main() {
	initClient()
	fmt.Println("init redis client success!")

	redisDemo()
}


```

运行结果：

```
D:\myCode\Go\src\learngo\database\redis>redis-cli-goDemo.exe
init redis client success!
zadd 5 success!Golang`s score is 100.000000 now.Golang 100
C/C++ 99
Java 98
Python 95
JavaScript 97
Java 98
C/C++ 99
Golang 100
```

遇到的问题：

35和69行处，需要传递的参数变为的指针类型，需要注意；redis-go的V8版本和之前版本的变动导致

```
D:\myCode\Go\src\learngo\database\redis>go build
# github.com/latteplus/redis-cli-goDemo
.\main.go:35:22: cannot use languages (type []redis.Z) as type []*redis.Z in argument to rdb.cmdable.ZAdd
.\main.go:69:40: cannot use op (type redis.ZRangeBy) as type *redis.ZRangeBy in argument to rdb.cmdable.ZRangeByScoreWithScores


```



### 根据前缀获取key

```go
vals, err := rdb.Keys(ctx, "prefix*").Result()
```



### 执行自定义命令

```go
res, err := rdb.Do(ctx, "set", "key", "value").Result()
```



### 根据通配符删除key

```go
iter := rdb.Scan(ctx, 0, "prefix*", 0).Iterator()
for iter.Next(ctx) {
	err := rdb.Del(ctx, iter.Val()).Err()
	if err != nil {
		panic(err)
	}
}

if err := iter.Err(); err != nil {
	panic(err)
}
```



### Pipline

pipline可以用于优化网络传输时间，本质是将多条发往redis-server的命令，积攒在一次进行发送，节省了每个命令的rtt值

pipline基本示例

```go
pipe := rdb.Pipeline()
incr := pipe.Incr(ctx,"pipline_counter")
pipe.Expire(ctx, "pipline_counter", time.Hour)
_, err := pipe.Exec()
fmt.Println(incr.Val(), err)
```

也可以使用piplined，基本示例

```go
var incr *redis.IntCmd
_, err := rdb.Pipelined(func(pipe redis.Pipeliner) error {
	incr = pipe.Incr("pipelined_counter")
	pipe.Expire("pipelined_counter", time.Hour)
	return nil
})
fmt.Println(incr.Val(), err)
```

**某些场景下，当有多条命令需要执行时，可以考虑用pipline来优化。**

### 事务

redis是单线程的，因此单个命令使用是原子的，但来自不同客户端的给定命令可以依次执行，利用在它们之间交替执行，但是`Multi/exec`可以确保在这2个语句之间执行的命令没有其他客户端正在执行命令；

在这种场景我们需要使用`TxPipeline`。`TxPipeline`总体上类似于上面的`Pipeline`，但是它内部会使用`MULTI/EXEC`包裹排队的命令。例如：

```go
pipe := rdb.TxPipeline()

incr := pipe.Incr("tx_pipeline_counter")
pipe.Expire("tx_pipeline_counter", time.Hour)

_, err := pipe.Exec()
fmt.Println(incr.Val(), err)
```

上面代码相当于在一个RTT下执行了下面的redis命令：

```bash
MULTI
INCR pipeline_counter
EXPIRE pipeline_counts 3600
EXEC
```

还有一个与上文类似的`TxPipelined`方法，使用方法如下：

```go
var incr *redis.IntCmd
_, err := rdb.TxPipelined(func(pipe redis.Pipeliner) error {
	incr = pipe.Incr("tx_pipelined_counter")
	pipe.Expire("tx_pipelined_counter", time.Hour)
	return nil
})
fmt.Println(incr.Val(), err)
```

### Watch

在某些场景下，我们除了要使用`MULTI/EXEC`命令外，还需要配合使用`WATCH`命令。在用户使用`WATCH`命令监视某个键之后，直到该用户执行`EXEC`命令的这段时间里，如果有其他用户抢先对被监视的键进行了替换、更新、删除等操作，那么当用户尝试执行`EXEC`的时候，事务将失败并返回一个错误，用户可以根据这个错误选择重试事务或者放弃事务。

```go
Watch(fn func(*Tx) error, keys ...string) error
```

Watch方法接收一个函数和一个或多个key字符串类型，作为参数；示例：

```go
// 监视watch_count的值，并在值不变的前提下将其值+1

key := "watch_count"
err = client.Watch(func(tx *redis.Tx) error {
    n, err := tx.Get(key).Int()
    if err != nil && err != redis.Nil {
        return err
    }
    _, err = tx.Pipelined(func(pipe, redis.Pipeliner) error {
        pipe.Set(key, n+1, 0)
        return nil
    })
    return err
},key)
```

v8版本中，官方文档示例：使用GET和SET命令以事务的方式递增Key值的示例，仅仅当Key的值不发生变化时提交一个事务

```go
func transactionDemo() {
	var (
		maxRetries   = 1000
		routineCount = 10
	)
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// Increment 使用GET和SET命令以事务方式递增Key的值
	increment := func(key string) error {
		// 事务函数
		txf := func(tx *redis.Tx) error {
			// 获得key的当前值或零值
			n, err := tx.Get(ctx, key).Int()
			if err != nil && err != redis.Nil {
				return err
			}

			// 实际的操作代码（乐观锁定中的本地操作）
			n++

			// 操作仅在 Watch 的 Key 没发生变化的情况下提交
			_, err = tx.TxPipelined(ctx, func(pipe redis.Pipeliner) error {
				pipe.Set(ctx, key, n, 0)
				return nil
			})
			return err
		}

		// 最多重试 maxRetries 次
		for i := 0; i < maxRetries; i++ {
			err := rdb.Watch(ctx, txf, key)
			if err == nil {
				// 成功
				return nil
			}
			if err == redis.TxFailedErr {
				// 乐观锁丢失 重试
				continue
			}
			// 返回其他的错误
			return err
		}

		return errors.New("increment reached maximum number of retries")
	}

	// 模拟 routineCount 个并发同时去修改 counter3 的值
	var wg sync.WaitGroup
	wg.Add(routineCount)
	for i := 0; i < routineCount; i++ {
		go func() {
			defer wg.Done()
			if err := increment("counter3"); err != nil {
				fmt.Println("increment error:", err)
			}
		}()
	}
	wg.Wait()

	n, err := rdb.Get(context.TODO(), "counter3").Int()
	fmt.Println("ended with", n, err)
}
```

