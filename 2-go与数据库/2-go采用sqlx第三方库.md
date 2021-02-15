# sqlx介绍

sqlx可以认为是`database/sql`的超集，在其之上提供了一组扩展，包括Get，Select等查询相关的方法

[参考链接](http://jmoiron.github.io/sqlx/)



# sqlx安装

```go
go get github.com/jmoiron/sqlx
```



# sqlx基本使用

## 连接数据库

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

var db *sqlx.DB

func initDB() (err error) {
	dsn := "root:123456@tcp(192.168.80.100:3306)/sql_test?charset=utf8mb4&parseTime=True"
	db, err = sqlx.Connect("mysql", dsn)
	if err != nil {
		return err
	}

	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(10)
	return nil
}

func main() {
	err := initDB()
	if err != nil {
		fmt.Println("init db failed,", err)
		return
	}
	fmt.Println("conncet database sucess!")
}



$ ./sqlx.exe
conncet database sucess!
```



## 查询

**查询单行：**

注意结构体user的字段要首字母大写，对外可见；因为要被sqlx中的db对象的Get方法访问，

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

type user struct {
	Id   int
	Name string
	Age  int
}

var db *sqlx.DB

func queryDemo() {
	sqlStr := `select id ,name ,age from user where id = ?`
	var u user
	err := db.Get(&u, sqlStr, 1)
	if err != nil {
		fmt.Println("get failed", err)
		return
	}
	fmt.Printf("%v, %v, %v \n", u.Id, u.Name, u.Age)

}
func initDB() (err error) {
	dsn := "root:123456@tcp(192.168.80.100:3306)/sql_test?charset=utf8mb4&parseTime=True"
	db, err = sqlx.Connect("mysql", dsn)
	if err != nil {
		return err
	}

	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(10)
	return nil
}

func main() {
	err := initDB()
	if err != nil {
		fmt.Println("init db failed,", err)
		return
	}
	fmt.Println("conncet database sucess!")

	queryDemo()
}


$ go run main.go
conncet database sucess!
id:1 name:wang age:18
```

**查询多行：**

查询多行，用db对象的Select方法

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

type user struct {
	Id   int
	Name string
	Age  int
}

var db *sqlx.DB


func queryMultiDemo() {
	sqlStr := `select id, name, age from user where id > ?`
	var u []user
	err := db.Select(&u, sqlStr, 1)
	if err != nil {
		fmt.Println("query failed:", err)
		return
	}

	fmt.Println(u)
}
func initDB() (err error) {
	dsn := "root:123456@tcp(192.168.80.100:3306)/sql_test?charset=utf8mb4&parseTime=True"
	db, err = sqlx.Connect("mysql", dsn)
	if err != nil {
		return err
	}

	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(10)
	return nil
}

func main() {
	err := initDB()
	if err != nil {
		fmt.Println("init db failed,", err)
		return
	}
	fmt.Println("conncet database sucess!")

	// queryDemo()
	queryMultiDemo()
}


$ go run main.go
conncet database sucess!
[{2 li 22} {3 gu 24}]

```



## 插入、更新、删除

插入、更新、删除和原生的sql库中的使用方法几乎一致；

**示例如下：**

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

type user struct {
	Id   int
	Name string
	Age  int
}

var db *sqlx.DB

func testSqlxInsertUpdateDelte() {
	sqlInsert := `insert user(name, age) values(?, ?)`
	ret1, err := db.Exec(sqlInsert, "zhangsan", 100)
	if err != nil {
		fmt.Println("insert failed:", err)
		return
	}
	lastId, err := ret1.LastInsertId()
	if err != nil {
		fmt.Println("get lastId failed:", err)
		return
	}
	fmt.Println("insert id is :", lastId)

	sqlUpdate := `update user set age=? where id = ?`
	ret2, err := db.Exec(sqlUpdate, 666, 1)
	if err != nil {
		fmt.Println("update failed: ", err)
		return
	}
	affectRow2, err := ret2.RowsAffected()
	if err != nil {
		fmt.Println("get affectRow2 failed:", err)
		return
	}
	fmt.Println("affectRow2 is : ", affectRow2)

	sqlDelete := `delete from user where id = ?`
	ret3, err := db.Exec(sqlDelete, 2)
	if err != nil {
		fmt.Println("delete failed:", err)
		return
	}
	affectRow3, err := ret3.RowsAffected()
	if err != nil {
		fmt.Println("get rowAffect3 failed:", err)
		return
	}
	fmt.Println("affectRow3 is :", affectRow3)

}
func initDB() (err error) {
	dsn := "root:123456@tcp(192.168.80.100:3306)/sql_test?charset=utf8mb4&parseTime=True"
	db, err = sqlx.Connect("mysql", dsn)
	if err != nil {
		return err
	}

	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(10)
	return nil
}

func main() {
	err := initDB()
	if err != nil {
		fmt.Println("init db failed,", err)
		return
	}
	fmt.Println("conncet database sucess!")

	testSqlxInsertUpdateDelte()

}

```

执行结果：

```
$ ./sqlx.exe
conncet database sucess!
insert id is : 4
affectRow2 is :  1
affectRow3 is : 1


MariaDB [sql_test]> select * from user;
+----+------+------+
| id | name | age  |
+----+------+------+
|  1 | wang |   18 |
|  2 | li   |   22 |
|  3 | gu   |   24 |
+----+------+------+
3 rows in set (0.00 sec)

MariaDB [sql_test]> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | wang     |  666 |
|  3 | gu       |   24 |
|  4 | zhangsan |  100 |
+----+----------+------+
3 rows in set (0.00 sec)
```



## NamedExec

*sqlx.DB类型对象的NamedExec()方法用来将要传递给sql字符串的变量进行绑定，根据字段名进行映射，一般是struct和map。

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

type user struct {
	Id   int
	Name string
	Age  int
}

var db *sqlx.DB

func NamedExecDemo() {
	sqlStr := `insert into user(name, age) values(:name, :age)`
	_, err := db.NamedExec(sqlStr, map[string]interface{}{
		"name": "lueluelue",
		"age":  233,
        // name和age会和上面values中的name和age一一对应。
	})
	if err != nil {
		fmt.Println("nameExec failed:", err)
		return
	}
	return
}

func initDB() (err error) {
	dsn := "root:123456@tcp(192.168.80.100:3306)/sql_test?charset=utf8mb4&parseTime=True"
	db, err = sqlx.Connect("mysql", dsn)
	if err != nil {
		return err
	}

	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(10)
	return nil
}

func main() {
	err := initDB()
	if err != nil {
		fmt.Println("init db failed,", err)
		return
	}
	fmt.Println("conncet database sucess!")

	// testSqlxInsertUpdateDelte()
	NamedExecDemo()

}

```

执行后，数据库的变化

```mysql
MariaDB [sql_test]> select * from user;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | wang     |  666 |
|  3 | gu       |   24 |
|  4 | zhangsan |  100 |
+----+----------+------+
3 rows in set (0.00 sec)

MariaDB [sql_test]> select * from user;
+----+-----------+------+
| id | name      | age  |
+----+-----------+------+
|  1 | wang      |  666 |
|  3 | gu        |   24 |
|  4 | zhangsan  |  100 |
|  5 | lueluelue |  233 |
+----+-----------+------+
4 rows in set (0.00 sec)
```



## NamedQuery

和NamedQuery性质一样，只不过是用于查询；

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

type user struct {
	Id   int
	Name string
	Age  int
}

var db *sqlx.DB

func NamedQueryDemo() {
	sqlStr := `select id,name,age from user where name=:name`
	rows, err := db.NamedQuery(sqlStr, map[string]interface{}{"name": "wang"})
	if err != nil {
		fmt.Println("query failed:", err)
		return
	}
	defer rows.Close()
	var u user
	for rows.Next() {
		err = rows.Scan(&u.Id, &u.Name, &u.Age)
		if err != nil {
			fmt.Println("scan failed:", err)
			continue
		}
		fmt.Printf("Id: %v Name: %v, Age: %v\n", u.Id, u.Name, u.Age)
	}
}

func initDB() (err error) {
	dsn := "root:123456@tcp(192.168.80.100:3306)/sql_test?charset=utf8mb4&parseTime=True"
	db, err = sqlx.Connect("mysql", dsn)
	if err != nil {
		return err
	}

	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(10)
	return nil
}

func main() {
	err := initDB()
	if err != nil {
		fmt.Println("init db failed,", err)
		return
	}
	fmt.Println("conncet database sucess!")

	// testSqlxInsertUpdateDelte()
	NamedQueryDemo()

}


$ ./sqlx.exe
conncet database sucess!
Id: 1 Name: wang, Age: 666
```



## 事务操作

sqlx包也可以通过db.Begin()和tx.Rollback tx.Commit()的方法进行事务操作，如下：

```go
func transactionDemo2()(err error) {
	tx, err := db.Beginx() // 开启事务
	if err != nil {
		fmt.Printf("begin trans failed, err:%v\n", err)
		return err
	}
	defer func() {
		if p := recover(); p != nil {
			tx.Rollback()
			panic(p) // re-throw panic after Rollback
		} else if err != nil {
			fmt.Println("rollback")
			tx.Rollback() // err is non-nil; don't change it
		} else {
			err = tx.Commit() // err is nil; if Commit returns error update err
			fmt.Println("commit")
		}
	}()

	sqlStr1 := "Update user set age=20 where id=?"

	rs, err := tx.Exec(sqlStr1, 1)
	if err!= nil{
		return err
	}
	n, err := rs.RowsAffected()
	if err != nil {
		return err
	}
	if n != 1 {
		return errors.New("exec sqlStr1 failed")
	}
	sqlStr2 := "Update user set age=50 where i=?"
	rs, err = tx.Exec(sqlStr2, 5)
	if err!=nil{
		return err
	}
	n, err = rs.RowsAffected()
	if err != nil {
		return err
	}
	if n != 1 {
		return errors.New("exec sqlStr1 failed")
	}
	return err
}
```




# sqlx.In

sql.In是sqlx提供的一个非常方便的函数；

## sqlx.In的批量插入示例

1、创建一张数据表：

```mysql
MariaDB [sql_test]> CREATE TABLE `user` (
    ->     `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
    ->     `name` VARCHAR(20) DEFAULT '',
    ->     `age` INT(11) DEFAULT '0',
    ->     PRIMARY KEY(`id`)
    -> )ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
