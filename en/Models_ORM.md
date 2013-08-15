# Simple example

Otherwise specified, this will be the default example we talked in the manual.

#### models.go:

```go
package main

import (
	"github.com/astaxie/beego/orm"
)

type User struct {
	Id          int        `orm:"auto"`     // Set `auto` as primary key.
	Name        string
	Profile     *Profile   `spdi:"rel(one)"` // OneToOne relation.
}

type Profile struct {
	Id          int     `orm:"auto"`
	Age         int16
	User        *User   `orm:"reverse(one)"` // Set reverse relation(optional)
}

func init() {
	// Register defined model(s).
	orm.RegisterModel(new(User), new(Profile))
}
```

#### main.go

```go
package main

import (
	"fmt"
	"github.com/astaxie/beego/orm"
	_ "github.com/go-sql-driver/mysql"
)

func init() {
	orm.RegisterDriver("mysql", orm.DR_MySQL)

	orm.RegisterDataBase("default", "mysql", "root:root@/orm_test?charset=utf8", 30)
}

func main() {
	o := orm.NewOrm()
	o.Using("default") // ORM uses `default` as default database, you're able to change to others.

	profile := NewProfile()
	profile.Age = 30

	user := NewUser()
	user.Profile = profile
	user.Name = "slene"

	fmt.Println(o.Insert(profile))
	fmt.Println(o.Insert(user))
}
```

## Database setting

For now, ORM supports following three tested drivers:

```go
import (
	_ "github.com/go-sql-driver/mysql"
	_ "github.com/lib/pq"
	_ "github.com/mattn/go-sqlite3"
)
```

### RegisterDriver

Three default database types:

```go
orm.DR_MySQL
orm.DR_Sqlite
orm.DR_Postgres
```

```go
// Argument 1   driverName
// Argument 2   database name
// Used to corresponding driverName
// mysql / sqlite3 / postgres are registered by default, you don't need to do it by yourself.
orm.RegisterDriver("mymysql", orm.DR_MySQL)
```

### RegisterDataBase

You have to register a database named `default` as default database of ORM.

```go
// Argument 1   customized database name for changing database when use ORM.
// Argument 2   driverName
// Argument 3   corresponding connect string
// Argument 4   Maximum connection number, using connection pool of Go.
orm.RegisterDataBase("default", "mysql", "root:root@/orm_test?charset=utf8", 30)
```

### Timezone

ORM uses `time.Local` as default:

* For create time of ORM.
* Convert time from database to ORM local time.

You can change this setting as follows:

```go
// Set default timezone to UTC.
orm.DefaultTimeLoc = time.UTC
```

ORM converts time from database time to your setting local time when execute `RegisterDataBase`, this helps you have correct time.

**Attention** ORM uses UTC as default access time due to SQLite3 time format.

## RegisterModel

如果使用 `orm.QuerySeter` 进行高级查询的话，这个是必须的。

反之，如果只使用 Raw 查询和 map struct，是无需这一步的。您可以去查看 [Raw SQL 查询](Models_RawSQL)

将你定义的 Model 进行注册，最佳设计是有单独的 models.go 文件，在它的 init 函数中进行注册。

迷你版 models.go

```go
package main

import "github.com/astaxie/beego/orm"

type User struct {
	Id   int    `orm:"auto"`
	name string
}

func init(){
	orm.RegisterModel(new(User))
}
```

`RegisterModel` 也可以同时注册多个 model：

```go
orm.RegisterModel(new(User), new(Profile), new(Post))
```

## ORM 接口使用

使用 ORM 必然接触的 `Ormer` 接口，我们来熟悉一下

```go
var o Ormer
o = orm.NewOrm() // 创建一个 Ormer
// NewOrm 的同时会执行 orm.BootStrap (整个 app 只执行一次)，用以验证模型之间的定义并缓存。
```

* type Ormer interface {
	* [Read(Modeler) error](Models_Object#read)
	* [Insert(Modeler) (int64, error)](Models_Object#insert)
	* [Update(Modeler) (int64, error)](Models_Object#update)
	* [Delete(Modeler) (int64, error)](Models_Object#delete)
	* [M2mAdd(Modeler, string, ...interface{}) (int64, error)](Models_Object#m2madd)
	* [M2mDel(Modeler, string, ...interface{}) (int64, error)](Models_Object#m2mdel)
	* [LoadRel(Modeler, string) (int64, error)](Models_Object#loadRel)
	* [QueryTable(interface{}) QuerySeter](#querytable)
	* [Using(string) error](#using)
	* [Begin() error](Models_Transaction)
	* [Commit() error](Models_Transaction)
	* [Rollback() error](Models_Transaction)
	* [Raw(string, ...interface{}) RawSeter](#raw)
	* [Driver() Driver](#driver)
* }


### QueryTable

传入表名，或者 Modeler 对象，返回一个 [QuerySeter](Models_Query#queryseter)。

```go
o := orm.NewOrm()
var qs QuerySeter
qs = o.QueryTable("user")
// 如果表没有定义过，会立刻 panic
```

### Using

切换为其他数据库：

```go
orm.RegisterDataBase("db1", "mysql", "root:root@/orm_db2?charset=utf8", 30)
orm.RegisterDataBase("db2", "sqlite3", "data.db", 30)

o1 := orm.NewOrm()
o1.Using("db1")

o2 := orm.NewOrm()
o2.Using("db2")

// 切换为其他数据库以后
// 这个 Ormer 对象的其下的 api 调用都将使用这个数据库

```

默认使用 `default` 数据库，无需调用 Using。

### Raw

使用 SQL 语句直接进行操作。

Raw 函数，返回一个 [RawSeter](Models_RawSQL) 用以对设置的 SQL 语句和参数进行操作。

```go
o := NewOrm()
var r RawSeter
r = o.Raw("UPDATE user SET name = ? WHERE name = ?", "testing", "slene")
```

### Driver

返回当前 ORM 使用的 db 信息：

```go
type Driver interface {
	Name() string
	Type() DriverType
}
```

```go
orm.RegisterDataBase("db1", "mysql", "root:root@/orm_db2?charset=utf8", 30)
orm.RegisterDataBase("db2", "sqlite3", "data.db", 30)

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

可能存在性能问题，不建议使用在产品模式。

```go
func main() {
	orm.Debug = true
...
```

默认使用 `os.Stderr` 输出日志信息。

改变输出到你自己的 `io.Writer`：

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

日志内容包括 **所有的数据库操作** 、事务、Prepare 等。