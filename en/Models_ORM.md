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

If you want to use query like `orm.QuerySeter`, you have to use this method; otherwise, you can directly use raw query or map struct without this step, more detail please see [Raw SQL](Models_RawSQL).

The best practice for registering model is that register them in init function in the `models.go` file.

Mini `models.go`:

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

`RegisterModel` also can register many models at once:

```go
orm.RegisterModel(new(User), new(Profile), new(Post))
```

## ORM interface

The interface `Ormer` is an important part for using ORM, let's see how it works:

```go
var o Ormer
o = orm.NewOrm() // 创建一个 Ormer
// NewOrm will call orm.BootStrap(only once for single App) to verify model definition and cache them.
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

Pass table name or use Modeler to return a [QuerySeter](Models_Query#queryseter).

```go
o := orm.NewOrm()
var qs QuerySeter
qs = o.QueryTable("user")
// Program panics if the table hasn't been registered.
```

### Using

Change to other databases:

```go
orm.RegisterDataBase("db1", "mysql", "root:root@/orm_db2?charset=utf8", 30)
orm.RegisterDataBase("db2", "sqlite3", "data.db", 30)

o1 := orm.NewOrm()
o1.Using("db1")

o2 := orm.NewOrm()
o2.Using("db2")

// After you changed the database, all operations will be applied on the new one.

```

ORM uses `default` database as default, you don't need to use `Using` for that.

### Raw

Use SQL statements to operate.

Raw function returns a [RawSeter](Models_RawSQL) to set SQL statements and arguments for operations.

```go
o := NewOrm()
var r RawSeter
r = o.Raw("UPDATE user SET name = ? WHERE name = ?", "testing", "slene")
```

### Driver

It returns db information of current ORM:

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

## Print SQL in debug mode

Simply set `Debug` to true to print SQL.

We do not recommend you to enable this feature in product mode.

```go
func main() {
	orm.Debug = true
...
```

Use `os.Stderr` to print information as default.

You are able to change to print to any object that implemented interface `io.Writer`:

```go
var w io.Writer
...
// 设置为你的 io.Writer
...
orm.DebugLog = orm.NewLog(w)
```

Log format:

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

Log content including **all database operations**, transactions and Prepares, etc.