Query OK, 0 rows affected (0.00 sec)

MariaDB [sql_test]> show tables;
+--------------------+
| Tables_in_sql_test |
+--------------------+
| user               |
+--------------------+
1 row in set (0.00 sec)

```
2、定义一个结构体：字段中通过tag和数据表user的字段进行关联

```go
type User struct {
    Name string `db:"name"`
    Age int `db:"age"`
}
```



**bindvars绑定变量**

查询占位符？在内部称为binvars，代码中应该使用使用它们向数据库传递值，因为这种方式可以防止sql注入，`database/sql`不尝试对查询文本进行任何验证；它与编码的参数一起按原样发送到服务器。除非驱动程序实现一个特殊的接口，否则在执行之前，查询是在服务器上准备的。因此`bindvars`是特定于数据库的:

- mysql中是？
- pgsql中是$1 $2
- sqlite中是$1 ?都支持
- oracel中是:name的方式

binvars仅仅用于参数化，即不可以通过传递的binvars改变sql的结构，这样避免了sql注入问题，**如下面通过binvars改变sql中表名或字段名将不起作用**

```go
db.Query("select * from ?", "table1")

db.Query("select ?,? from people", "name", "location")
```

**自行拼接语句实现批量插入：**

```go
// BatchInsertUsers 自行构造批量插入的语句
func BatchInsertUsers(users []*User) error {
	// 存放 (?, ?) 的slice
	valueStrings := make([]string, 0, len(users))
	// 存放values的slice
	valueArgs := make([]interface{}, 0, len(users) * 2)
	// 遍历users准备相关数据
	for _, u := range users {
		// 此处占位符要与插入值的个数对应
		valueStrings = append(valueStrings, "(?, ?)")
		valueArgs = append(valueArgs, u.Name)
		valueArgs = append(valueArgs, u.Age)
	}
	// 自行拼接要执行的具体语句
	stmt := fmt.Sprintf("INSERT INTO user (name, age) VALUES %s",
		strings.Join(valueStrings, ","))
	_, err := DB.Exec(stmt, valueArgs...)
	return err
}
```



**使用sqlx.In实现批量插入：**



**还可以使用NameExec实现批量插入：**

```go
// BatchInsertUsers3 使用NamedExec实现批量插入
func BatchInsertUsers3(users []*User) error {
	_, err := DB.NamedExec("INSERT INTO user (name, age) VALUES (:name, :age)", users)
	return err
}
```



**这里采用sqlx.In的方式的示例代码：**

```go
package main

