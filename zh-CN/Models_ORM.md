# ORM 使用方法

beego/orm 的使用例子

后文例子如无特殊说明都以这个为基础。

##### models.go:

```go
package main

import (
	"github.com/astaxie/beego/orm"
)

type User struct {
	Id          int
	Name        string
	Profile     *Profile   `orm:"rel(one)"` // OneToOne relation
}

type Profile struct {
	Id          int
	Age         int16
	User        *User   `orm:"reverse(one)"` // 设置反向关系(可选)
}

func init() {
	// 需要在init中注册定义的model
	orm.RegisterModel(new(User), new(Profile))
}
```

##### main.go

```go
package main

import (
	"fmt"
	"github.com/astaxie/beego/orm"
)

func init() {
	// 如果执行 RegisterDriver 则 _ "github.com/go-sql-driver/mysql" 不需要，否则出错 
	orm.RegisterDriver("mysql", orm.DR_MySQL)

	orm.RegisterDataBase("default", "mysql", "root:root@/orm_test?charset=utf8")
}

func main() {
	o := orm.NewOrm()
	o.Using("default") // 默认使用 default，你可以指定为其他数据库

	profile := new(Profile)
	profile.Age = 30

	user := new(User)
	user.Profile = profile
	user.Name = "slene"

	fmt.Println(o.Insert(profile))
	fmt.Println(o.Insert(user))
}
```

## 数据库的设置

目前 ORM 支持三种数据库，以下为测试过的 driver

将你需要使用的 driver 加入 import 中

```go
import (
	_ "github.com/go-sql-driver/mysql"
	_ "github.com/lib/pq"
	_ "github.com/mattn/go-sqlite3"
)
```

#### RegisterDriver

三种默认数据库类型

```go
orm.DR_MySQL
orm.DR_Sqlite
orm.DR_Postgres
```

```go
// 参数1   driverName
// 参数2   数据库类型
// 这个用来设置 driverName 对应的数据库类型
// mysql / sqlite3 / postgres 这三种是默认已经注册过的，所以可以无需设置
orm.RegisterDriver("mymysql", orm.DR_MySQL)
```

#### RegisterDataBase

ORM 必须注册一个别名为 `default` 的数据库，作为默认使用。

ORM 使用 golang 自己的连接池

```go
// 参数1        数据库的别名，用来在ORM中切换数据库使用
// 参数2        driverName
// 参数3        对应的链接字符串
orm.RegisterDataBase("default", "mysql", "root:root@/orm_test?charset=utf8")

// 参数4(可选)  设置最大空闲连接
// 参数5(可选)  设置最大数据库连接 (go >= 1.2)
maxIdle := 30
maxConn := 30
orm.RegisterDataBase("default", "mysql", "root:root@/orm_test?charset=utf8", maxIdle, maxConn)
```

#### SetMaxIdleConns

根据数据库的别名，设置数据库的最大空闲连接

```go
orm.SetMaxIdleConns("default", 30)
```

#### SetMaxOpenConns

根据数据库的别名，设置数据库的最大数据库连接  (go >= 1.2)

```go
orm.SetMaxOpenConns("default", 30)
```

#### 时区设置

ORM 默认使用 time.Local 本地时区

* 作用于 ORM 自动创建的时间
* 从数据库中取回的时间转换成 ORM 本地时间

如果需要的话，你也可以进行更改

```go
// 设置为 UTC 时间
orm.DefaultTimeLoc = time.UTC
```

ORM 在进行 RegisterDataBase 的同时，会获取数据库使用的时区，然后在 time.Time 类型存取的时做相应转换，以匹配时间系统，从而保证时间不会出错。

**注意:** 鉴于 Sqlite3 的设计，存取默认都为 UTC 时间

## 注册模型

如果使用 orm.QuerySeter 进行高级查询的话，这个是必须的。

反之，如果只使用 Raw 查询和 map struct，是无需这一步的。您可以去查看 [Raw SQL 查询](Models_RawSQL)

#### RegisterModel

将你定义的 Model 进行注册，最佳设计是有单独的 models.go 文件，在他的 init 函数中进行注册。


迷你版 models.go

```go
package main

import "github.com/astaxie/beego/orm"

type User struct {
	Id   int
	name string
}

func init(){
	orm.RegisterModel(new(User))
}
```

RegisterModel 也可以同时注册多个 model

```go
orm.RegisterModel(new(User), new(Profile), new(Post))
```

详细的 struct 定义请查看文档 [模型定义](Models_Models)

#### RegisterModelWithPrefix

使用表名前缀

```go
orm.RegisterModelWithPrefix("prefix_", new(User))
```

创建后的表名为 prefix_user

## ORM 接口使用

使用 ORM 必然接触的 Ormer 接口，我们来熟悉一下

```go
var o Ormer
o = orm.NewOrm() // 创建一个 Ormer
// NewOrm 的同时会执行 orm.BootStrap (整个 app 只执行一次)，用以验证模型之间的定义并缓存。
```

