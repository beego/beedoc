# Models - beego ORM

[![Build Status](https://drone.io/github.com/astaxie/beego/status.png)](https://drone.io/github.com/astaxie/beego/latest) [![Go Walker](http://gowalker.org/api/v1/badge)](http://gowalker.org/github.com/astaxie/beego/orm)

Beego ORM is a powerful ORM framework for the Go programming language.

It's heavily influenced by Django ORM and SQLAlchemy.

This project still under deployment, changes may break APIs that have been used in your programs.

**Supported Database:**

* MySQL: [github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)
* PostgreSQL: [github.com/lib/pq](https://github.com/lib/pq)
* Sqlite3: [github.com/mattn/go-sqlite3](https://github.com/mattn/go-sqlite3)

Passed all test, but need more feedback.

**Features:**

* Full Go types support
* Easy for usage, simple CRUD operation
* Auto join with relation tables
* Cross DataBase compatible query
* Raw SQL query / mapper without orm model
* Fully tested keep stable and strong

more features please read the documentation.

**Installation:**

	go get github.com/astaxie/beego/orm

## Changelog

* 2013-08-13: Update test for database types
* 2013-08-13: Go types support, such as int8, uint8, byte, rune
* 2013-08-13: Improve supoort of date / datetime timezone

## Quick Start

### Simple Usage

```go
package main

import (
	"fmt"
	"github.com/astaxie/beego/orm"
	_ "github.com/go-sql-driver/mysql" // import your used driver
)

// Model Struct
type User struct {
	Id   int
	Name string `orm:"size(100)"`
}

func init() {
	// register model
	orm.RegisterModel(new(User))

	// set default database
	orm.RegisterDataBase("default", "mysql", "root:root@/my_db?charset=utf8", 30)
}

func main() {
	o := orm.NewOrm()

	user := User{Name: "slene"}

	// insert
	id, err := o.Insert(&user)
	fmt.Printf("ID: %d, ERR: %v\n", id, err)

	// update
	user.Name = "astaxie"
	num, err := o.Update(&user)
	fmt.Printf("NUM: %d, ERR: %v\n", num, err)

	// read one
	u := User{Id: user.Id}
	err = o.Read(&u)
	fmt.Printf("ERR: %v\n", err)

	// delete
	num, err = o.Delete(&u)
	fmt.Printf("NUM: %d, ERR: %v\n", num, err)
}
```

### Next with relation

```go
type Post struct {
	Id    int    `orm:"auto"`
	Title string `orm:"size(100)"`
	User  *User  `orm:"rel(fk)"`
}

var posts []*Post
qs := o.QueryTable("post")
num, err := qs.Filter("User__Name", "slene").All(&posts)
```

### Use Raw sql

Sometimes you may want to use Raw SQL to query / mapping without ORM setting.

```go
var maps []Params
num, err := o.Raw("SELECT id FROM user WHERE name = ?", "slene").Values(&maps)
if num > 0 {
	fmt.Println(maps[0]["id"])
}
```

### Transaction

```go
o.Begin()
...
user := User{Name: "slene"}
id, err := o.Insert(&user)
if err == nil {
	o.Commit()
} else {
	o.Rollback()
}
```

### Debug Log Queries

In development environment, you can simply use:

	func main() {
		orm.Debug = true
	...

to enable log queries.

Outputs including all queries, such as exec / prepare / transaction.

For example:

	[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [    db.Exec /     0.4ms] - 	[INSERT INTO `user` (`name`) VALUES (?)] - `slene`
	...

Note: we are not recommend using this in product environment.
