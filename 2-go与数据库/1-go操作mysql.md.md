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
	_ "github.com/go-sql-driver/mysql"
)

func main()  {
	// DSN:data source name
	dsn: = "root:root@tcp(127.0.0.1:3306)/db1"
	db, err := sql.Open("mysql", dsn)
	if err != nil {
		panic(err)
	}
	defer db.Close()
}
```



### 初始化连接

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