import (
	"database/sql/driver"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

type User struct {
	Name string `db:"name"`
	Age  int    `db:"age"`
}

var db *sqlx.DB

func (u User) Value() (driver.Value, error) {
	return []interface{}{u.Name, u.Age}, nil
}

// BatchInsertUsers2 使用sqlx.In帮我们拼接语句和参数, 注意传入的参数是[]interface{}
func BatchInsertUsers2(users []interface{}) error {
	query, args, _ := sqlx.In(
		"INSERT INTO user (name, age) VALUES (?), (?), (?)",
		users..., // 如果arg实现了 driver.Valuer, sqlx.In 会通过调用 Value()来展开它
	)
	fmt.Println(query) // 查看生成的querystring
	fmt.Println(args)  // 查看生成的args
	_, err := db.Exec(query, args...)
	return err
}

func initDB() (err error) {
	dsn := "root:123456@tcp(192.168.80.100:3306)/sql_test?charset=utf8mb4&parseTime=True"
	db, err = sqlx.Connect("mysql", dsn)
	if err != nil {
		return err
	}

	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(10)
	return nil
}

func main() {
	err := initDB()
	if err != nil {
		fmt.Println("init db failed,", err)
		return
	}
	fmt.Println("conncet database sucess!")

	defer db.Close()
	u1 := User{Name: "zhangsan", Age: 18}
	u2 := User{Name: "lisi", Age: 28}
	u3 := User{Name: "wangermazi", Age: 38}

	// 方法2
	users2 := []interface{}{u1, u2, u3}
	err = BatchInsertUsers2(users2)
	if err != nil {
		fmt.Printf("BatchInsertUsers2 failed, err:%v\n", err)
	}

}

```

执行结果：

```
$ ./sqlx.exe
conncet database sucess!
INSERT INTO user (name, age) VALUES (?, ?), (?, ?), (?, ?)
[zhangsan 18 lisi 28 wangermazi 38]

MariaDB [sql_test]> select * from user;
Empty set (0.00 sec)

MariaDB [sql_test]> select * from user;
+----+------------+------+
| id | name       | age  |
+----+------------+------+
|  1 | zhangsan   |   18 |
|  2 | lisi       |   28 |
|  3 | wangermazi |   38 |
+----+------------+------+

```



## sqlx.In的查询示例

关于`sqlx.In`这里再补充一个用法，在`sqlx`查询语句中实现In查询和FIND_IN_SET函数。即实现`SELECT * FROM user WHERE id in (3, 2, 1);`和`SELECT * FROM user WHERE id in (3, 2, 1) ORDER BY FIND_IN_SET(id, '3,2,1');`。

**in查询：**

查询id，在给定id集合中的数据

```go
// QueryByIDs 根据给定ID查询
func QueryByIDs(ids []int)(users []User, err error){
	// 动态填充id
	query, args, err := sqlx.In("SELECT name, age FROM user WHERE id IN (?)", ids)
	if err != nil {
		return
	}
	// sqlx.In 返回带 `?` bindvar的查询语句, 我们使用Rebind()重新绑定它
	query = DB.Rebind(query)

	err = DB.Select(&users, query, args...)
	return
}
```

**in查询和FIND_IN_SET函数**

查询id在给定id集合的数据，并维持给定id集合的顺序

```go
// QueryAndOrderByIDs 按照指定id查询并维护顺序
func QueryAndOrderByIDs(ids []int)(users []User, err error){
	// 动态填充id
	strIDs := make([]string, 0, len(ids))
	for _, id := range ids {
		strIDs = append(strIDs, fmt.Sprintf("%d", id))
	}
	query, args, err := sqlx.In("SELECT name, age FROM user WHERE id IN (?) ORDER BY FIND_IN_SET(id, ?)", ids, strings.Join(strIDs, ","))
	if err != nil {
		return
	}

	// sqlx.In 返回带 `?` bindvar的查询语句, 我们使用Rebind()重新绑定它
	query = DB.Rebind(query)

	err = DB.Select(&users, query, args...)
	return
}
```

