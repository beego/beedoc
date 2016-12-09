---
name: ORM Usage
sort: 2
---

# ORM Usage

The example of beego/orm is set out below:

All the code samples in this section are based on this example unless otherwise stated.

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
	User        *User   `orm:"reverse(one)"` // Reverse relationship (optional)
}

func init() {
	// Need to register model in init
	orm.RegisterModel(new(User), new(Profile))
}
```

##### main.go

```go
package main

import (
	"fmt"
	"github.com/astaxie/beego/orm"
	_ "github.com/go-sql-driver/mysql"
)

func init() {
	orm.RegisterDriver("mysql", orm.DRMySQL)

	orm.RegisterDataBase("default", "mysql", "root:root@/orm_test?charset=utf8")
}

func main() {
	o := orm.NewOrm()
	o.Using("default") // Using default, you can use other database

	profile := new(Profile)
	profile.Age = 30

	user := new(User)
	user.Profile = profile
	user.Name = "slene"

	fmt.Println(o.Insert(profile))
	fmt.Println(o.Insert(user))
}
```

## Set up database

ORM supports three popular databases. Here are the tested drivers, you need to import them:

```go
import (
	_ "github.com/go-sql-driver/mysql"
	_ "github.com/lib/pq"
	_ "github.com/mattn/go-sqlite3"
)
```

#### RegisterDriver

Three default databases:

```go
// For version 1.6
orm.DRMySQL
orm.DRSqlite
orm.DRPostgres

// < 1.6
orm.DR_MySQL
orm.DR_Sqlite
orm.DR_Postgres
```

```go
// param 1: driverName
// param 2: database type
// This mapping driverName and database type
// mysql / sqlite3 / postgres registered by default already
orm.RegisterDriver("mysql", orm.DRMySQL)
```

#### RegisterDataBase

ORM must register a database with alias `default`.

ORM uses golang built-in connection pool.

```go
// param 1:        Database alias. ORM will use it to switch database.
// param 2:        driverName
// param 3:        connection string
orm.RegisterDataBase("default", "mysql", "root:root@/orm_test?charset=utf8")

// param 4 (optional):  set maximum idle connections
// param 4 (optional):  set maximum connections (go >= 1.2)
maxIdle := 30
maxConn := 30
orm.RegisterDataBase("default", "mysql", "root:root@/orm_test?charset=utf8", maxIdle, maxConn)
```

#### SetMaxIdleConns

Set maximum idle connections according to database alias:

```go
orm.SetMaxIdleConns("default", 30)
```

#### SetMaxOpenConns

Set maximum connections (go >= 1.2) according to database alias:

```go
orm.SetMaxOpenConns("default", 30)
```

#### Timezone Config

ORM uses time.Local by default

* used for ORM automatically created time
* convert time queried from database into ORM local time

You can change it if needed:

```go
// Set to UTC time
orm.DefaultTimeLoc = time.UTC
```
ORM will get timezone of database while performing `RegisterDataBase`. When setting or getting time.Time it will convert accordingly to match system time and make sure the time is correct.

**Note:**

* Because of Sqlite3 set and get use UTC time by default.
* When using `go-sql-driver` driver，please pay attention to your DSN config.
  From a version of `go-sql-driver` the default uses utc timezone not local. So if you use another timezone, please set it.
  eg: `root:root@/orm_test?charset=utf8&loc=Asia%2FShanghai`
  ref: [loc](https://github.com/go-sql-driver/mysql#loc) / [parseTime](https://github.com/go-sql-driver/mysql#parsetime)

## Registering Model

This is compulsory if you use orm.QuerySeter for advanced query.

Otherwise you don't need to do this if you use raw SQL query and map struct only. [Raw SQL Query](rawsql.md)

#### RegisterModel

Register the Model you defined. The best practice is to have a single models.go file and register in it's init function.

Mini models.go

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

RegisterModel can register multiple models at the same time:

```go
orm.RegisterModel(new(User), new(Profile), new(Post))
```

For detailed struct definition, see [Model define](models.md)

#### Generate Tables

You may want Beego to automatically create your database tables.
One way to do this is by using the method described in the [cli](cmd.md) documentation. 
Alternatively you could choose to autogenerate your tables by including the following
in your main.go file in your main block. 

```go
// Database alias.
name := "default"

// Drop table and re-create.
force := true

// Print log.
verbose := true

