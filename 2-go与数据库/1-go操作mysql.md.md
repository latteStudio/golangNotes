# go操作mysql

## 连接

golang中，`database/sql`包提供了保证sql或类sql数据库的泛用接口，并不提供具体的数据库驱动，相当于定义了一个通用规范，使用`database/sql`包时必须注入一个数据库驱动（其必须实现sql包定义的接口规范）

常用的数据库都有现成的连接库实现：[如myql驱动](https://github.com/go-sql-driver/mysql)

###　下载依赖

```
$ go get -u github.com/go-sql-driver/mysql
```



### 使用mysql驱动



Open函数签名：

```go
func Open(driverName, dataSourceName string) (*DB, error)
```

其中：

- driveName表示数据库类型，mysql，pgsql这种
- dataSourceName表示数据库连接字符串，ip端口用户名密码这些
- 返回一个DB类型的指针类型，和可能的error

连接示例：

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
)

func main() {
	// DSN:data source name
	dsn := "root:123456@tcp(192.168.80.100:3306)/mysql"
	db, err := sql.Open("mysql", dsn)
	if err != nil {
		panic(err)
	}
	defer db.Close()
	fmt.Println("connect mysql is successful")
}
$ ./mysql.exe
connect mysql is successful
```



### 初始化连接

e.g.

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
)

var db *sql.DB

func initDB() (err error) {
	dsn := "root:123456@tcp(192.168.80.100:3306)/db1?charset=utf8mb4&parseTime=True"
	db, err = sql.Open("mysql", dsn)
	if err != nil {
		return err

	}

	err = db.Ping()
	if err != nil {
		return err
	}
	return nil
}

func main() {
	err := initDB()
	if err != nil {
		fmt.Println("init db failed", err)
		return
	}
	fmt.Println("init db success!")
}


$ ./mysql.exe
init db failed Error 1130: Host '192.168.80.1' is not allowed to connect to this MariaDB server


因为mysql默认没有放行程序所在ip，所以不通
放行
MariaDB [(none)]> grant all on *.* to root@'192.168.80.%' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.01 sec)


再运行
$ ./mysql.exe
init db success!
```



### SetMaxOpenConns

### SetMaxIdleConns

## CRUD

### 建库建表

### 查询

### 插入数据

### 更新数据

### 删除数据

## mysql预处理

### 什么是预处理

### 为什么要预处理

### go实现mysql预处理

### sql注入

## go实现mysql事务

### 什么是事务

### 事务的ACID

### 事务相关方法

### 事务示例

# 练习