切换数据库，或者，进行事务处理，都会作用于这个 Ormer 对象，以及其进行的任何查询。

所以：需要 **切换数据库** 和 **事务处理** 的话，不要使用全局保存的 Ormer 对象。


* type Ormer interface {
	* [Read(interface{}, ...string) error](Models_Object#read)
	* [Insert(interface{}) (int64, error)](Models_Object#insert)
	* [Update(interface{}, ...string) (int64, error)](Models_Object#update)
	* [Delete(interface{}) (int64, error)](Models_Object#delete)
	* [LoadRelated(interface{}, string, ...interface{}) (int64, error)](Models_Query#载入关系字段)
	* [QueryM2M(interface{}, string) QueryM2Mer](Models_Query#多对多关系操作)
	* [QueryTable(interface{}) QuerySeter](#querytable)
	* [Using(string) error](#using)
	* [Begin() error](Models_Transaction)
	* [Commit() error](Models_Transaction)
	* [Rollback() error](Models_Transaction)
	* [Raw(string, ...interface{}) RawSeter](#raw)
	* [Driver() Driver](#driver)
* }


#### QueryTable

传入表名，或者 Model 对象，返回一个 [QuerySeter](Models_Query#queryseter)

```go
o := orm.NewOrm()
var qs QuerySeter
qs = o.QueryTable("user")
// 如果表没有定义过，会立刻 panic
```

#### Using

切换为其他数据库

```go
orm.RegisterDataBase("db1", "mysql", "root:root@/orm_db2?charset=utf8")
orm.RegisterDataBase("db2", "sqlite3", "data.db")

o1 := orm.NewOrm()
o1.Using("db1")

o2 := orm.NewOrm()
o2.Using("db2")

// 切换为其他数据库以后
// 这个 Ormer 对象的其下的 api 调用都将使用这个数据库
```

默认使用 `default` 数据库，无需调用 Using

#### Raw

使用 sql 语句直接进行操作

Raw 函数，返回一个 [RawSeter](Models_RawSQL) 用以对设置的 sql 语句和参数进行操作

```go
o := NewOrm()
var r RawSeter
r = o.Raw("UPDATE user SET name = ? WHERE name = ?", "testing", "slene")
```

#### Driver

返回当前 ORM 使用的 db 信息

```go
type Driver interface {
	Name() string
	Type() DriverType
}
```

```go
orm.RegisterDataBase("db1", "mysql", "root:root@/orm_db2?charset=utf8")
orm.RegisterDataBase("db2", "sqlite3", "data.db")

o1 := orm.NewOrm()
o1.Using("db1")
dr := o1.Driver()
fmt.Println(dr.Name() == "db1") // true
fmt.Println(dr.Type() == orm.DR_MySQL) // true

o2 := orm.NewOrm()
o2.Using("db2")
dr = o2.Driver()
fmt.Println(dr.Name() == "db2") // true
fmt.Println(dr.Type() == orm.DR_Sqlite) // true
```

## 调试模式打印查询语句

简单的设置 Debug 为 true 打印查询的语句

可能存在性能问题，不建议使用在产品模式

```go
func main() {
	orm.Debug = true
...
```

默认使用 os.Stderr 输出日志信息

改变输出到你自己的 io.Writer

```go
var w io.Writer
...
// 设置为你的 io.Writer
...
orm.DebugLog = orm.NewLog(w)
```

日志格式

```go
[ORM] - 时间 - [Queries/数据库名] - [执行操作/执行时间] - [SQL语句] - 使用标点 `,` 分隔的参数列表 - 打印遇到的错误
```

```go
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [    db.Exec /     0.4ms] - [INSERT INTO `user` (`name`) VALUES (?)] - `slene`
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [    db.Exec /     0.5ms] - [UPDATE `user` SET `name` = ? WHERE `id` = ?] - `astaxie`, `14`
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [db.QueryRow /     0.4ms] - [SELECT `id`, `name` FROM `user` WHERE `id` = ?] - `14`
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [    db.Exec /     0.4ms] - [INSERT INTO `post` (`user_id`,`title`,`content`) VALUES (?, ?, ?)] - `14`, `beego orm`, `powerful amazing`
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [   db.Query /     0.4ms] - [SELECT T1.`name` `User__Name`, T0.`user_id` `User`, T1.`id` `User__Id` FROM `post` T0 INNER JOIN `user` T1 ON T1.`id` = T0.`user_id` WHERE T0.`id` = ? LIMIT 1000] - `68`
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [    db.Exec /     0.4ms] - [DELETE FROM `user` WHERE `id` = ?] - `14`
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [   db.Query /     0.3ms] - [SELECT T0.`id` FROM `post` T0 WHERE T0.`user_id` IN (?) ] - `14`
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [    db.Exec /     0.4ms] - [DELETE FROM `post` WHERE `id` IN (?)] - `68`
```

日志内容包括 **所有的数据库操作**，事务，Prepare，等