// Error.
err := orm.RunSyncdb(name, force, verbose)
if err != nil {
	fmt.Println(err)
}
```
After the initial "bee run" command, change the values of force and verbose to false. 
The default behavior for Beego is to add additional columns when the model is updated.
You will need to manually handle dropping your columns if they are removed from your model. 


#### RegisterModelWithPrefix

Using table prefix

```go
orm.RegisterModelWithPrefix("prefix_", new(User))
```

The created table name is prefix_user

#### NewOrmWithDB

You may need to manage db pools by yourself. (eg: need two query in one connection)

But you want use orm awesome features. Bingo!

```go
var driverName, aliasName string
// driverName name of your driver (go-sql-driver: mysql)
// aliasName custom db alias name
var db *sql.DB
...
o := orm.NewOrmWithDB(driverName, aliasName, db)
```

#### GetDB

Get *sql.DB from registered databases. Will use `default` as default if you do not set.

```go
db, err := orm.GetDB()
if err != nil {
	fmt.Println("get default DataBase")
}

db, err := orm.GetDB("alias")
if err != nil {
	fmt.Println("get alias DataBase")
}
```

#### ResetModelCache

Reset registered models. Commonly used to write test cases.

```go
orm.ResetModelCache()
```

## ORM API Usage

Let's see how to use Ormer API:

```go
var o Ormer
o = orm.NewOrm() // create a Ormer // While running NewOrm, it will run orm.BootStrap (only run once in the whole app lifetime) to validate the definition between models and cache it.
```
Switching database or using transactions will affect Ormer object and all its queries.

So don't use a global Ormer object if you need to switch databases or use transactions.


* type Ormer interface {
	* [Read(interface{}, ...string) error](object.md#read)
	* [ReadOrCreate(interface{}, string, ...string) (bool, int64, error)](object.md#readorcreate)
	* [Insert(interface{}) (int64, error)](object.md#insert)
	* [InsertMulti(int, interface{}) (int64, error)](object.md#insertmulti)
	* [Update(interface{}, ...string) (int64, error)](object.md#update)
	* [Delete(interface{}) (int64, error)](object.md#delete)
	* [LoadRelated(interface{}, string, ...interface{}) (int64, error)](query.md#载入关系字段)
	* [QueryM2M(interface{}, string) QueryM2Mer](query.md#多对多关系操作)
	* [QueryTable(interface{}) QuerySeter](#querytable)
	* [Using(string) error](#using)
	* [Begin() error](transaction.md)
	* [Commit() error](transaction.md)
	* [Rollback() error](transaction.md)
	* [Raw(string, ...interface{}) RawSeter](#raw)
	* [Driver() Driver](#driver)
* }


#### QueryTable

Pass in a table name or a Model object and return a [QuerySeter](query.md#queryseter)

```go
o := orm.NewOrm()
var qs QuerySeter
qs = o.QueryTable("user")
// Panics if the table can't be found
```

#### Using

Switch to  another database:

```go
orm.RegisterDataBase("db1", "mysql", "root:root@/orm_db2?charset=utf8")
orm.RegisterDataBase("db2", "sqlite3", "data.db")

o1 := orm.NewOrm()
o1.Using("db1")

o2 := orm.NewOrm()
o2.Using("db2")

// After switching database
// The API calls of this Ormer will use the new database
```

Use `default` database, no need to use `Using`

#### Raw

Use raw SQL query:

Raw function will return a [RawSeter](rawsql.md) to execute a query with the SQL and params provided:

```go
o := NewOrm()
var r RawSeter
r = o.Raw("UPDATE user SET name = ? WHERE name = ?", "testing", "slene")
```

#### Driver

The current db infomation used by ORM

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
fmt.Println(dr.Type() == orm.DRMySQL) // true

o2 := orm.NewOrm()
o2.Using("db2")
dr = o2.Driver()
fmt.Println(dr.Name() == "db2") // true
fmt.Println(dr.Type() == orm.DRSqlite) // true
```

## Print out SQL query in debugging mode

Setting `orm.Debug` to true will print out SQL queries

It may cause performance issues. It's not recommend to be used in a production env.

```go
func main() {
	orm.Debug = true
...
```
Prints to `os.Stderr` by default.

You can change it to your own `io.Writer`

```go
var w io.Writer
...
// Use your `io.Writer`
...
orm.DebugLog = orm.NewLog(w)
```

Logs formatting

```go
[ORM] - time - [Queries/database name] - [operation/executing time] - [SQL query] - separate params with `,`  -errors 
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

The log contains all the database operations, transactions, prepare etc.
